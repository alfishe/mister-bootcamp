# MiSTer FPGA Knowledge Base — TODO & Gap Analysis

> **Last updated**: 2026-04-30
>
> This file tracks every planned article, its current status, quality tier, and priority.
> See [AGENTS.md](AGENTS.md) for quality tier definitions (Deep / Adequate / Shallow).

---

## Status Legend

| Symbol | Meaning |
|---|---|
| ✅ | Complete — meets target quality tier |
| 🔄 | In progress |
| 📋 | Planned — not started |
| 🔀 | Migrated from existing docs (needs breadcrumb + cross-links) |
| ⬆️ | Needs quality upgrade |

---

## Phase 1 — Foundation (The Physical Layer)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 1 | `00_overview/README.md` | ✅ | Target: Deep | Evolution from MiST/Minimig, HW vs SW emulation, platform success factors |
| 2 | `00_overview/getting_started.md` | ✅ | Target: Adequate | Persona-based guide: retro gamers, arcade builders, CRT enthusiasts, students, software devs, FPGA engineers, tinkerers, preservationists |
| 5 | `01_system_architecture/platform_architecture.md` | ✅ | Target: Deep | Abstract SoC concepts, FPGA+HPS pipelines, physical data flows |
| 6 | `02_hardware_platforms/de10_nano.md` | 📋 | Target: Deep | Physical board reference, specs, pins |
| 7 | `02_hardware_platforms/addon_boards.md` | 📋 | Target: Adequate | Physical addons: SDRAM, IO board, USB hub, Direct Video |

## Phase 2 — Host Software (The ARM/HPS Control Plane)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 8 | `03_hps_linux/custom_linux.md` | 📋 | Target: Deep | Buildroot, kernel, U-Boot documentation |
| 9 | `04_hps_binary/architecture.md` | 📋 | Target: Deep | Main_MiSTer module map |
| 10| `04_hps_binary/build/overview.md` | 📋 | Target: Deep | ARM toolchain, Makefile, cross-compile |
| 11| `05_configuration/mister_ini_guide.md` | 📋 | Target: Adequate | User-facing INI guide |
| 12| `05_configuration/osd.md` | 🔀 | Target: Adequate | OSD rendering and navigation architecture |

## Phase 3 — Hardware Emulation (The FPGA Plane)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 13| `06_fpga_subsystem/sys_top.md` | 📋 | Target: Deep | Top-level hardware abstraction |
| 14| `06_fpga_subsystem/hps_io_module.md` | 📋 | Target: Deep | Central framework module |
| 15| `06_fpga_subsystem/sdram_controller.md` | 📋 | Target: Deep | sdram.sv deep dive |
| 16| `06_fpga_subsystem/ddr3_architecture.md` | 📋 | Target: Deep | F2H AXI, DDR3 allocation |
| 17| `07_fpga_cores_architecture/template_walkthrough.md` | 📋 | Target: Deep | Core developer entry point |
| 18| `07_fpga_cores_architecture/build/overview.md` | 📋 | Target: Deep | Quartus, QPF → RBF pipeline, CI automation |
| 19| `08_fpga_cores_catalog/arcade_and_mra.md` | 📋 | Target: Deep | MRA XML, ROM merging, DIP switches |
| 20| `08_fpga_cores_catalog/minimig.md` | 📋 | Target: Deep | Cross-link to Amiga Bootcamp |
| 21| `08_fpga_cores_catalog/ao486.md` | 📋 | Target: Deep | 486SX, IDE DMA, VGA, Sound Blaster |
| 22| `08_fpga_cores_catalog/nes.md` | 📋 | Target: Adequate | Mappers, PPU/APU, iNES/NES 2.0 |
| 23| `08_fpga_cores_catalog/snes.md` | 📋 | Target: Adequate | Coprocessors, 5A22/S-PPU/S-SMP |
| 24| `08_fpga_cores_catalog/genesis.md` | 📋 | Target: Adequate | 68000+Z80, VDP, YM2612, Mega CD, 32X |
| 25| `08_fpga_cores_catalog/psx.md` | 📋 | Target: Adequate | R3000A, GTE, GPU, MDEC, CD-ROM |
| 26| `08_fpga_cores_catalog/n64.md` | 📋 | Target: Adequate | VR4300, RDP/RSP, 8MB RDRAM |

## Phase 4 — Peripherals & I/O (The Interfaces)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 27| `09_video_audio/hdmi_scaler.md` | 📋 | Target: Deep | ascal.vhd polyphase scaler |
| 28| `09_video_audio/audio_pipeline.md` | 📋 | Target: Deep | DAC, I2S, HDMI audio, filters |
| 29| `09_video_audio/analog_video.md` | 📋 | Target: Adequate | VGA, composite, S-Video, CRT |
| 30| `10_input_devices/snac_llapi.md` | 📋 | Target: Deep | Direct controller wiring to FPGA |
| 31| `11_storage/file_transfer.md` | 🔀 | Target: Adequate | VHD mounting, SD card |
| 32| `12_networking/wifi_setup.md` | 📋 | Target: Adequate | WiFi, SSH, FTP, Samba |

## Phase 5 — Features, Ecosystem & Reference

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 33| `13_save_states/save_state_architecture.md` | 📋 | Target: Deep | DDR3 snapshot/restore, rewind |
| 34| `14_extensions/retroachievements.md` | 📋 | Target: Deep | odelot fork, DDRAM bridge, rcheevos |
| 35| `14_extensions/cheats.md` | 📋 | Target: Adequate | Cheat engine, .cht format |
| 36| `14_extensions/mra_format.md` | 📋 | Target: Deep | MRA XML schema |
| 37| `15_ecosystem/README.md` | 🔀 | Target: Adequate | Project catalog index: alternative platforms, commercial products, tools, content |
| 37a| `15_ecosystem/update_scripts.md` | 📋 | Target: Adequate | update_all, downloader |
| 38| `16_advanced_topics/mistex.md` | 📋 | Target: Adequate | SBC+FPGA alternative platform port |
| 39| `16_advanced_topics/analogue_comparison.md` | 📋 | Target: Adequate | openFPGA vs MiSTer |
| 40| `17_references/uio_command_reference.md` | 🔀 | Target: Adequate | Command reference |

---

## Content Migration Tracker

Files from `/Volumes/TB4-4Tb/Projects/mister/doc/existing/` to be migrated:

| Source | Target | Status | Notes |
|---|---|---|---|
| `overview.md` | `00_overview/platform_architecture.md` | ✅ | Replaced by new platform architecture |
| `hps-bus.md` | `06_fpga_subsystem/hps_bus.md` | 🔀 | Copied and breadcrumb added |
| `spi-protocol.md` | `06_fpga_subsystem/spi_protocol.md` | 🔀 | Copied and breadcrumb added |
| `core-configuration.md` | `05_configuration/core_configuration_legacy.md` | 🔀 | Copied, needs split to conf_str.md + mister_ini.md |
| `command-reference.md` | `17_references/uio_command_reference.md` | 🔀 | Copied and breadcrumb added |
| `memory-map.md` | `02_hardware_platforms/memory_map.md` | 🔀 | Copied and breadcrumb added |
| `fpga-loading.md` | `06_fpga_subsystem/fpga_loading.md` | 🔀 | Copied and breadcrumb added |
| `keyboard.md` | `10_input_devices/keyboard.md` | 🔀 | Copied and breadcrumb added |
| `mouse.md` | `10_input_devices/mouse.md` | 🔀 | Copied and breadcrumb added |
| `joystick.md` | `10_input_devices/joystick.md` | 🔀 | Copied and breadcrumb added |
| `fdd.md` | `11_storage/floppy_emulation.md` | 🔀 | Copied and breadcrumb added |
| `hdd-ide.md` | `11_storage/hdd_ide_emulation.md` | 🔀 | Copied and breadcrumb added |
| `file-transfer.md` | `11_storage/file_transfer.md` | 🔀 | Copied and breadcrumb added |
| `osd.md` | `05_configuration/osd.md` | 🔀 | Copied and breadcrumb added |
| Linux/Buildroot docs | `03_hps_linux/` | 🔀 | Copied |
| U-Boot patches | `03_hps_linux/uboot_patches/` | 🔀 | Copied |

---

## Priority Matrix

```
                    IMPACT
              Low         High
         ┌───────────┬───────────┐
   Low   │ Phase 4   │ Phase 5   │
EFFORT   │ Peripherals│ Extensions│
         ├───────────┼───────────┤
   High  │ Phase 3   │ Phase 1+2 │
         │ FPGA Core │ Foundation│
         └───────────┴───────────┘

Build order: Phase 1 → Migration → Phase 2 → Phase 3 → Phase 4 → Phase 5
```
