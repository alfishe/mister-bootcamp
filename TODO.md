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

## Tier 0 — Overview & Entry Points

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 0a | `00_overview/platform_architecture.md` | 📋 | Target: Deep | Full system diagrams, layered slices, competitive comparison |
| 0b | `00_overview/start_retro_user.md` | 📋 | Target: Adequate | Retro gamer onboarding, hardware shopping, first core load |
| 0c | `00_overview/start_linux_dev.md` | 📋 | Target: Adequate | Linux dev path: SSH, cross-compile, HPS binary, custom images |
| 0d | `00_overview/start_fpga_dev.md` | 📋 | Target: Adequate | FPGA dev path: Template core, Quartus, build→deploy→test |

## Tier 1 — Platform Foundation

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 1 | `01_hardware_platform/de10_nano.md` | 📋 | Target: Deep | Board-level reference |
| 2 | `01_hardware_platform/addon_boards.md` | 📋 | Target: Adequate | SDRAM, IO board, USB hub, Direct Video |
| 3 | `03_core_architecture/template_walkthrough.md` | 📋 | Target: Deep | Core developer entry point |
| 4 | `02_framework/hps_io_module.md` | 📋 | Target: Deep | Central framework module |
| 5 | `02_framework/sys_top.md` | 📋 | Target: Deep | Top-level hardware abstraction |
| 6 | `04_hps_binary/architecture.md` | 📋 | Target: Deep | Main_MiSTer module map |
| 7 | `06_video_audio/hdmi_scaler.md` | 📋 | Target: Deep | ascal.vhd polyphase scaler |
| 8 | `06_video_audio/audio_pipeline.md` | 📋 | Target: Deep | DAC, I2S, HDMI audio, filters |
| 9 | `11_save_states/save_state_architecture.md` | 📋 | Target: Deep | DDR3 snapshot/restore, rewind |

## Tier 2 — Per-Core Documentation

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 10 | `05_core_catalog/minimig.md` | 📋 | Target: Deep | Cross-link to Amiga Bootcamp (public GitHub) |
| 11 | `05_core_catalog/ao486.md` | 📋 | Target: Deep | 486SX, IDE DMA, VGA, Sound Blaster |
| 12 | `05_core_catalog/nes.md` | 📋 | Target: Adequate | Mappers, PPU/APU, iNES/NES 2.0 |
| 13 | `05_core_catalog/snes.md` | 📋 | Target: Adequate | Coprocessors, 5A22/S-PPU/S-SMP |
| 14 | `05_core_catalog/genesis.md` | 📋 | Target: Adequate | 68000+Z80, VDP, YM2612, Mega CD, 32X |
| 15 | `05_core_catalog/psx.md` | 📋 | Target: Adequate | R3000A, GTE, GPU, MDEC, CD-ROM |
| 16 | `05_core_catalog/n64.md` | 📋 | Target: Adequate | VR4300, RDP/RSP, 8MB RDRAM |
| 17 | `05_core_catalog/arcade_and_mra.md` | 📋 | Target: Deep | MRA XML, ROM merging, DIP switches |

## Tier 3 — Platform Extensions

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 18 | `13_extensions/retroachievements.md` | 📋 | Target: Deep | odelot fork, DDRAM bridge, rcheevos |
| 19 | `13_extensions/cheats.md` | 📋 | Target: Adequate | Cheat engine, .cht format |
| 20 | `13_extensions/mra_format.md` | 📋 | Target: Deep | MRA XML schema |
| 21 | `12_networking_remote/wifi_setup.md` | 📋 | Target: Adequate | WiFi, SSH, FTP, Samba |
| 22 | `14_community_ecosystem/update_scripts.md` | 📋 | Target: Adequate | update_all, downloader |
| 23 | `09_configuration/mister_ini_guide.md` | 📋 | Target: Adequate | User-facing INI guide |
| 24 | `07_input_devices/snac_llapi.md` | 📋 | Target: Deep | Direct controller wiring to FPGA |
| 25 | `06_video_audio/analog_video.md` | 📋 | Target: Adequate | VGA, composite, S-Video, CRT |

## Tier 4 — Advanced Topics

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 26 | `16_advanced_topics/custom_linux.md` | 📋 | Target: Deep | Buildroot, kernel, U-Boot |
| 27 | `16_advanced_topics/mistex.md` | 📋 | Target: Adequate | Xilinx/Zynq port |
| 28 | `16_advanced_topics/analogue_comparison.md` | 📋 | Target: Adequate | openFPGA vs MiSTer |
| 29 | `02_framework/sdram_controller.md` | 📋 | Target: Deep | sdram.sv deep dive |
| 30 | `02_framework/ddr3_architecture.md` | 📋 | Target: Deep | F2H AXI, DDR3 allocation |

## Tier 5 — Build System Documentation

### FPGA Build (Quartus → RBF)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 31 | `03_core_architecture/build/overview.md` | 📋 | Target: Deep | QPF → RBF pipeline |
| 32 | `03_core_architecture/build/native_linux.md` | 📋 | Target: Adequate | Quartus 17.1 on Ubuntu |
| 33 | `03_core_architecture/build/native_windows.md` | 📋 | Target: Adequate | Quartus 17.1 on Windows |
| 34 | `03_core_architecture/build/docker_x86.md` | 📋 | Target: Adequate | raetro/quartus container |
| 35 | `03_core_architecture/build/docker_apple_silicon.md` | 📋 | Target: Deep | quartus-on-docker project |
| 36 | `03_core_architecture/build/ci_automation.md` | 📋 | Target: Adequate | GitHub Actions |

### HPS Build (ARM Cross-Compile)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 37 | `04_hps_binary/build/overview.md` | 📋 | Target: Deep | ARM toolchain, Makefile |
| 38 | `04_hps_binary/build/native_linux.md` | 📋 | Target: Adequate | arm-linux-gnueabihf-g++ |
| 39 | `04_hps_binary/build/docker_cross.md` | 📋 | Target: Adequate | Docker cross-compile |
| 40 | `04_hps_binary/build/windows_wsl2.md` | 📋 | Target: Adequate | WSL2 workflow |
| 41 | `04_hps_binary/build/macos.md` | 📋 | Target: Adequate | macOS + Apple Silicon |

---

## Content Migration Tracker

Files from `/Volumes/TB4-4Tb/Projects/mister/doc/existing/` to be migrated:

| Source | Target | Status | Notes |
|---|---|---|---|
| `overview.md` | `00_overview/platform_architecture.md` (seed) | 📋 | Expand with diagrams |
| `hps-bus.md` | `02_framework/hps_bus.md` | 📋 | Add breadcrumb, FPGA KB links |
| `spi-protocol.md` | `02_framework/spi_protocol.md` | 📋 | Add breadcrumb |
| `core-configuration.md` | `09_configuration/conf_str.md` + `mister_ini.md` | 📋 | Split into 2 articles |
| `command-reference.md` | `15_references/uio_command_reference.md` | 📋 | Add breadcrumb |
| `memory-map.md` | `01_hardware_platform/memory_map.md` | 📋 | Add breadcrumb, FPGA KB links |
| `fpga-loading.md` | `02_framework/fpga_loading.md` | 📋 | Add breadcrumb |
| `keyboard.md` | `07_input_devices/keyboard.md` | 📋 | Add breadcrumb |
| `mouse.md` | `07_input_devices/mouse.md` | 📋 | Add breadcrumb |
| `joystick.md` | `07_input_devices/joystick.md` | 📋 | Add breadcrumb |
| `fdd.md` | `08_storage/floppy_emulation.md` | 📋 | Add breadcrumb |
| `hdd-ide.md` | `08_storage/hdd_ide_emulation.md` | 📋 | Add breadcrumb |
| `file-transfer.md` | `08_storage/file_transfer.md` | 📋 | Add breadcrumb |
| `osd.md` | `09_configuration/osd.md` | 📋 | Add breadcrumb, expand |
| Linux/Buildroot docs | `10_linux_system/buildroot_custom_image.md` | 📋 | Merge 2 files |
| U-Boot patches | `10_linux_system/uboot_patches/` | 📋 | Copy as-is |

---

## Priority Matrix

```
                    IMPACT
              Low         High
         ┌───────────┬───────────┐
   Low   │ Tier 2    │ Tier 3    │
EFFORT   │ Per-core  │ Extensions│
         ├───────────┼───────────┤
   High  │ Tier 4    │ Tier 0+1  │
         │ Advanced  │ Foundation│
         └───────────┴───────────┘

Build order: Tier 0 → Migration → Tier 1 → Tier 5 → Tier 3 → Tier 2 → Tier 4
```
