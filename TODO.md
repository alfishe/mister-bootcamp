# MiSTer FPGA Knowledge Base — TODO & Gap Analysis

> **Last updated**: 2026-05-06
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
| 6 | `02_hardware_platforms/de10_nano.md` | ✅ | Target: Deep | Physical board reference, specs, pins |
| 7 | `02_hardware_platforms/addon_boards.md` | ✅ | Deep | Physical addons: SDRAM, IO board, USB hub, Direct Video |

## Phase 2 — Host Software (The ARM/HPS Control Plane)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 8 | `03_hps_linux/alternative_distros.md` | ✅ | Deep | Debian, Ubuntu, Arch Linux ARM on MiSTer; MiSTerArch, MOnSieurFPGA |
| 8a| `03_hps_linux/custom_linux.md` | 📋 | Superseded | Covered by buildroot/, kernel/, uboot/, image_build/, alternative_distros.md |
| 9 | `04_hps_binary/build/overview.md` | ✅ | Deep | Main_MiSTer binary architecture, modules, startup sequence |
| 10| `04_hps_binary/build/toolchain.md` | ✅ | Deep | ARM toolchain, Makefile, cross-compile |
| 11| `05_configuration/mister_ini_guide.md` | ✅ | Deep | User-facing INI guide |
| 12| `05_configuration/osd.md` | ✅ | Deep | OSD rendering, input mapping, and alpha blending architecture |

## Phase 3 — Hardware Emulation (The FPGA Plane)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 13| `06_fpga_subsystem/sys_top.md` | ✅ | Deep | Top-level hardware abstraction: ports, submodule map, SSPI command decoder, video/audio/memory pipelines, conditional compilation |
| 14| `06_fpga_subsystem/hps_io_module.md` | ✅ | Deep | `hps_io.sv` command decoder: UIO commands, file I/O engine, joystick/keyboard/mouse, CONF_STR, status word, SD card, EXT_BUS |
| 15| `06_fpga_subsystem/sdram_controller.md` | 📋 | Target: Deep | sdram.sv deep dive |
| 16| `06_fpga_subsystem/ddr3_architecture.md` | ✅ | Deep | F2SDRAM bridge, sysmem_lite, f2sdram_safe_terminator, ddr_svc arbiter, ddram.sv wrapper, ddram_ctrl.v cached controller, HPS bridge management |
| 17| `07_fpga_cores_architecture/template_walkthrough.md` | ✅ | Deep | Core developer entry point: repo structure, emu module, CONF_STR, video/audio/memory contracts, common pitfalls |
| 18| `07_fpga_cores_architecture/build/overview.md` | ✅ | Deep | Quartus compilation pipeline: QPF/QSF structure, TCL automation, build_id.tcl, RBF generation, CI/CD, seed sweeping, Q13 vs Q17 |
| 19| `08_fpga_cores_catalog/arcade_and_mra.md` | ✅ | Deep | MRA XML schema, ROM assembly pipeline, interleave/map, DIP switches, NVRAM, MGL format, Joteco framework |
| 20| `08_fpga_cores_catalog/minimig.md` | ✅ | Deep | Amiga OCS/ECS/AGA emulation, DDR3 Fast RAM via ddram_ctrl.v, RTG, a314 shared folder, CD32 |
| 21| `08_fpga_cores_catalog/ao486.md` | ✅ | Deep | 486SX soft CPU, Sound Blaster 16/OPL3, SVGA, IDE/CD-ROM, UART networking, DOS/Windows compat |
| 22| `08_fpga_cores_catalog/nes.md` | 📋 | Target: Adequate | Mappers, PPU/APU, iNES/NES 2.0 |
| 23| `08_fpga_cores_catalog/snes.md` | 📋 | Target: Adequate | Coprocessors, 5A22/S-PPU/S-SMP |
| 24| `08_fpga_cores_catalog/genesis.md` | 📋 | Target: Adequate | 68000+Z80, VDP, YM2612, Mega CD, 32X |
| 25| `08_fpga_cores_catalog/psx.md` | 📋 | Target: Adequate | R3000A, GTE, GPU, MDEC, CD-ROM |
| 26| `08_fpga_cores_catalog/n64.md` | 📋 | Target: Adequate | VR4300, RDP/RSP, 8MB RDRAM |

## Phase 4 — Peripherals & I/O (The Interfaces)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 27| `09_video_audio/hdmi_scaler.md` | ✅ | Deep | Superseded by `ascal_deep_dive.md` (67.7 KB Deep article) |
| 28| `09_video_audio/audio_pipeline.md` | ✅ | Deep | Superseded by `audio_pipeline_deep_dive.md` — source-grounded Deep upgrade with audio_out.v analysis |
| 29| `09_video_audio/analog_video.md` | ✅ | Deep | Superseded by `analog_direct_video_architecture.md` — source-grounded Deep upgrade with vga_out.sv + yc_out.sv |
| 30| `10_input_devices/snac_llapi.md` | ✅ | Deep | SNAC hardware interface, per-core OEM protocols (NES/SNES/Genesis/PSX/N64), LLAPI wireless, 74LVC245 level shifting |
| 31| `11_storage/file_transfer.md` | 🔀 | Target: Adequate | VHD mounting, SD card |
| 32| `12_networking/wifi_setup.md` | 📋 | Target: Adequate | WiFi, SSH, FTP, Samba |

## Phase 5 — Features, Ecosystem & Reference

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 33| `13_save_states/save_state_architecture.md` | ✅ | Deep | DDR3 shared-memory save states, CONF_STR SS tag, process_ss() 4-slot polling, sector I/O alternative, per-core implementations |
| 34| `14_extensions/retroachievements.md` | 📋 | Target: Deep | odelot fork, DDRAM bridge, rcheevos |
| 35| `14_extensions/cheats.md` | ✅ | Adequate | Cheat engine: ZIP/CRC discovery, Game Genie/PAR formats, FIO index 255 delivery, arcade MRA cheats, N64 cheat system, lazy loading |
| 36| `14_extensions/mra_format.md` | ✅ | Deep | Superseded by `arcade_and_mra.md` (620 lines Deep article in 08 catalog) |
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
| `core-configuration.md` | `05_configuration/` | ✅ | Split into `conf_str.md` and `mister_ini_guide.md` |
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
