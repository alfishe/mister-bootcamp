[← FPGA Subsystem](README.md) · [↑ Knowledge Base](../README.md)

# SDRAM Timing Theory & Phase Alignment

A source-grounded analysis of the timing challenges in driving external SDR SDRAM at 100–167 MHz via the DE10-Nano's GPIO headers. Covers round-trip delay budgets, the altddio_out clock generation mechanism, PLL phase-shift strategies, and frequency-specific timing margins.

Sources:
* [`Menu_MiSTer/rtl/sdram.sv`](https://github.com/MiSTer-devel/Menu_MiSTer/blob/master/rtl/sdram.sv) — altddio_out instantiation
* [`Minimig-AGA_MiSTer/Minimig.sv`](https://github.com/MiSTer-devel/Minimig-AGA_MiSTer/blob/master/Minimig.sv) — PLL + pll_cfg runtime reconfiguration

## Table of Contents

1. [The Challenge: GPIO Propagation Delay](#1-the-challenge-gpio-propagation-delay)
2. [The altddio_out Clock Generation](#2-the-altddio_out-clock-generation)
3. [Timing Budget Analysis](#3-timing-budget-analysis)
4. [PLL Phase-Shift Configuration](#4-pll-phase-shift-configuration)
5. [Frequency-Specific Requirements](#5-frequency-specific-requirements)
6. [MemTest as Validation Tool](#6-memtest-as-validation-tool)
7. [Common Failure Modes](#7-common-failure-modes)
8. [Cross-References](#cross-references)

---

## 1. The Challenge: GPIO Propagation Delay

The MiSTer SDRAM daughterboard is connected to the FPGA via the **GPIO-1** header. Unlike internal DDR3 (which uses hardened silicon controllers and on-die termination), the SDR SDRAM relies on standard FPGA I/O pins and PCB traces.

At high frequencies, the time it takes for a signal to travel from the FPGA to the SDRAM chip (and back) becomes significant relative to the clock period:

| Frequency | Clock Period | 1/4 Period | Margin Name |
|-----------|-------------|-----------|-------------|
| 96 MHz | 10.4 ns | 2.6 ns | Comfortable |
| 100 MHz | 10.0 ns | 2.5 ns | Standard |
| 112 MHz | 8.9 ns | 2.2 ns | Minimig Amiga |
| 120 MHz | 8.3 ns | 2.1 ns | Tight |
| 143 MHz | 7.0 ns | 1.75 ns | Very tight |
| 167 MHz | 6.0 ns | 1.5 ns | Extreme |

The fundamental problem: the FPGA and SDRAM chip are **separate physical devices** connected by ~3–5 cm of PCB trace. Signals must travel:
1. **FPGA → SDRAM**: Address, command, data (writes) — must meet setup time at SDRAM input
2. **SDRAM → FPGA**: Read data — must travel back and meet setup time at FPGA input

Each direction incurs:
- FPGA output pin delay: ~2–3 ns
- PCB trace propagation: ~0.2 ns/cm × 3–5 cm = ~0.6–1.0 ns
- SDRAM input setup time: ~1.5 ns
- SDRAM output access time: ~5–6 ns (CL=2)

---

## 2. The altddio_out Clock Generation

All MiSTer SDRAM controllers use the same clock generation technique: an `altddio_out` DDR output primitive that inverts the SDRAM clock relative to the internal logic clock.

```verilog
// sdram.sv:L271-294 — DDR clock output
altddio_out #(
    .intended_device_family("Cyclone V"),
    .width(1)
) sdramclk_ddr (
    .datain_h(1'b0),   // Rising edge of outclock → drive low
    .datain_l(1'b1),   // Falling edge of outclock → drive high
    .outclock(clk),     // System clock input
    .dataout(SDRAM_CLK) // Output to SDRAM chip
);
```

### 2.1 How It Works

The DDR output primitive drives two data values per clock cycle — one on the rising edge of `outclock`, one on the falling edge:

```
Internal clk:  ___/‾‾‾\___/‾‾‾\___/‾‾‾\___
datain_h:           0         0         0
                  ↓         ↓         ↓
datain_l:      1         1         1
            ↓         ↓         ↓
SDRAM_CLK:  ‾‾‾\___/‾‾‾\___/‾‾‾\___
```

The result: `SDRAM_CLK` is the **inverse** of the internal `clk`. When the internal logic sees a rising edge, the SDRAM sees a falling edge, and vice versa.

### 2.2 Why Inversion Helps

Consider a write operation:
1. On the rising edge of `clk`, the FPGA drives address/command/data onto the pins
2. These signals propagate through the output buffers and PCB traces
3. The SDRAM samples them on the **next** rising edge of `SDRAM_CLK`

Because `SDRAM_CLK` is inverted, the SDRAM's rising edge occurs at the **falling edge** of the internal clock — giving the signals a full half-cycle (~5 ns at 100 MHz) to propagate and stabilize before being sampled.

For reads:
1. The SDRAM outputs data aligned to `SDRAM_CLK` rising edge
2. The FPGA samples the data on the **next** rising edge of `clk`
3. Again, the inversion gives a half-cycle of propagation time

---

## 3. Timing Budget Analysis

### 3.1 Write Path Budget (FPGA → SDRAM)

At 100 MHz with altddio_out inversion alone (no additional PLL shift):

```
Timing window: half cycle = 5.0 ns

Available:    |------- 5.0 ns -------|
FPGA output:  |== 2.5 ns ==|            FPGA pin driver delay
PCB trace:    |== 0.8 ns ==|            ~4 cm trace
SDRAM setup:  |== 1.5 ns ==|            tDS (data setup time)
                               |== 0.2 ns ==|  Margin
```

At 100 MHz, the margin is ~0.2 ns — very tight. Most cores add PLL phase shift to increase this.

### 3.2 Read Path Budget (SDRAM → FPGA)

```
Timing window: full cycle minus SDRAM access = 10.0 ns - tAC

SDRAM access: |==== 6.0 ns ====|        tAC (CL=2 output access)
PCB trace:                      |== 0.8 ns ==|  Return trace delay
FPGA setup:                               |== 0.5 ns ==|  FPGA pin tSU
                                              |== 2.7 ns ==|  Margin
```

The read path has more margin than the write path because the full SDRAM access time (tAC) is accounted for by the CAS latency pipeline, and the return data has a full cycle to propagate.

### 3.3 Why Write Path Is the Bottleneck

The write path is constrained because:
- Data and clock travel in the **same direction** (FPGA → SDRAM)
- The SDRAM samples data on the next clock edge
- Only a half-cycle is available for setup

The read path is easier because:
- Data and clock travel in **opposite directions** (SDRAM → FPGA)
- The CAS latency pipeline provides a full cycle for propagation

---

## 4. PLL Phase-Shift Configuration

The altddio_out inversion provides an automatic ~180° phase shift, but this is often insufficient for the write path margin. Most cores add additional **negative phase shift** via the PLL to advance `SDRAM_CLK` relative to the internal clock.

### 4.1 How PLL Phase Shift Works

The Cyclone V PLL can shift the output clock phase in fine increments. A negative phase shift means `SDRAM_CLK` transitions **earlier** than the internal `clk`, giving more setup time at the SDRAM inputs.

```
No PLL shift:             clk ___/‾‾‾\___
                          SDRAM_CLK ‾‾‾\___/‾‾‾
                          Data valid: ----|  ← 0.2 ns margin

PLL shift -45° (2.5 ns): clk ___/‾‾‾\___
                          SDRAM_CLK ‾\___/‾‾‾\_
                          Data valid: ------|  ← 2.7 ns margin
```

### 4.2 Runtime PLL Reconfiguration (Minimig)

The Minimig core uses a `pll_cfg` megafunction to adjust the PLL at runtime when switching between PAL and NTSC modes:

```verilog
// Minimig.sv:L272-301 — PLL + runtime reconfiguration
pll pll (
    .refclk(CLK_50M),
    .outclk_0(clk_114),
    .outclk_1(clk_sys),
    .reconfig_to_pll(reconfig_to_pll),
    .reconfig_from_pll(reconfig_from_pll),
    .locked(locked)
);

pll_cfg pll_cfg (
    .mgmt_clk(CLK_50M),
    .mgmt_write(cfg_write),
    .mgmt_address(cfg_address),
    .mgmt_writedata(cfg_data),
    .reconfig_to_pll(reconfig_to_pll),
    .reconfig_from_pll(reconfig_from_pll)
);
```

This allows the core to reconfigure the PLL's VCO frequency and phase shift without reloading the bitstream — essential for the Amiga's PAL/NTSC frequency switch.

---

## 5. Frequency-Specific Requirements

### 5.1 Speed Grades and Module Selection

| Module Type | Max Frequency | CAS Latency | Chip Rating | Typical Use |
|-------------|--------------|-------------|-------------|-------------|
| PC100 | 100 MHz | CL=2 | -8E (8 ns tCK) | Most 8/16-bit cores |
| PC133 | 133 MHz | CL=3 | -75 (7.5 ns tCK) | SNES, Genesis, some arcade |
| PC167 | 167 MHz | CL=3 | -6 (6 ns tCK) | Saturn, Neo Geo, high-end arcade |

### 5.2 CAS Latency Trade-Off

At higher frequencies, CAS latency must increase from CL=2 to CL=3:

```verilog
// sdram.sv:L71 — Generic controller uses CL=2 for <100 MHz
localparam CAS_LATENCY = 3'd2;  // 2 for < 100MHz, 3 for >100MHz

// sdram.v:L160 — MemTest controller uses CL=3 for stability
sdaddr2 <= 13'b000_0_00_011_0_010;  // CL=3, burst=4
```

| CAS Latency | Min Clock | Max Clock (PC133) | Read Latency (cycles) |
|-------------|-----------|-------------------|----------------------|
| CL=2 | — | 100 MHz | 2 |
| CL=3 | 100 MHz | 133+ MHz | 3 |

CL=3 adds one cycle of read latency but allows operation at higher frequencies. The MemTest controller always uses CL=3 for maximum stability across all module types.

### 5.3 Dual-SDRAM Considerations

With two SDRAM modules installed, both chips share the same clock, address, and command lines but have separate chip selects. The additional capacitive loading on shared signals reduces the maximum achievable frequency:

- Single module: ~2–3 pF load per pin
- Dual module: ~4–6 pF load per pin

This additional loading increases rise/fall times, potentially reducing the maximum frequency by 10–20 MHz on marginal modules.

---

## 6. MemTest as Validation Tool

The [MemTest_MiSTer](https://github.com/MiSTer-devel/MemTest_MiSTer) core is the definitive tool for verifying SDRAM timing margins. It performs:

1. **Sequential walk**: Writes and reads every address in the SDRAM address space
2. **Pattern tests**: Uses walking-1s, walking-0s, checkerboard, and pseudo-random patterns
3. **Chip validation**: Tests both chip selects independently
4. **Frequency sweep**: Can be configured at different PLL frequencies

If MemTest passes at a given frequency and phase shift, the SDRAM is reliable for core operation. If it fails:
- **Specific address range**: Likely a row/address wiring issue
- **Random bit errors**: Phase alignment problem — try more negative PLL shift
- **Consistent bit position**: PCB trace length mismatch on that data line

---

## 7. Common Failure Modes

### 7.1 Insufficient Write Setup Margin

**Symptom**: MemTest shows errors on writes but not reads. Data corruption is pattern-dependent.

**Cause**: The PLL phase shift is not negative enough. The SDRAM samples data before it has stabilized.

**Fix**: Increase the negative PLL phase shift by 5–10° and retest.

### 7.2 Hold Violation at High Frequency

**Symptom**: Errors appear only at 120+ MHz. Passes at 100 MHz.

**Cause**: At high frequencies, the half-cycle window becomes too small. The data changes before the SDRAM has finished sampling the previous value.

**Fix**: Use CL=3 instead of CL=2, and ensure the SDRAM chip is rated for the target frequency (PC133 or PC167).

### 7.3 Trace Length Mismatch

**Symptom**: Specific data bits (e.g., DQ[4]) always fail while others pass.

**Cause**: A PCB trace for that data line is significantly longer or shorter than the others, causing it to arrive out of phase.

**Fix**: Use a better quality SDRAM module with length-matched traces. The official MiSTer SDRAM boards are designed with matched trace lengths.

### 7.4 Refresh Starvation at High Utilization

**Symptom**: Gradual data corruption over time (seconds to minutes of operation).

**Cause**: The core is issuing requests so frequently that the SDRAM controller cannot schedule refresh cycles within the 64 ms window.

**Fix**: Ensure the core has natural idle periods where the controller can insert refreshes. See [SDRAM Controller Deep Dive](sdram_controller.md) §10 for refresh scheduling strategies.

---

## Cross-References

- [SDRAM Controller Deep Dive](sdram_controller.md) — Complete analysis of all three controller implementations: state machines, initialization, refresh, arbitration
- [Memory Controllers](memory_controllers.md) — Architectural overview of SDRAM vs DDR3 memory subsystems
- [DDR3 Architecture](ddr3_architecture.md) — The DDR3/F2H AXI path for non-deterministic high-bandwidth access
- [FPGA Performance Metrics](fpga_performance_metrics.md) — SDRAM bandwidth and utilization baselines
- [FPGA Compilation Guide](fpga_compilation_guide.md) — Timing closure strategies for SDRAM controllers
