[← FPGA Subsystem](README.md) · [↑ Knowledge Base](../README.md)

# FPGA Performance Metrics & Resource Utilization

This document provides a technical baseline for the resource consumption and timing performance of the MiSTer platform on the Intel Cyclone V SoC (5CSEBA6U23C8).

---

## 1. Hardware Baseline: Cyclone V 5CSEBA6U23C8

The DE10-Nano is equipped with a middle-range Cyclone V SoC. Understanding its ceiling is critical for core development and multi-core (dual-SDRAM) planning.

| Resource Type | Total Capacity | Description |
| :--- | :--- | :--- |
| **Logic Elements (LE)** | 110,000 | Generic logic units. |
| **Adaptive Logic Modules (ALM)** | 41,509 | Grouped logic units (preferred for Quartus optimization). |
| **M10K Block RAM** | 5,570 Kbits | ~696 KB of internal, low-latency memory. |
| **DSP Blocks** | 112 | Hardened multipliers/accumulators (used for audio/video processing). |
| **Global Clock Networks** | 16 | Dedicated low-skew paths. |

---

## 2. Representative Core Resource Usage

Below are approximate utilization metrics for popular MiSTer cores. These values include the `sys/` framework overhead.

| Core | ALMs | M10K (Kbits) | DSPs | Complexity Level |
| :--- | :--- | :--- | :--- | :--- |
| **NES** | ~3,500 (8%) | ~250 | 1 | Low |
| **SNES** | ~14,000 (33%) | ~1,200 | 18 | Medium |
| **Genesis** | ~11,000 (26%) | ~800 | 2 | Medium |
| **PSX (PlayStation)** | ~35,000 (84%) | ~4,800 | 60 | **High (Near Ceiling)** |
| **AO486** | ~22,000 (53%) | ~3,500 | 4 | High |
| **Saturn** | ~39,500 (95%) | ~5,400 | 90 | **Extreme (Fitting Challenge)** |

> [!IMPORTANT]
> **The 80% Rule**: In FPGA design, as utilization crosses 80%, Quartus's ability to achieve timing closure (meeting $F_{max}$) drops exponentially. High-complexity cores like PSX and Saturn often require "Seed Sweeping" (compiling with different random logic placements) to find a valid route.

---

## 3. Clocking & $F_{max}$ (Frequency Maximum)

### 3.1 Timing Domains
MiSTer cores typically operate in three primary clock domains:

1.  **Core Domain (`clk_sys`)**: The emulated system's master clock (e.g., 21.47 MHz for SNES).
2.  **Framework Domain (`clk_100m`)**: A fixed clock for the HPS-FPGA bridge and framework logic.
3.  **Video Domain (`clk_vid`)**: The pixel clock required for the current output resolution (e.g., 148.5 MHz for 1080p60).

### 3.2 Achieving Timing Closure
Quartus must ensure that signals can travel between registers within one clock cycle. 

*   **Low Frequency Cores**: Systems like the Commodore 64 or NES run at <10 MHz. $F_{max}$ is rarely an issue.
*   **High Frequency Cores**: Modern scalers or systems like the PSX (33 MHz) or Saturn require careful pipelining. If the core's $F_{max}$ is lower than its required operating frequency, the system will behave erratically or crash.

> [!TIP]
> Use the **TimeQuest Timing Analyzer** in Quartus to view the "Slack" of your design. Positive slack means timing is met; negative slack requires logic optimization or DSE sweeps.

---

## 4. Power & Thermal Considerations

While the DE10-Nano is relatively low-power, high-utilization cores (Saturn, PSX) can significantly increase the FPGA temperature.

*   **Idle Power**: ~2-3W.
*   **Heavy Load**: Can peak at 5-7W.
*   **Thermal Mitigation**: A heatsink and fan are highly recommended for the Cyclone V when running cores with >70% ALM utilization to prevent logic glitches caused by thermal-induced timing drift.

---

## Read Also
* [FPGA Compilation Guide](fpga_compilation_guide.md) — How to achieve timing closure for heavy cores.
* [Memory Controllers](memory_controllers.md) — Understanding how BRAM and SDRAM are utilized.
