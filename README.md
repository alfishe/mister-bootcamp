# MiSTer FPGA — The Developer's Source of Truth

**From silicon to scanlines**

> *Everything the wiki summarises but never explains — grounded in Verilog, C, and the actual hardware.*
>
> A comprehensive technical reference for the MiSTer FPGA platform: DE10-Nano hardware, the `sys/` framework, HPS binary internals, core development, video/audio pipeline, input handling, RetroAchievements integration, build tooling, and the embedded Linux stack. Written for people who want to understand what's actually happening inside the box.

---

## Quick Start

| You are... | Start here |
|---|---|
| 🎮 **Retro gamer / new to MiSTer** | [Getting Started — Retro Gaming](00_overview/start_retro_user.md) |
| 🐧 **Software / Linux developer** | [Getting Started — Linux Dev](00_overview/start_linux_dev.md) |
| ⚡ **FPGA / HDL engineer** | [Getting Started — FPGA Dev](00_overview/start_fpga_dev.md) |
| 🔬 **Want the full architecture** | [Platform Architecture Overview](00_overview/platform_architecture.md) |

---

## What's Inside

| Layer | Coverage |
|---|---|
| **Overview** | Platform architecture, competitive landscape, persona-based entry points |
| **Hardware Platform** | DE10-Nano board, Cyclone V SoC, SDRAM/IO/USB add-on boards, physical memory map |
| **Framework** | `sys_top.v`, `hps_io.sv`, HPS_BUS 49-bit bus, SPI-over-GPO, FPGA loading, SDRAM controller, DDR3 architecture |
| **Core Architecture** | Template walkthrough, core development guide, Quartus build system (native + Docker + Apple Silicon) |
| **HPS Binary** | `Main_MiSTer` C++ codebase architecture, ARM cross-compilation (Linux/Windows/macOS/Docker) |
| **Core Catalog** | Per-system deep dives: Minimig (Amiga), ao486 (x86), NES, SNES, Genesis, PSX, N64, Arcade/MRA |
| **Video & Audio** | HDMI scaler (ascal), analog output (VGA/composite/S-Video), audio pipeline, filters |
| **Input Devices** | Keyboard, mouse, joystick, SNAC, LLAPI — from USB HID through the HPS to FPGA registers |
| **Storage** | Floppy/HDD/CD-ROM emulation, SD card file I/O, ROM/disk image management |
| **Configuration** | CONF_STR (RTL), MiSTer.ini (system), .CFG persistence, OSD menu system |
| **Linux System** | Kernel patches, U-Boot, Buildroot image generation, device tree, filesystem layout, driver model |
| **Save States** | DDR3 snapshot/restore architecture, rewind, per-core implementation |
| **Networking & Remote** | WiFi, SSH, FTP, Samba, remote management |
| **Extensions** | RetroAchievements (odelot fork deep dive), cheats, MRA format |
| **Community Ecosystem** | Update scripts, distributions, custom firmware |
| **References** | UIO opcode table, memory map, configuration variable reference |
| **Advanced Topics** | Custom Linux images, MiSTeX (Xilinx port), Analogue Pocket comparison |

---

## Sources

| Reference | URL |
|---|---|
| MiSTer-devel (main org) | https://github.com/MiSTer-devel |
| Main_MiSTer (HPS binary) | https://github.com/MiSTer-devel/Main_MiSTer |
| Template_MiSTer (core template + sys/) | https://github.com/MiSTer-devel/Template_MiSTer |
| Linux-Kernel_MiSTer | https://github.com/MiSTer-devel/Linux-Kernel_MiSTer |
| u-boot_MiSTer | https://github.com/MiSTer-devel/u-boot_MiSTer |
| mr-fusion (SD image builder) | https://github.com/MiSTer-devel/mr-fusion |
| RetroAchievements fork | https://github.com/odelot/Main_MiSTer |
| Intel Cyclone V HPS TRM | https://www.intel.com/content/www/us/en/docs/programmable/683126/ |
| DE10-Nano User Manual | https://www.terasic.com.tw/cgi-bin/page/archive.pl?No=1046 |

## Cross-References

This knowledge base is part of a documentation suite:

| Repository | Scope |
|---|---|
| [Amiga Bootcamp](https://github.com/alfishe/amiga-bootcamp) | Amiga hardware, AmigaOS, custom chips, software — the reference for Minimig core internals |
| [FPGA Knowledge Base](https://github.com/alfishe/fpga-bootcamp) | General FPGA: Cyclone V architecture, AXI buses, timing, HDL, vendor toolchains |

---

## Documentation Map

### 00 — Overview
| File | Topic |
|---|---|
| [platform_architecture.md](00_overview/platform_architecture.md) | Full system architecture: Cyclone V dual-world, HPS↔FPGA pipeline, data flow diagrams, competitive comparison |
| [start_retro_user.md](00_overview/start_retro_user.md) | Getting started for retro gamers: hardware, setup, cores, ROMs, MiSTer.ini, RetroAchievements |
| [start_linux_dev.md](00_overview/start_linux_dev.md) | Getting started for Linux/software developers: SSH, cross-compilation, HPS binary, custom images |
| [start_fpga_dev.md](00_overview/start_fpga_dev.md) | Getting started for FPGA engineers: Cyclone V recap, sys/ framework, Template core, Quartus builds |

### 01 — Hardware Platform
| File | Topic |
|---|---|
| [de10_nano.md](02_hardware_platforms/de10_nano.md) | DE10-Nano board reference: Cyclone V 5CSEBA6, DDR3, ADC, HDMI TX, USB, Arduino header |
| [addon_boards.md](02_hardware_platforms/addon_boards.md) | SDRAM board (active/passive), IO board (VGA, serial), USB Hub, Direct Video adapter, MT32-Pi |
| [memory_map.md](02_hardware_platforms/memory_map.md) | Cyclone V SoC physical address map as used by MiSTer |

### 02 — Framework (`sys/`)
| File | Topic |
|---|---|
| [sys_top.md](06_fpga_subsystem/sys_top.md) | Top-level hardware abstraction: PLL tree, bridge wiring, GPO/GPI, HDMI/VGA muxing |
| [hps_io_module.md](06_fpga_subsystem/hps_io_module.md) | `hps_io.sv` deep dive: port map, parameters, opcode dispatch, status register |
| [hps_bus.md](06_fpga_subsystem/hps_bus.md) | HPS_BUS[48:0] — the 49-bit parallel bus between `sys_top.v` and `hps_io.sv` |
| [spi_protocol.md](06_fpga_subsystem/spi_protocol.md) | SPI-over-GPO: software-emulated SPI via FPGA Manager registers |
| [fpga_loading.md](06_fpga_subsystem/fpga_loading.md) | RBF bitstream loading: FPGA Manager state machine, bridge teardown/restore, warm reboot |
| [sdram_controller.md](06_fpga_subsystem/sdram_controller.md) | `sdram.sv`: bank interleaving, refresh, dual-port, active/passive board detection |
| [ddr3_architecture.md](06_fpga_subsystem/ddr3_architecture.md) | F2H AXI bridge, DDR3 port allocation (scaler, save states, RA mirror), bandwidth budget |

### 03 — Core Architecture
| File | Topic |
|---|---|
| [template_walkthrough.md](07_fpga_cores_architecture/template_walkthrough.md) | Step-by-step guide to building a core from `Template_MiSTer` |
| [build/overview.md](07_fpga_cores_architecture/build/overview.md) | FPGA build pipeline: QPF → Synthesis → Fitter → Assembler → RBF |
| [build/native_linux.md](07_fpga_cores_architecture/build/native_linux.md) | Quartus 17.1 Lite on Ubuntu/Debian |
| [build/native_windows.md](07_fpga_cores_architecture/build/native_windows.md) | Quartus 17.1 on Windows (GUI + CLI) |
| [build/docker_x86.md](07_fpga_cores_architecture/build/docker_x86.md) | Docker build with `raetro/quartus:17.1` |
| [build/docker_apple_silicon.md](07_fpga_cores_architecture/build/docker_apple_silicon.md) | Quartus on Apple Silicon via Rosetta 2 (quartus-on-docker project) |
| [build/ci_automation.md](07_fpga_cores_architecture/build/ci_automation.md) | GitHub Actions CI for core compilation |

### 04 — HPS Binary (`Main_MiSTer`)
| File | Topic |
|---|---|
| [architecture.md](04_hps_binary/architecture.md) | Module map, main loop, scheduler, key data structures |
| [build/overview.md](04_hps_binary/build/overview.md) | ARM cross-compilation toolchain and Makefile |
| [build/native_linux.md](04_hps_binary/build/native_linux.md) | `arm-linux-gnueabihf-g++` on Ubuntu |
| [build/docker_cross.md](04_hps_binary/build/docker_cross.md) | Docker-based ARM cross-compile (all platforms) |
| [build/windows_wsl2.md](04_hps_binary/build/windows_wsl2.md) | WSL2 cross-compilation workflow |
| [build/macos.md](04_hps_binary/build/macos.md) | macOS + Apple Silicon ARM cross-compile |

### 05 — Core Catalog
| File | Topic |
|---|---|
| [minimig.md](08_fpga_cores_catalog/minimig.md) | Amiga OCS/ECS/AGA — chipset mapping, hps_ext.v, SDRAM vs DDR3. → [Amiga Bootcamp](https://github.com/alfishe/amiga-bootcamp) |
| [ao486.md](08_fpga_cores_catalog/ao486.md) | 486SX soft CPU, IDE/CD-ROM DMA, VGA, Sound Blaster |
| [nes.md](08_fpga_cores_catalog/nes.md) | NES: mappers, PPU/APU, iNES/NES 2.0 |
| [snes.md](08_fpga_cores_catalog/snes.md) | SNES: GSU/SA-1/SDD-1/DSP coprocessors |
| [genesis.md](08_fpga_cores_catalog/genesis.md) | Genesis: 68000+Z80, VDP, YM2612, Mega CD, 32X |
| [psx.md](08_fpga_cores_catalog/psx.md) | PSX: R3000A, GTE, GPU, MDEC, CD-ROM, CHD |
| [n64.md](08_fpga_cores_catalog/n64.md) | N64: VR4300, RDP/RSP, 8MB RDRAM |
| [arcade_and_mra.md](08_fpga_cores_catalog/arcade_and_mra.md) | Arcade cores: MRA XML, ROM merging, DIP switches, MAME compatibility |

### 06 — Video & Audio
| File | Topic |
|---|---|
| [hdmi_scaler.md](09_video_audio/hdmi_scaler.md) | `ascal.vhd`: polyphase scaler, DDR3 framebuffer, filter loading |
| [audio_pipeline.md](09_video_audio/audio_pipeline.md) | Sigma-delta DAC, I2S, HDMI audio, filters, 96 kHz mode |
| [analog_video.md](09_video_audio/analog_video.md) | IO board analog output, direct video, CRT considerations |

### 07 — Input Devices
| File | Topic |
|---|---|
| [keyboard.md](10_input_devices/keyboard.md) | PS/2 keyboard emulation via USB HID |
| [mouse.md](10_input_devices/mouse.md) | PS/2 mouse emulation |
| [joystick.md](10_input_devices/joystick.md) | Joystick/gamepad handling, analog sticks, autofire |
| [snac_llapi.md](10_input_devices/snac_llapi.md) | SNAC & LLAPI: direct controller wiring to FPGA fabric |

### 08 — Storage
| File | Topic |
|---|---|
| [floppy_emulation.md](11_storage/floppy_emulation.md) | Floppy disk emulation: ADF/DSK images, sector I/O protocol |
| [hdd_ide_emulation.md](11_storage/hdd_ide_emulation.md) | HDD/IDE emulation: VHD images, DMA path |
| [file_transfer.md](11_storage/file_transfer.md) | File I/O protocol: ROM loading, save file upload/download |

### 09 — Configuration
| File | Topic |
|---|---|
| [conf_str.md](05_configuration/conf_str.md) | CONF_STR: RTL-embedded OSD menu declaration, status[127:0] register, parse_config() |
| [mister_ini.md](05_configuration/mister_ini.md) | MiSTer.ini: system INI file, section matching, variable reference |
| [mister_ini_guide.md](05_configuration/mister_ini_guide.md) | User-facing MiSTer.ini guide: video modes, filters, per-core overrides, recipes |
| [osd.md](05_configuration/osd.md) | OSD rendering pipeline and menu navigation |

### 10 — Linux System
| Folder / File | Topic |
|---|---|
| [kernel/](10_linux_system/kernel/) | Linux kernel patches, configuration, Cyclone V SoC support, FPGA Manager driver |
| [uboot/](10_linux_system/uboot/) | U-Boot patches, boot sequence, warm/cold reboot, OCRAM handoff |
| [buildroot/](10_linux_system/buildroot/) | Buildroot-based image generation, package selection, custom overlays |
| [device_tree/](10_linux_system/device_tree/) | Device tree for DE10-Nano, HPS↔FPGA bridge bindings, overlay compilation |
| [filesystem/](10_linux_system/filesystem/) | SD card layout (`/media/fat/`), partition scheme, MiSTer directory conventions |
| [scripts/](10_linux_system/scripts/) | System scripts: startup, networking, USB mount, core switching |
| [drivers/](10_linux_system/drivers/) | Kernel driver model: FPGA Manager, USB gadget, I2C, GPIO |

### 11 — Save States
| File | Topic |
|---|---|
| [save_state_architecture.md](13_save_states/save_state_architecture.md) | DDR3 snapshot/restore, SSPI protocol, per-core requirements, rewind |

### 12 — Networking & Remote
| File | Topic |
|---|---|
| [wifi_setup.md](12_networking/wifi_setup.md) | USB WiFi, wpa_supplicant, SSH, FTP, Samba, web console |

### 13 — Extensions
| File | Topic |
|---|---|
| [retroachievements.md](14_extensions/retroachievements.md) | RetroAchievements: odelot fork, DDRAM bridge, rcheevos engine, per-console hashing |
| [cheats.md](14_extensions/cheats.md) | Cheat engine, .cht format, GameShark/Action Replay conversion |
| [mra_format.md](14_extensions/mra_format.md) | MRA XML schema, ROM merging, MAME set management |

### 14 — Community Ecosystem
| File | Topic |
|---|---|
| [update_scripts.md](15_ecosystem/update_scripts.md) | `update_all`, `downloader`, custom database URLs |

### 15 — References
| File | Topic |
|---|---|
| [uio_command_reference.md](17_references/uio_command_reference.md) | Complete UIO/FIO opcode table with bit-level detail |

### 16 — Advanced Topics
| File | Topic |
|---|---|
| [custom_linux.md](16_advanced_topics/custom_linux.md) | Building custom MiSTer Linux images from scratch |
| [mistex.md](16_advanced_topics/mistex.md) | MiSTeX: porting MiSTer framework to Xilinx/Zynq |
| [analogue_comparison.md](16_advanced_topics/analogue_comparison.md) | Analogue Pocket openFPGA vs MiSTer: architecture & feature comparison |
