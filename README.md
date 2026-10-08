# Synthesizable AMBA AXI4-Lite Interface for On-Chip BRAM Memory Subsystem

[![Verilog HDL](https://img.shields.io/badge/Language-Verilog_HDL-blue.svg)](https://en.wikipedia.org/wiki/Verilog)
[![FPGA Target](https://img.shields.io/badge/Target-Xilinx_Artix--7-red.svg)](https://www.xilinx.com)
[![Protocol](https://img.shields.io/badge/Protocol-AMBA_AXI4--Lite-green.svg)](https://developer.arm.com)
[![Course](https://img.shields.io/badge/Course-E3_231_Digital_Systems_Design_with_FPGAs-purple.svg)](https://iisc.ac.in)

Author: **Anirudh Chaturvedi**  
M.Tech in Microelectronics & VLSI Design, Indian Institute of Science (IISc), Bangalore  
Department of Electronic Systems Engineering (DESE)

---

## 1. Architectural Overview

This repository contains a complete, synthesizable hardware implementation of the **AMBA AXI4-Lite memory-mapped interface** architected from RTL primitives. The interface manages bidirectional read and write access to a 32-bit $\times$ 256 on-chip Block RAM (BRAM) on the **AMD Xilinx Artix-7 FPGA (`XC7A35T`)**.

### Key Architectural Highlights
- **Dual Communicating FSMs:** Autonomous Finite State Machines implemented in `AXI4_LITE_MASTER` and `AXI4_LITE_SLAVE` coordinating memory-mapped transactions with zero CPU intervention.
- **5 Independent Handshake Channels:** Full adherence to the ARM AMBA AXI4 specification with decoupled Address Write (AW), Write Data (W), Write Response (B), Address Read (AR), and Read Data (R) channels.
- **Strict VALID/READY Interlocking:** Guaranteed deadlock-free protocol invariants (`VALID` assertions independent of `READY`, no circular combinational dependencies).
- **Burst Command Decoding:** 16-bit host command format (`CMD_IN`) supporting burst address generation up to 128 words.

---

## 2. Command Protocol & Channel Hierarchy

The top-level controller (`AXI4_LITE_TOP`) coordinates testbench commands, Master FSM transactions, Slave FSM translations, and synchronous single-port BRAM memory operations.

### Command Structure (`CMD_IN[15:0]`)
| Bits | Field | Description |
| :---: | :---: | :--- |
| `[15]` | **R/W Flag** | `0` = Write transaction, `1` = Read transaction |
| `[14:8]` | **Word Count** | 7-bit burst length (number of consecutive 32-bit words) |
| `[7:0]` | **Base Address** | 8-bit starting word address in on-chip BRAM ($0$ to $255$) |

---

## 3. Five-Channel Protocol Handshake

```
+--------------------+                       +--------------------+
|                    | == [AW] AWADDR, AWVALID ==> |                    |
|                    | <= [AW] AWREADY ========= |                    |
|                    |                           |                    |
|  AXI4-Lite Master  | == [W]  WDATA,  WVALID ===> |  AXI4-Lite Slave   |
|  (AXI4_LITE_MASTER)| <= [W]  WREADY ========== |  (AXI4_LITE_SLAVE) |
|                    |                           |                    |
|                    | <= [B]  BRESP,  BVALID === |                    |
|                    | == [B]  BREADY ==========> |                    |
|                    |                           |                    |
|                    | == [AR] ARADDR, ARVALID ==> |                    |
|                    | <= [AR] ARREADY ========= |                    |
|                    |                           |                    |
|                    | <= [R]  RDATA,  RVALID === |                    |
|                    | == [R]  RREADY ==========> |                    |
+--------------------+                       +--------------------+
                                                        ||
                                            +------------------------+
                                            | On-Chip 32x256 BRAM    |
                                            +------------------------+
```

---

## 4. Resource Utilization & Implementation Results

Target Device: **Xilinx Artix-7 FPGA (`XC7A35T-1CPG236C`)**

| Resource | Used | Available | Utilization (%) |
| :--- | :---: | :---: | :---: |
| **Slice LUTs** | < 150 | 20,800 | **< 1.0%** |
| **Flip-Flops (FF)** | < 120 | 41,600 | **< 0.5%** |
| **Block RAM (BRAM 36Kb)** | 1 | 50 | **2.0%** |
| **Clock Frequency** | 100 MHz | - | **Setup Slack Met (WNS > +5.0 ns)** |

---

## 5. Verification & Simulation Stages

Verification was carried out across three distinct simulation stages in AMD Xilinx Vivado:
1. **Behavioral Simulation (RTL):** Cycle-accurate protocol handshakes and memory array validation.
2. **Post-Synthesis Functional Simulation:** Gate-level functional validation verifying FSM state encoding.
3. **Post-Implementation Timing Simulation:** Verified under full parasitic wire delays and physical clock skew.

Detailed technical monograph and interview defense analyses are available in [`docs/Design_and_Implementation_of_AXI4_Lite_Interface_Report.pdf`](docs/Design_and_Implementation_of_AXI4_Lite_Interface_Report.pdf) and [`docs/Guide_7_AXI4_Lite_BRAM_Interface.pdf`](docs/Guide_7_AXI4_Lite_BRAM_Interface.pdf).
