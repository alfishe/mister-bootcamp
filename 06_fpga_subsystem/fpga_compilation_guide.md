[← FPGA Subsystem](README.md) · [↑ Knowledge Base](../README.md)

# FPGA Compilation & Timing Closure Guide

This document outlines the standard compilation workflow for MiSTer FPGA cores, focusing on the Intel Quartus toolchain and strategies for achieving timing closure on the Cyclone V.

---

## 1. Toolchain Requirements

### 1.1 Intel Quartus Prime Lite
The MiSTer project standard is **Intel Quartus Prime Lite Edition 17.0**. 
*   **Why 17.0?**: Newer versions often produce larger bitstreams or require different IP license handling. Version 17.0 is known for its stability and compatibility with the `Template_MiSTer` framework.
*   **License**: The Lite Edition is free and supports the 5CSEBA6U23C8 chip used in the DE10-Nano.

### 1.2 Essential Files
Every core repository typically contains:
*   `project.qpf`: The Quartus project file.
*   `project.qsf`: Quartus Settings File (pin assignments, global settings).
*   `project.sdc`: Synopsys Design Constraints (clock definitions, input/output delays).

---

## 2. Compilation Workflow

### 2.1 The Build Process
1.  **Framework Sync**: Ensure the `sys/` submodule is up to date.
2.  **Analysis & Synthesis**: Converts HDL to a logic gate netlist.
3.  **Fitter (Place & Route)**: Maps gates to physical ALMs and routing wires on the FPGA.
4.  **Assembler**: Generates the `.sof` (SRAM Object File).
5.  **Conversion**: The `.sof` is converted to an uncompressed `.rbf` (Raw Binary File) for MiSTer to load.

### 2.2 RBF Generation
MiSTer requires a Raw Binary File (`.rbf`). This is usually generated via a script or the Quartus GUI:
*   **Settings**: Passive Parallel x16, Uncompressed.
*   **Command**: `quartus_cpf -c project.sof project.rbf`

---

## 3. Achieving Timing Closure

For complex cores (e.g., PSX, Saturn), the default Fitter settings may fail to meet timing (resulting in negative slack).

### 3.1 Seed Sweeping
The Fitter uses a random "seed" to start its placement algorithm. If timing fails, changing the seed often results in a successful build by finding a more efficient routing path.
*   MiSTer developers often use scripts to compile 10-20 seeds in parallel and pick the one with the best (most positive) slack.

### 3.2 Design Space Explorer (DSE II)
Quartus includes DSE II, a tool that automatically iterates through different optimization settings (High Area, High Performance, Low Power) and seeds to find the optimal configuration for your core.

### 3.3 Overclocking the Framework
While retro cores run slowly, the `sys/` framework (HDMI, SDRAM controller) often runs at 100MHz or higher. If the framework fails timing, the OSD might flicker or the SDRAM might corrupt data.

---

## 4. Common Compilation Hazards

| Hazard | Symptom | Mitigation |
| :--- | :--- | :--- |
| **Logic Bloat** | ALM utilization > 95% | Refactor HDL, use M10K for LUTs (ROMs), or simplify core features. |
| **Routing Congestion** | Timing fails despite low ALM usage | High fan-out signals (like Resets) need to be distributed via Global Clock buffers. |
| **Missing SDC Constraints** | Core works on one build but fails on another | Ensure all clocks (`clk_sys`, `clk_vid`) are explicitly defined in the `.sdc` file. |

---

## Read Also
* [Performance Metrics](fpga_performance_metrics.md) — Typical resource counts for cores.
* [Memory Controllers](memory_controllers.md) — How to optimize BRAM usage.
