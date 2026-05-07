[← FPGA Subsystem](README.md) · [↑ Knowledge Base](../README.md)

# FPGA Debugging & Telemetry Tools

Debugging a hardware core running in real-time is significantly more challenging than debugging software — you cannot simply add a `printf` statement. This document covers the four primary debugging approaches available to MiSTer core developers: SignalTap II logic analysis, HPS telemetry tunneling, LED-based status indicators, and physical probe measurement.

---

## 1. Internal Probing: SignalTap II

**SignalTap II** is Intel's embedded logic analyzer that captures internal FPGA signals in real-time while the core is running on the DE10-Nano.

### 1.1 Prerequisites

- **USB Blaster Connection**: Connect a USB cable to the DE10-Nano's USB-Blaster port (Mini-USB near the DC jack)
- **JTAG Server**: The Quartus JTAG server must be running and detect the DE10-Nano
- **BRAM Budget**: SignalTap uses M10K blocks for capture memory. On heavy cores (PSX, Saturn), you may only have enough BRAM for a few signals at shallow depth

### 1.2 Setup Procedure

1. In Quartus, open **Tools → SignalTap II Logic Analyzer**
2. Add the signals you wish to monitor (e.g., `cpu_reset`, `vsync`, `bus_addr`)
3. Assign a **Sample Clock** (usually `clk_sys`)
4. Set the **Sample Depth** (trade-off: deeper capture = more M10K blocks)
5. Configure **Triggers**: Basic (edge, level) or Advanced (multi-signal conditions)
6. Compile the core — the SignalTap logic synthesizes into the `.rbf`
7. Program the DE10-Nano and use the SignalTap window to capture

### 1.3 BRAM Cost Estimation

| Sample Depth | 16 signals | 32 signals | 64 signals |
|---|---|---|---|
| 256 | 0.5 M10K | 1 M10K | 2 M10K |
| 1K | 2 M10K | 4 M10K | 8 M10K |
| 4K | 8 M10K | 16 M10K | 32 M10K |
| 16K | 32 M10K | 64 M10K | 128 M10K |

The MemTest core uses 48/553 M10K blocks (9%). With SignalTap at 32 signals × 4K depth, you'd consume an additional 16 M10K — easily affordable. The PSX core uses ~400/553 M10K blocks, leaving very little room.

### 1.4 Tips

- **Filter with triggers**: Don't capture everything. Set a trigger on `vsync` rising edge to capture one frame's worth of data.
- **Use signal groups**: Organize related signals (CPU bus, DMA bus) into groups and enable only the one you need.
- **Incremental debugging**: Start with a wide but shallow capture, narrow the trigger, then increase depth.

---

## 2. HPS-to-FPGA Telemetry (Tunneling)

Because the FPGA and HPS share the AXI bridges, you can "tunnel" debug information from the core to the Linux side without consuming BRAM.

### 2.1 Using `HPS_BUS` / `io_dout`

The `HPS_BUS[48:0]` is bidirectional. A core developer can map internal status registers to the `io_dout[15:0]` field:

```verilog
// In emu.v — map debug register to io_dout
always @(*) begin
    case(io_addr)
        4'h0: io_dout = cpu_pc[15:0];     // Program counter
        4'h1: io_dout = {7'd0, cpu_state}; // CPU state
        4'h2: io_dout = dma_count;         // DMA counter
        default: io_dout = 16'd0;
    endcase
end
```

On the HPS side, read `/dev/mem` at the FPGA Manager's GPI address to retrieve the data.

### 2.2 The UART `printf` Equivalent

Some developers implement a simple UART TX module in the core and wire it to an HPS UART or memory-mapped register:

```verilog
// Simple debug UART — 8N1 at 115200 baud
module debug_uart (
    input clk, input [7:0] char, input send,
    output tx
);
    // Shift out 8 data bits + start/stop
endmodule
```

The HPS polls the register and prints characters to the Linux console, giving you a `printf`-like debugging experience.

### 2.3 Framework Status Word

The `hps_io.sv` module provides a 32-bit `status` word that the HPS reads every frame. Bits 16–31 are available for core-defined status:

```verilog
// In emu.v — drive status[31:16] with debug info
assign status[31] = cpu_halt;
assign status[30] = dma_active;
assign status[29:16] = scanline_count;
```

The HPS logs this to the debug console or displays it in the OSD.

---

## 3. Physical Indicators: Status LEDs

The MiSTer framework maps system-level events to the physical LEDs on the DE10-Nano and I/O Board:

| LED | Location | Default Mapping | Source |
|---|---|---|---|
| **USER LED (Green)** | DE10-Nano | Core-defined status | `LED_USER` from `emu.v` |
| **DISK LED (Amber)** | DE10-Nano | SD Card / HDD activity | `LED_DISK` from `hps_io.sv` |
| **POWER LED (Red)** | DE10-Nano | 5V Power Rails | Hardware-wired |
| **LED 1 (Green)** | IO Board | Core-defined | `LED_POWER[0]` |
| **LED 2 (Amber)** | IO Board | Core-defined | `LED_POWER[1]` |

> [!TIP]
> During initial core bring-up, map the CPU's `HALT` or `ERROR` state to the USER LED for immediate visual feedback of a crash. This costs zero resources and provides instant diagnostics.

---

## 4. Physical Probing: Logic Analyzers & Oscilloscopes

For timing-critical debugging (SDRAM phase shifts, SNAC latency, video sync), a logic analyzer or oscilloscope is required.

### 4.1 GPIO Pin Mapping

The IO Board's User I/O (SNAC) pins provide direct, unbuffered access to FPGA pins. Internal signals can be routed to these pins:

```verilog
// Route debug signals to GPIO pins for oscilloscope
assign GPIO_1[0] = sdram_clk;        // SDRAM clock
assign GPIO_1[1] = sdram_cas;        // CAS signal
assign GPIO_1[2] = cpu_access;       // CPU bus cycle
```

### 4.2 SDRAM Phase Measurement

When debugging SDRAM timing:
1. Route `SDRAM_CLK` and one `SDRAM_DQ` bit to GPIO pins
2. Measure the delay between clock edge and data transition with an oscilloscope
3. Adjust the PLL phase shift until setup margin is confirmed

### 4.3 Video Sync Measurement

For video timing issues:
1. Route `VGA_HS`, `VGA_VS`, and `VGA_DE` to GPIO pins
2. Measure pulse widths and verify they match the expected timing
3. Check for glitches (sub-ns pulses) that indicate timing violations

---

## 5. MemTest Core

The [MemTest_MiSTer](https://github.com/MiSTer-devel/MemTest_MiSTer) core is itself a debugging tool — it walks the entire SDRAM address space and validates every bit. It is the **gold standard** for verifying SDRAM reliability.

The core reports errors via the OSD, including the failing address and expected vs. actual data. This is sufficient to diagnose:
- Phase alignment issues (errors in specific bit positions)
- Trace length mismatches (errors at specific addresses)
- Refresh starvation (gradual corruption over time)
- Chip select problems (errors on one module only)

---

## 6. Cross-References

- [Compilation Guide](fpga_compilation_guide.md) — Timing closure and SignalTap synthesis considerations
- [SDRAM Timing Theory](sdram_timing_theory.md) — Phase alignment measurement techniques
- [HPS IO Module](hps_io_module.md) — The `io_dout` and `status` word interfaces for telemetry
- [Input Latency & SNAC](input_latency_and_snac.md) — Physical probe measurement of input latency
- [Framework Overview](fpga_framework_overview.md) — HPS_BUS signal mapping
