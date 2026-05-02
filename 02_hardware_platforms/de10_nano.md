[← Section Index](README.md) · [↑ Knowledge Base](../README.md)

# DE10-Nano Hardware Reference

The **Terasic DE10-Nano** is a development kit built around the Intel/Altera Cyclone V System-on-Chip (SoC). Originally designed for university engineering courses, robotics, and industrial IoT prototyping, it was selected in 2017 as the foundation for the MiSTer project. At the time, it represented an unprecedented intersection of a high-capacity FPGA, a capable ARM processor, and affordable pricing.

This document serves as the physical hardware reference for the core board within the MiSTer stack, documenting the silicon capabilities and explaining how the MiSTer architecture maps to the board's physical layout.

---

## 1. Cyclone V SoC Architecture

The physical brain of the DE10-Nano is the **Cyclone V SE 5CSEBA6U23I7NDK**. Built on TSMC's 28-nm Low Power (LP) process, this chip integrates two entirely separate computing paradigms on a single silicon die, communicating via internal AMBA AXI bridges.

### 1.1 FPGA Fabric Capacity & Limits

The "A6" in the part number denotes the largest FPGA fabric available in this specific 672-pin U23 package. While software emulators are constrained by IPC (Instructions Per Clock) and cache misses, FPGA cores are constrained by logic availability, memory blocks, and signal routing congestion.

| Resource | Quantity | Technical Implications for Emulation |
|---|---|---|
| **Logic Elements (LEs)** | ~110,000 | The fundamental metric for complexity. 110K LEs is vast for 8/16-bit consoles (SNES is ~35K, Amiga is ~25K) but becomes a tight constraint for 32-bit systems. The N64 and PlayStation cores push this to >85% utilization, which severely complicates place-and-route timing closure in Quartus. |
| **Adaptive Logic Modules (ALMs)** | 41,500 | Altera's architecture groups LEs into ALMs. An ALM contains an 8-input fracturable look-up table (LUT), two adders, and four registers. Efficient HDL targets ALM packing to reduce routing delays across the die. |
| **Block RAM (M10K)** | 5,570 Kb (~696 KB) | Dedicated 10-Kilobit internal SRAM blocks. M10K blocks are true dual-port, crucial for video line buffers, custom caches, sprite attribute tables, and internal boot ROMs. |
| **Memory Logic Array Blocks (MLABs)** | 621 Kb | ALMs configured as small, distributed RAM (640 bits each) instead of logic. Useful for small shift registers or tiny LUTs without wasting large M10K blocks. |
| **DSP Blocks** | 112 | Hardened silicon for fast math (18×18 or 27×27 multipliers). Essential for the `ascal.vhd` polyphase scaler (resizing video without latency) and implementing complex audio synthesis filters (e.g., Yamaha FM chips or the PlayStation SPU reverb). |
| **Fractional PLLs** | 6 | Phase-Locked Loops. Retro consoles rely on highly specific, non-standard master clocks (e.g., SNES at 21.477 MHz). The PLLs take the board's 50 MHz reference and synthesize exact frequencies with zero jitter. |

### 1.2 The Hard Processor System (HPS)

The HPS is the "host" computer. It manages the platform, loads the FPGA bitstreams via the Linux FPGA Manager, handles file I/O, and manages the USB/Network stack.

* **CPU**: Dual-core ARM Cortex-A9 MPCore running at 800 MHz. It includes NEON SIMD engines and a hardware FPU.
* **HPS Memory**: 1 GB DDR3 SDRAM with a 32-bit data bus. 
  * > [!WARNING]
    > **Architectural Constraint**: This RAM is physically wired exclusively to the HPS memory controller. The FPGA can access it via the internal F2H AXI bridge, but latency is high and non-deterministic due to contention with the Linux kernel and memory refresh cycles. Cycle-accurate hardware emulation *cannot* use this for primary system RAM, which is why the external MiSTer SDRAM board was invented.
* **Peripherals**: The HPS contains hardened silicon controllers for Gigabit Ethernet, USB 2.0 OTG, MicroSD MMC, and UART.

---

## 2. Physical Expansion Headers & MiSTer Mapping

The DE10-Nano provides a limited number of physical pins routed to the outside world. To recreate complete computer systems, MiSTer utilizes almost every available GPIO pin.

### 2.1 GPIO-0 (40-pin Header)
* **Physical Location**: Top edge of the board.
* **Routing**: Connected directly to the FPGA fabric.
* **MiSTer Role**: Connects to the **I/O Board**.
* **Architectural Function**: This header is dedicated to the user interface and analog outputs. The `sys_top.v` framework routes specific internal signals here:
  * **Analog Video**: VGA_R, VGA_G, VGA_B (typically 6-bits per color), HSync, VSync. These drive the R-2R resistor ladder DACs on the I/O board.
  * **Analog Audio**: Routes digital audio to the I/O board's DAC (Sigma-Delta or I2S).
  * **SNAC**: Dedicates pins for the Serial Native Accessory Converter, allowing zero-latency polling of original OEM controllers (NES, Genesis, etc.) directly by the FPGA core.
  * **Secondary SD Card**: A dedicated SPI bus for cores that require mounting hard drive images independently of the HPS (e.g., the `ao486` core).
  * **User Buttons & LEDs**: OSD toggle, user reset, and status indicators.

### 2.2 GPIO-1 (40-pin Header)
* **Physical Location**: Bottom edge of the board.
* **Routing**: Connected directly to the FPGA fabric.
* **MiSTer Role**: Connects to the **SDRAM Board**.
* **Architectural Function**: This header implements a wide, parallel bus to provide deterministic memory to the cores.
  * **Data Bus**: 16-bit or 32-bit parallel bidirectional data (`SDRAM_DQ`).
  * **Address Bus**: 13-bit multiplexed address (`SDRAM_A`).
  * **Control Lines**: Bank select (BA0, BA1), CAS, RAS, WE, clock, and clock enable.
  * *Implementation Note*: The memory controller (`sdram.sv`) runs entirely inside the FPGA fabric. Because these traces are exposed and unshielded, the physical SDRAM board design must be exceptionally clean to support >130 MHz operation required by cores like the Amiga or SNES.

### 2.3 LTC Connector (14-pin Header)
* **Physical Location**: Near the Arduino header.
* **MiSTer Role**: Interfaces with the bottom side of the I/O Board.
* **Usage**: Routes signals primarily for analog audio input (Tape/ADC) into the FPGA. Cores like the ZX Spectrum, Amstrad CPC, or Apple II use this to load software from real cassette tapes via physical audio jacks on the I/O board.

### 2.4 Arduino Uno R3 Header
* **Physical Location**: Middle of the board.
* **Hardware**: Includes standard digital GPIO and 6 analog inputs connected to an onboard 12-bit ADC (LTC2308).
* **MiSTer Role**: Generally unused in the standard stack. The I/O board physical footprint covers this header. However, specific community modifications occasionally leverage the onboard ADC for niche analog controller inputs (like potentiometers or steering wheels) bypassing the USB stack.

---

## 3. Clock & Power Infrastructure

### 3.1 Power Delivery
* **Input**: 5V DC via a 5.5mm outer / 2.1mm inner barrel jack.
* **Regulators**: The board features Linear Technology (LTC) step-down regulators to generate the necessary 3.3V, 2.5V, 1.5V (DDR3), 1.2V, and 1.1V (FPGA Core) rails.
* **Power Budgeting**: The bare DE10-Nano consumes around 0.5A to 0.8A depending on FPGA load. 
> [!CAUTION]
> A fully expanded MiSTer setup (USB Hub, WiFi adapter, I/O board fan, SDRAM, connected controllers) can pull 1.5A to 2.0A. Using the stock 2A supply is marginal. Voltage droop under load will cause the SDRAM controller to misread data, resulting in silent core crashes or graphical glitches. A high-quality **5V / 3A (or greater)** power supply is required.

### 3.2 Clock Distribution
* **HPS Reference**: A 25 MHz oscillator drives the ARM Cortex-A9 PLLs and the DDR3 memory controller.
* **FPGA Reference**: Two 50 MHz oscillators (connected to different FPGA clock banks) provide the base clocks for the FPGA fabric. The MiSTer `sys_top.v` framework routes these into the Cyclone V's fractional PLLs to synthesize the exact, non-standard frequencies required by the retro cores.

---

## 4. Official Documentation & Datasheets

For deep hardware hacking, exact pinout routing, or custom board design, refer to the official vendor documentation:

| Document | Link | Description |
|---|---|---|
| **Terasic DE10-Nano Product Page** | [Terasic DE10-Nano](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&CategoryNo=218&No=1046) | Official product page, specs, and purchase links. |
| **DE10-Nano User Manual** | [User Manual PDF (via Terasic)](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&CategoryNo=218&No=1046&PartNo=4) | Contains the exact pin assignments for GPIO-0 and GPIO-1 (Requires free Terasic account to download, or found in MiSTer hardware repos). |
| **DE10-Nano Schematics** | [Schematics PDF (via Terasic)](https://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&CategoryNo=218&No=1046&PartNo=4) | Complete board schematics (Rev. C is most common). |
| **Cyclone V Device Datasheet** | [Altera Datasheet](https://docs.altera.com/r/docs/683801/current/cyclone-v-device-datasheet) | Primary silicon document: electrical, switching, and timing specifications. |
| **Cyclone V Documentation Hub** | [Altera Documentation](https://docs.altera.com/search?keyword=Cyclone%20V%20Device%20Handbook) | Central search portal for the Cyclone V Device Handbook and architectural specifications. |

> **See also**: For details on the community-developed boards that plug into these headers, see [Add-on Boards](addon_boards.md) (Planned). For logical data routing within the SoC, see [Platform Architecture](../01_system_architecture/platform_architecture.md).
