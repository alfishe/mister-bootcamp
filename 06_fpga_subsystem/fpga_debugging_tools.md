[← FPGA Subsystem](README.md) · [↑ Knowledge Base](../README.md)

# FPGA Debugging & Telemetry Tools

Debugging a hardware core running in real-time is significantly more challenging than debugging software. This document outlines the tools available within the MiSTer framework for internal signal probing and telemetry.

---

## 1. Internal Probing: SignalTap II

**SignalTap II** is a logic analyzer embedded within the Quartus toolchain that allows you to capture and view internal FPGA signals in real-time while the core is running on the DE10-Nano.

### 1.1 Requirements
*   **USB Blaster Connection**: You must connect a USB cable to the DE10-Nano's USB-Blaster port (the Mini-USB port near the DC jack).
*   **BRAM Budget**: SignalTap uses the FPGA's internal M10K blocks to store captured data. On heavy cores (PSX, Saturn), you may not have enough BRAM left to probe more than a few signals.

### 1.2 Setup
1.  In Quartus, open the **SignalTap II Logic Analyzer**.
2.  Add the signals you wish to monitor (e.g., `cpu_reset`, `v-sync`, `bus_addr`).
3.  Assign a **Sample Clock** (usually the core's `clk_sys`).
4.  Compile the core. The SignalTap logic will be synthesized into the `.rbf`.
5.  Run the core and use the SignalTap window to trigger on specific conditions.

---

## 2. HPS-to-FPGA Telemetry (Tunneling)

Because the FPGA and HPS share the AXI bridges, you can "tunnel" debug information from the core to the Linux side without using SignalTap's precious BRAM.

### 2.1 Using `HPS_BUS`
The `HPS_BUS[48:0]` is a two-way street. A core developer can map internal status registers to the `io_dout` (16-bit) field.
*   **FPGA Side**: Map the register to `io_dout` when a specific UIO chip-select is active.
*   **HPS Side**: Use a simple C utility or Python script to read `/dev/mem` at the FPGA Manager's GPI address (`0xFF706014`) and log the values.

### 2.2 The `printf` equivalent
Some developers implement a simple UART tx module in the core and wire it to one of the HPS UART pins or map it to a memory-mapped register that the HPS polls and prints to the Linux console.

---

## 3. Physical Indicators: Status LEDs

The MiSTer framework maps several system-level events to the physical LEDs on the DE10-Nano and the I/O Board.

| LED | Default Mapping | Source |
| :--- | :--- | :--- |
| **USER LED (Green)** | Core-defined status | Driven by `emu.v` |
| **DISK LED (Amber)** | SD Card / HDD activity | Driven by `hps_io.sv` |
| **POWER LED (Red)** | 5V Power Rails | Hardware-wired |

> [!TIP]
> During initial core bring-up, it is common practice to map the CPU's `HALT` or `ERROR` state to the USER LED for immediate visual feedback of a crash.

---

## 4. Signal Probing via GPIO (Oscilloscopes)

For timing-critical debugging (like SDRAM phase shifts or SNAC latency), a logic analyzer or oscilloscope is required.
*   The **User I/O (SNAC)** pins on the I/O Board provide direct, unbuffered access to FPGA pins.
*   Mapping internal signals to these pins allows for sub-nanosecond measurement of signal propagation.

---

## Read Also
* [Framework Overview](fpga_framework_overview.md) — For mapping the HPS_BUS.
* [Compilation Guide](fpga_compilation_guide.md) — For SignalTap synthesis considerations.
