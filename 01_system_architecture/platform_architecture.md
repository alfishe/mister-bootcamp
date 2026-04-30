[← Hardware Platform Section](README.md)

# MiSTer Hardware Platform Architecture

> **Target**: Deep Technical Reference
> **Status**: Outline / Placeholders Only

This document details the physical hardware architecture of the MiSTer platform, breaking down how the various PCBs, interconnects, and silicon components form a unified system capable of cycle-accurate hardware recreation.

## 1. Competitive Architectural Analysis & Evolution

*Placeholder: Trace the lineage of the project and compare it against contemporary alternatives to show how the physical architecture arrived at its current state.*

### 1.1 The Minimig Architecture (The Origin)
*Placeholder: Discuss Dennis van Weeren's original Minimig. A single, small FPGA dedicated entirely to the Amiga chipset, paired with a very basic microcontroller.*

```mermaid
%% Placeholder for Minimig architecture diagram (FPGA + basic PIC/MCU)
```
- **Pros:** Groundbreaking proof of concept, focused simplicity.
- **Cons:** Extremely limited logic elements, tightly coupled to a single platform, no standardized I/O framework.

### 1.2 The MiST Architecture (The Stepping Stone)
*Placeholder: Discuss Till Harbaum's MiST board. The shift to a larger Altera Cyclone III and a dedicated ARM microcontroller for I/O and OSD management. This established the "Core + Controller" paradigm.*

```mermaid
%% Placeholder for MiST architecture diagram (Cyclone III + dedicated ARM over SPI)
```
- **Pros:** Established the multi-core platform concept, introduced a standardized OSD and generic I/O firmware.
- **Cons:** Slow external SPI bus bottleneck between ARM and FPGA, limited RAM bandwidth, exhausted logic capacity for 32-bit generation cores.

### 1.3 The Analogue Pocket Architecture (The Commercial Alternative)
*Placeholder: Contrast with modern commercial offerings. The Pocket uses two FPGAs (a primary Cyclone V and a secondary Cyclone 10) but lacks an embedded Linux-capable SoC, relying instead on proprietary AnalogueOS firmware.*

```mermaid
%% Placeholder for Analogue Pocket architecture diagram (Dual FPGA, proprietary firmware)
```
- **Pros:** Highly polished consumer experience, dual-FPGA allows seamless OS overlay without consuming core logic, integrated portable form factor.
- **Cons:** Closed ecosystem, proprietary development tools (Analogue Studio), no underlying Linux OS for advanced network/USB extension, hard limits on open-source community contributions.

### 1.4 The MiSTer SoC Architecture (The Synthesis)
*Placeholder: Explain Sorgelig's genius move to adopt the cheap, off-the-shelf DE10-Nano. The paradigm shift of moving from a discrete FPGA + external MCU to a modern SoC where the hard-silicon ARM Cortex-A9 (HPS) and FPGA fabric are fused on the same die. The internal high-speed AXI interconnects completely eliminated the old I/O bottlenecks.*

```mermaid
%% Placeholder for MiSTer SoC architecture diagram (Cyclone V SoC, HPS-FPGA AXI bridges, Linux stack)
```
- **Pros:** Massive logic capacity, elimination of external bus bottlenecks via AXI, full embedded Linux stack (network, Bluetooth, USB), entirely open-source framework (`sys_top`).
- **Cons:** Relies on a specific industrial development board subject to market pricing, steeper learning curve for Linux-layer configuration.

## 2. System Block Diagram

*Placeholder: A comprehensive Mermaid `flowchart TD` or block diagram illustrating the DE10-Nano, SDRAM board, I/O board, USB Hub, and how they interconnect via GPIO headers and LTC connectors. This is the "big picture" physical view.*

```mermaid
%% Placeholder for full physical block diagram
```

## 3. The Core Board: Terasic DE10-Nano

### Cyclone V SoC (5CSEBA6U23I7)
*Placeholder: Detail the heart of the system. Dual-core ARM Cortex-A9 (HPS) running at 800MHz paired with 110K LE FPGA fabric. Discuss the internal AXI bridges (H2F, F2H, LWH2F).*

### DDR3 Memory (HPS-Side)
*Placeholder: Discuss the 1GB DDR3 RAM. Why it's attached to the HPS side and how the FPGA accesses it via the F2H bridge for non-timing-critical storage (e.g., ROM caching, CD images, save states).*

### On-board Peripherals
*Placeholder: The integrated HDMI TX chip (ADV7513), Gigabit Ethernet, MicroSD slot, and USB OTG.*

## 4. The Expansion Stack: Add-on Boards

### 4.1 The SDRAM Board (FPGA-Side)
*Placeholder: The most critical add-on. Explain **why** DDR3 isn't enough. Discuss the need for zero-latency, deterministic memory access to mimic retro console architectures (like the Amiga Chip RAM or Neo Geo ROM space). Cover standard (32MB) vs. dual/128MB setups.*

### 4.2 The I/O Board
*Placeholder: Detail the analog breakout. Analog video (VGA/Component/RGB), analog audio (via sigma-delta DAC or I2S), secondary SD card slot, user buttons, and status LEDs.*

### 4.3 The USB Hub Board
*Placeholder: Explain how it interfaces with the DE10-Nano's single USB OTG port via the pogo pins or bridge board, providing 7 USB ports for controllers, Wi-Fi/Bluetooth dongles, and keyboards.*

### 4.4 Direct Video & SNAC
*Placeholder: 
- **Direct Video:** Bypassing the I/O board to push raw 24-bit analog RGB/YPbPr through the HDMI port using a cheap DAC dongle.
- **SNAC (Serial Native Accessory Converter):** Connecting original OEM controllers directly to the FPGA GPIO to bypass the Linux/USB stack entirely, achieving zero input latency.*

## 5. Power Architecture

*Placeholder: Discuss the power delivery system. The DE10-Nano's 5V input, power limits when running a full stack (Hub + IO + heavy cores), and the necessity of the barrel jack splitter or upgraded power supplies.*

## 6. Architectural Data Flows

*Placeholder: Trace specific paths through the hardware. Detailed sub-sections with Mermaid diagrams showing exactly how data moves through the system during typical operation.*

- **Boot Sequence:** Linux init -> Main_MiSTer -> RBF loading.
- **Input Pipeline:** USB Device -> Linux evdev -> Main_MiSTer -> HPS2FPGA Bridge -> Core vs. SNAC -> GPIO -> FPGA.
- **Video Pipeline:** Core Video Out -> ascal.vhd (Scaler) -> HDMI Transmitter vs. FPGA -> Resistor ladder (VGA).
- **Storage/Memory Pipeline:** Core requesting ROM data -> HPS intercept -> DDR3 caching -> SDRAM injection.
- **Audio Pipeline:** FPGA -> I2S -> I/O Board DAC vs. FPGA -> I2S -> HDMI TX.
