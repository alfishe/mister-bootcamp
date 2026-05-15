# MiSTer FPGA Knowledge Base — TODO & Gap Analysis

> **Last updated**: 2026-05-07
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
| 15| `06_fpga_subsystem/sdram_controller.md` | ✅ | Deep | Three SDRAM controller variants, state machines, init, refresh, 8/16-bit mode, dual-chip, burst, cache snoop |
| 16| `06_fpga_subsystem/ddr3_architecture.md` | ✅ | Deep | F2SDRAM bridge, sysmem_lite, f2sdram_safe_terminator, ddr_svc arbiter, ddram.sv wrapper, ddram_ctrl.v cached controller, HPS bridge management |
| 16a| `06_fpga_subsystem/sdram_timing_theory.md` | ✅ | Deep | altddio_out clock inversion, write/read path timing budgets, PLL phase shift, runtime reconfiguration, frequency-specific requirements, MemTest validation |
| 16b| `06_fpga_subsystem/memory_controllers.md` | ✅ | Adequate | SDRAM vs DDR3 architecture, controller variants table, DDRAM wrapper, dual-path Minimig diagram, DDR3 contention profiles |
| 16c| `06_fpga_subsystem/fpga_compilation_guide.md` | ✅ | Adequate | Quartus Q13/Q17/Q20+ comparison, SDC constraint examples, CDC false paths, seed sweeping, DSE II, timing failure patterns |
| 16d| `06_fpga_subsystem/fpga_debugging_tools.md` | ✅ | Adequate | SignalTap II BRAM cost table, io_dout telemetry, UART printf, status word, MemTest as validation, GPIO oscilloscope probing |
| 16e| `06_fpga_subsystem/fpga_performance_metrics.md` | ✅ | Adequate | Verified MemTest compilation data, framework overhead analysis, M10K budget, console/computer/arcade utilization tables, power/thermal |
| 16f| `06_fpga_subsystem/input_latency_and_snac.md` | ✅ | Deep | Three input paths with Mermaid diagram, USB latency budget, SNAC sub-µs budget, LLAPI comparison, per-core protocol summary, input-to-display measurement
| 17| `07_fpga_cores_architecture/template_walkthrough.md` | ✅ | Deep | Core developer entry point: repo structure, emu module, CONF_STR, video/audio/memory contracts, common pitfalls |
| 18| `07_fpga_cores_architecture/build/overview.md` | ✅ | Deep | Quartus compilation pipeline: QPF/QSF structure, TCL automation, build_id.tcl, RBF generation, CI/CD, seed sweeping, Q13 vs Q17 |
| 19| `08_fpga_cores_catalog/arcade_and_mra.md` | ✅ | Deep | MRA XML schema, ROM assembly pipeline, interleave/map, DIP switches, NVRAM, MGL format, Joteco framework |
| 20| `08_fpga_cores_catalog/minimig.md` | ✅ | Deep | Amiga OCS/ECS/AGA emulation, DDR3 Fast RAM via ddram_ctrl.v, RTG, a314 shared folder, CD32 |
| 21| `08_fpga_cores_catalog/ao486.md` | ✅ | Deep | 486SX soft CPU, Sound Blaster 16/OPL3, SVGA, IDE/CD-ROM, UART networking, DOS/Windows compat |
| 22| `08_fpga_cores_catalog/nes.md` | ✅ | Adequate | 2A03 CPU+APU, 2C02 PPU, 200+ mappers, FDS, expansion audio, iNES/NES 2.0 |
| 23| `08_fpga_cores_catalog/snes.md` | ✅ | Adequate | 5A22 CPU, dual S-PPU, S-SMP, DSP/Super FX/SA-1/S-DD1/CX4/SPC7110 coprocessors, MSU-1 |
| 24| `08_fpga_cores_catalog/genesis.md` | ✅ | Adequate | 68000+Z80, YM7101 VDP, YM2612/YM3438, SVP, Nuked MD alternative |
| 25| `08_fpga_cores_catalog/psx.md` | ✅ | Adequate | R3000A, GTE, GPU, SPU, MDEC, CD-ROM, DualShock/NeGcon/GunCon input |
| 26| `08_fpga_cores_catalog/n64.md` | ✅ | Adequate | VR4300i, RSP+RDP, RDRAM, SDRAM requirements, error diagnostics |
| 41| `08_fpga_cores_catalog/gba.md` | ✅ | Adequate | GBA: ARM7TDMI CPU, 240×160 LCD, 4 GB ROM, save types, Robert Peip core |
| 42| `08_fpga_cores_catalog/gb_gbc.md` | ✅ | Adequate | Game Boy / Game Boy Color: DMG Sharp LR35902, 4-shade LCD, SGB, GBC double-speed |
| 43| `08_fpga_cores_catalog/c64.md` | ✅ | Adequate | C64: 6510 CPU, VIC-II, SID 6581/8580, PLA, CIA, cartridge types |
| 44| `08_fpga_cores_catalog/atari_st.md` | ✅ | Adequate | Atari ST/STe: 68000, YM2149, Shifter, Blitter (Mega ST), MSA/ST disk images |
| 45| `08_fpga_cores_catalog/saturn.md` | ✅ | Adequate | Sega Saturn: dual SH-2, SCU DSP, VDP1/VDP2, SMPC, CD block, cartridge |
| 46| `08_fpga_cores_catalog/pc_engine.md` | ✅ | Adequate | PC Engine / TG-16 / SuperGrafx: HuC6280 CPU, HuC6260/HuC6270 VDC, CD-ROM² |
| 47| `08_fpga_cores_catalog/msx.md` | ✅ | Adequate | MSX/MSX2/MSX2+/TurboR: Z80, TMS9918/V9938/V9958 VDP, PSG+SCC+FM-PAC audio |
| 48| `08_fpga_cores_catalog/neogeo.md` | ✅ | Adequate | Neo Geo: 68000, Z80, LSPC/B1/C1 chipset, 330 ROM slots, memory card |

## Phase 4 — Peripherals & I/O (The Interfaces)

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 27| `09_video_audio/hdmi_scaler.md` | ✅ | Deep | Superseded by `ascal_deep_dive.md` (67.7 KB Deep article) |
| 28| `09_video_audio/audio_pipeline.md` | ✅ | Deep | Superseded by `audio_pipeline_deep_dive.md` — source-grounded Deep upgrade with audio_out.v analysis |
| 29| `09_video_audio/analog_video.md` | ✅ | Deep | Superseded by `analog_direct_video_architecture.md` — source-grounded Deep upgrade with vga_out.sv + yc_out.sv |
| 30| `10_input_devices/snac_llapi.md` | ✅ | Deep | SNAC hardware interface, per-core OEM protocols (NES/SNES/Genesis/PSX/N64), LLAPI wireless, 74LVC245 level shifting |
| 31| `11_storage/file_transfer.md` | 🔀 | Target: Adequate | VHD mounting, SD card |
| 32| `12_networking/wifi_setup.md` | ✅ | Adequate | WiFi adapter setup, SSH/FTP/SFTP, Samba, CIFS/NFS mount, Tailscale VPN |
| 49| `05_configuration/crt_setup.md` | ✅ | Adequate | CRT/analog video setup: 15 kHz, Direct Video, IO board VGA, mister.ini video_mode, scaler bypass |
| 50| `10_input_devices/controller_setup.md` | ✅ | Adequate | Controller setup & mapping: 8bitdo pairing, USB hub quirks, OSD mapping, gamepad DB |
| 51| `12_networking/remote_management.md` | ✅ | Adequate | Remote management: mrext web UI, headless operation, phone control, API |

## Phase 5 — Features, Ecosystem & Reference

| # | Article | Status | Quality | Notes |
|---|---|---|---|---|
| 33| `13_save_states/save_state_architecture.md` | ✅ | Deep | DDR3 shared-memory save states, CONF_STR SS tag, process_ss() 4-slot polling, sector I/O alternative, per-core implementations |
| 34| `14_extensions/retroachievements.md` | ✅ | Deep | odelot fork, DDRAM bridge (Full Mirror / Selective Address), rcheevos SDK, 14 supported cores |
| 35| `14_extensions/cheats.md` | ✅ | Adequate | Cheat engine: ZIP/CRC discovery, Game Genie/PAR formats, FIO index 255 delivery, arcade MRA cheats, N64 cheat system, lazy loading |
| 36| `14_extensions/mra_format.md` | ✅ | Deep | Superseded by `arcade_and_mra.md` (620 lines Deep article in 08 catalog) |
| 37| `15_ecosystem/README.md` | 🔀 | Target: Adequate | Project catalog index: alternative platforms, commercial products, tools, content |
| 52| `15_ecosystem/software_tools.md` | ✅ | Adequate | Ecosystem tools: mrext suite, software ports (ScummVM/DOSBox/PrBoom), system utilities |
| 37a| `15_ecosystem/update_scripts.md` | ✅ | Adequate | Update All settings UI, Downloader JSON manifest, PC Launcher, file download flow |
| 38| `16_advanced_topics/mistex.md` | ✅ | Adequate | SBC+FPGA alternative: SPI-over-GPIO vs HPS bus, supported hardware, core compatibility |
| 39| `16_advanced_topics/analogue_comparison.md` | ✅ | Adequate | openFPGA vs MiSTer: architecture, core ecosystem, developer experience, latency |
| 40| `17_references/uio_command_reference.md` | ✅ | Adequate | Protocol diagram, full opcode tables, joystick bit mapping, CONF_STR format, cross-refs |

---

## Content Migration Tracker

Files from `/Volumes/TB4-4Tb/Projects/mister/doc/existing/` to be migrated:

| Source | Target | Status | Notes |
|---|---|---|---|
| `overview.md` | `00_overview/platform_architecture.md` | ✅ | Replaced by new platform architecture |
| `hps-bus.md` | `06_fpga_subsystem/hps_bus.md` | ✅ | Superseded by `hps_io_module.md` and `hps_bridge_reference.md` |
| `spi-protocol.md` | `06_fpga_subsystem/spi_protocol.md` | ✅ | Superseded by `hps_io_module.md` and `hps_bridge_reference.md` |
| `core-configuration.md` | `05_configuration/` | ✅ | Split into `conf_str.md` and `mister_ini_guide.md` |
| `command-reference.md` | `17_references/uio_command_reference.md` | ✅ | Expanded with protocol diagram, joystick mapping, cross-refs |
| `memory-map.md` | `02_hardware_platforms/memory_map.md` | ✅ | Breadcrumb + cross-refs added |
| `fpga-loading.md` | `06_fpga_subsystem/fpga_loading.md` | ✅ | Breadcrumb + cross-refs added |
| `keyboard.md` | `10_input_devices/keyboard.md` | ✅ | Breadcrumb + cross-refs added |
| `mouse.md` | `10_input_devices/mouse.md` | ✅ | Breadcrumb + cross-refs added |
| `joystick.md` | `10_input_devices/joystick.md` | ✅ | Breadcrumb + cross-refs added |
| `fdd.md` | `11_storage/floppy_emulation.md` | ✅ | Breadcrumb + cross-refs added |
| `hdd-ide.md` | `11_storage/hdd_ide_emulation.md` | ✅ | Breadcrumb + cross-refs added |
| `file-transfer.md` | `11_storage/file_transfer.md` | ✅ | Breadcrumb + cross-refs added |
| `osd.md` | `05_configuration/osd.md` | ✅ | Superseded by new Deep OSD article |
| Linux/Buildroot docs | `03_hps_linux/` | ✅ | Breadcrumbs + cross-refs on buildroot/ articles already present |
| U-Boot patches | `03_hps_linux/uboot_patches/` | ✅ | README index updated with subdirectory table |
| `03_hps_linux/README.md` | `03_hps_linux/README.md` | ✅ | Article table + subdirectory table with descriptions |

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
