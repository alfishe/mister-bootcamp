[← Knowledge Base](../README.md)

# MiSTer FPGA Platform Architecture & Evolution

> **Target**: Deep Technical Reference & Historical Context
> **Status**: Outline / Placeholders Only

This document serves as the definitive high-level overview of the MiSTer FPGA platform. It explores not just *how* the system is architected, but *why* it succeeded, how it evolved from its predecessors, and the fundamental differences between software emulation and cycle-accurate hardware recreation.

## 1. The Core Philosophy: Why MiSTer is Unique

*Placeholder: Explain that MiSTer is not a closed console or a commercial product, but an open-source hardware and software framework. Discuss the concept of the "bare-metal" approach versus OS-mediated emulation.*

### Hardware Recreation vs. Software Emulation
*Placeholder: Deep dive into the fundamental differences.*
- *Sequential execution (software/CPU) vs. Massively parallel execution (FPGA).*
- *The concept of "Cycle Accuracy" at the gate level.*
- *Latency advantages (zero-lag controller polling, instant display output) compared to the USB/OS/Driver/Render pipeline of software emulators.*

### The "Not Just Different" Factor
*Placeholder: Discuss why hardware preservation is distinct from software preservation. It's about replicating the electrical timing, bus contention, and analog quirks of the original silicon, not just translating opcodes.*

## 2. Platform Architecture & Evolution

*Placeholder: Briefly summarize the platform's architectural evolution. Explain how the project moved from the single-FPGA Minimig, through the "FPGA + ARM microcontroller" MiST, to finally arriving at the Cyclone V SoC (fusing the ARM Cortex-A9 and FPGA fabric onto a single die).*

> **Deep Dive**: For a comprehensive breakdown of the physical hardware, add-on boards, and a competitive comparison with Analogue Pocket and Minimig, see the [Hardware Platform Architecture](../01_system_architecture/platform_architecture.md) reference.

### The Architect: Alexey Melnikov (Sorgelig)
*Placeholder: The genesis and enduring success of the MiSTer platform is inextricably linked to its founder and lead architect, Alexey Melnikov, known within the community as **[Sorgelig](https://github.com/sorgelig)**. Transitioning from early contributions to the MiST platform, he recognized the bottleneck of the legacy architecture and single-handedly pioneered the pivot to the Terasic DE10-Nano. 

Sorgelig's role goes far beyond simple project management; he is the "eternal maintainer" of the platform's core infrastructure. He authored the `Main_MiSTer` Linux binary, established the `sys_top` bridging framework, and either created or substantially optimized the vast majority of the core catalog (ranging from Arcade boards to complex 16/32-bit consoles).

**Development Impact (Estimated as of April 30, 2026):**
*   **Total Commits:** ~9,000+ across the `MiSTer-devel` organization.
*   **Lines of Code (SLOC):** While exact line counts fluctuate with upstream merges, his direct contributions across VHDL, SystemVerilog, C/C++, and Linux shell scripts easily exceed **1,000,000+** lines of code, encompassing the underlying framework, hardware scaling (Scaler/Menu logic), and direct core logic creation.

### The Community Ecosystem: Key Contributors
While Sorgelig acts as the platform's anchor, MiSTer's explosive growth and ability to emulate advanced 32/64-bit systems is the result of a dedicated cadre of high-level FPGA engineers and software developers. 

| Contributor / Handle | Primary Domain | Notable Contributions / Special Thanks |
| :--- | :--- | :--- |
| **Till Harbaum** ([harbaum](https://github.com/harbaum)) | Platform Forefather | Creator of the original MiST board. His "FPGA + ARM" concept paved the foundational path for the DE10-Nano port. |
| **Dennis van Weeren** | The Originator | Creator of the Minimig. Proved cycle-accurate Amiga FPGA recreation was possible, launching the entire modern hardware preservation movement. |
| **Robert Peip** ([FPGAzumSpass](https://github.com/FPGAzumSpass)) | 32/64-bit Consoles | PlayStation (PSX), Nintendo 64 (N64), Game Boy Advance, WonderSwan. Pioneered SDRAM/DDR3 caching architectures for high-bandwidth systems. |
| **Jose Tejada** ([Jotego](https://github.com/jotego)) | Arcade Preservation | The colossal "JT" arcade library. CPS1/1.5/2/3, Neo Geo, Sega System 16, and exhaustive reverse-engineering of custom Yamaha sound chips. |
| **Sergey Dvodnenko** ([srg320](https://github.com/srg320)) | Complex Multi-CPU Systems | Sega Saturn, Sega 32X, MegaCD. Shattered expectations by proving the DE10-Nano could handle wildly complex multi-processor environments. |
| **[Kitrinx](https://github.com/Kitrinx)** | Audio, Video, & Framework | Advanced Shadow Mask rendering, I2S audio integration, critical SNES core accuracy improvements, and N64 co-development. |
| **[Rysha](https://github.com/Rysha)** | CD-ROM Subsystems | Mastered optical drive timing emulation. Crucial CD-ROM block implementation and debugging for PSX, Saturn, and MegaCD. |
| **[Theypsilon](https://github.com/theypsilon)** | Software Ecosystem | Creator of the `update_all` script, the definitive automated tool that seamlessly synchronizes cores, scripts, and arcade MRA files. |
| **[dnotq](https://github.com/dnotq)** | Hardware / Input Latency | Pioneered SNAC (Serial Native Accessory Converter), bypassing the Linux USB stack for true zero-lag OEM controller support. |
| **Alan** ([alanswx](https://github.com/alanswx)) | Linux Stack / Connectivity | SAMBA/network integration, secondary SD card support, and foundational UI/OSD enhancements. |


### From Project to Platform
*Placeholder: Describe the transition from a single developer's pet project to a standardized API/framework. Explain how the `sys/` framework abstracted the board I/O, allowing developers to focus on core logic rather than HDMI scalers or SD card protocols.*

## 3. Platform Architecture (High-Level)

*Placeholder: Provide a broad architectural overview of the current platform.*

### The HPS-FPGA Duality
*Placeholder: Diagram and explain the division of labor. HPS (Linux) handles "slow" tasks: USB polling, SD card filesystem access, network, UI. FPGA handles "fast" tasks: the actual core logic, video scaling, audio DAC.*

### The Physical Stack
*Placeholder: Outline the physical add-on boards and their architectural necessity.*
- *DE10-Nano (Core).*
- *SDRAM Board (Why the FPGA needs deterministic, zero-latency memory that the DDR3 cannot provide).*
- *I/O Board (Analog video/audio, SNAC).*
- *USB Hub (Bridging input to the Linux layer).*

## 4. The Explosion: Key Success Factors

*Placeholder: Analyze why MiSTer succeeded where other FPGA projects remained niche or highly commercialized.*

### The Standardized Framework
*Placeholder: Explain how `Main_MiSTer` and the `sys/` folder democratized core development. A developer could port a core from MiST and get HDMI, USB controllers, and an OSD "for free."*

### The Open Source Ecosystem
*Placeholder: Contrast with closed-source commercial FPGA consoles (e.g., Analogue). Discuss the momentum of global collaboration, shared components (like the jtframe audio chips), and the community-driven update scripts.*

### Unprecedented Fidelity
*Placeholder: Highlight specific milestones that proved the platform's capability (e.g., the ao486 core, the fully cycle-accurate PSX core, the N64 milestone). Show how it shifted from 8/16-bit to 32/64-bit generation.*

## 5. Architectural Slices & Data Flows (Coming Soon)

*Placeholder: Detailed sub-sections with Mermaid diagrams showing exactly how data moves through the system during typical operation.*

- **Boot Sequence:** Linux init -> Main_MiSTer -> RBF loading.
- **Input Pipeline:** USB Device -> Linux evdev -> Main_MiSTer -> HPS2FPGA Bridge -> Core.
- **Video Pipeline:** Core Video Out -> ascal.vhd (Scaler) -> HDMI Transmitter.
- **Storage/Memory Pipeline:** Core requesting ROM data -> HPS intercept -> DDR3 caching -> SDRAM injection.

---

## 6. Entry Points

| File | Topic | Quality |
|---|---|---|
| [start_retro_user.md](start_retro_user.md) | Getting started for retro gamers | 📋 Target: Adequate |
| [start_linux_dev.md](start_linux_dev.md) | Getting started for software / Linux developers | 📋 Target: Adequate |
| [start_fpga_dev.md](start_fpga_dev.md) | Getting started for FPGA / HDL engineers | 📋 Target: Adequate |

## 7. Cross-References

- [Hardware Platform](../02_hardware_platforms/README.md) — DE10-Nano board, add-on boards
- [Framework](../06_fpga_subsystem/README.md) — sys_top, hps_io, HPS_BUS
- [FPGA KB — Cyclone V](https://github.com/alfishe/fpga-bootcamp/blob/main/01_vendors_and_families/altera_intel/)
