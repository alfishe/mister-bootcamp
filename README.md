# MiSTer FPGA — The Developer's Source of Truth

**Hardware you can read. Systems you can rewrite.**

> *Everything the wiki summarises but never explains — grounded in Verilog, C, and the actual hardware.*
>
> A comprehensive technical reference for the MiSTer FPGA platform: DE10-Nano hardware, the `sys/` framework, HPS binary internals, core development, video/audio pipeline, input handling, RetroAchievements integration, build tooling, and the embedded Linux stack. Written for people who want to understand what's actually happening inside the box.

---

## Quick Start

| You are... | What's in it for you | Start here |
|---|---|---|
| 🎮 **Retro gamer** | Cycle-accurate classics, zero input lag | [Retro Gaming](00_overview/getting_started.md#-retro-gaming) |
| 🕹️ **Arcade builder** | MiSTer in a JAMMA cabinet | [Arcade](00_overview/getting_started.md#️-arcade-builder) |
| 📺 **CRT enthusiast** | Real scanlines, native 15kHz | [Analog Video](00_overview/getting_started.md#-crt--analog-video) |
| 🎓 **CS / EE student** | Study real CPUs in synthesizable RTL | [Student Path](00_overview/getting_started.md#-student--learner) |
| 💻 **Software developer** | Hack the ARM Linux control stack | [Software Dev](00_overview/getting_started.md#-software-developer) |
| ⚡ **FPGA engineer** | Write cores on a mature framework | [FPGA Dev](00_overview/getting_started.md#-fpga--hdl-engineer) |
| 🔧 **Tinkerer / maker** | Build, solder, 3D-print, mod | [Tinkerer](00_overview/getting_started.md#-tinkerer--maker) |
| 📡 **Preservationist** | Bit-perfect hardware preservation | [Preservation](00_overview/getting_started.md#-preservationist--archivist) |
| 🔬 **Full architecture** | Complete technical reference | [Platform Architecture](01_system_architecture/platform_architecture.md) |

**New to MiSTer?** Start with the [Platform Overview](00_overview/README.md) — what it is, how it evolved, and why it matters.

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

| Repository | Scope |
|---|---|
| [Amiga Bootcamp](https://github.com/alfishe/amiga-bootcamp) | Amiga hardware, AmigaOS, custom chips, software — the reference for Minimig core internals |
| [FPGA Knowledge Base](https://github.com/alfishe/fpga-bootcamp) | General FPGA: Cyclone V architecture, AXI buses, timing, HDL, vendor toolchains |

---

## Documentation Map

### Phase 1 — Foundation

#### 00 — Overview
| File | Topic |
|---|---|
| [README.md](00_overview/README.md) | Platform overview, evolution, architecture, success factors, challenges & growing pains |
| [getting_started.md](00_overview/getting_started.md) | Persona-based getting started — retro gamers, developers, students, builders, preservationists |

#### 01 — System Architecture
| File | Topic |
|---|---|
| [platform_architecture.md](01_system_architecture/platform_architecture.md) | Abstract SoC concepts, HPS↔FPGA pipeline, competitive analysis, data flow diagrams |

#### 02 — Hardware Platforms
| File | Topic |
|---|---|
| [de10_nano.md](02_hardware_platforms/de10_nano.md) | DE10-Nano board reference: Cyclone V 5CSEBA6, DDR3, ADC, HDMI TX |
| [addon_boards.md](02_hardware_platforms/addon_boards.md) | SDRAM board, IO board (VGA, serial), USB Hub, Direct Video, MT32-Pi |
| [memory_map.md](02_hardware_platforms/memory_map.md) | Cyclone V SoC physical address map as used by MiSTer |

---

### Phase 2 — Host Software (HPS Control Plane)

#### 03 — HPS Linux
| Folder / File | Topic |
|---|---|
| [kernel/](03_hps_linux/kernel/) | Linux kernel patches, Cyclone V SoC support, FPGA Manager driver |
| [uboot/](03_hps_linux/uboot/) | U-Boot patches, boot sequence, warm/cold reboot, OCRAM handoff |
| [buildroot/](03_hps_linux/buildroot/) | Buildroot-based image generation, package selection, custom overlays |
| [alternative_distros.md](03_hps_linux/alternative_distros.md) | Debian, Ubuntu, Arch Linux ARM on MiSTer; MiSTerArch and MOnSieurFPGA |
| [device_tree/](03_hps_linux/device_tree/) | Device tree for DE10-Nano, HPS↔FPGA bridge bindings |
| [filesystem/](03_hps_linux/filesystem/) | SD card layout (`/media/fat/`), partition scheme |
| [scripts/](03_hps_linux/scripts/) | System scripts: startup, networking, USB mount, core switching |
| [drivers/](03_hps_linux/drivers/) | Kernel driver model: FPGA Manager, USB gadget, I2C, GPIO |

#### 04 — HPS Binary (`Main_MiSTer`)
| File | Topic |
|---|---|
| [architecture.md](04_hps_binary/architecture.md) | Module map, main loop, scheduler, key data structures |
| [build/overview.md](04_hps_binary/build/overview.md) | Main_MiSTer binary architecture: modules, startup sequence, FPGA communication |
| [build/toolchain.md](04_hps_binary/build/toolchain.md) | ARM cross-compilation toolchain and Makefile |

#### 05 — Configuration
| File | Topic |
|---|---|
| [mister_ini_guide.md](05_configuration/mister_ini_guide.md) | User-facing MiSTer.ini guide: video modes, filters, per-core overrides |
| [osd.md](05_configuration/osd.md) | OSD rendering pipeline and menu navigation |
| [core_configuration_legacy.md](05_configuration/core_configuration_legacy.md) | Legacy configuration reference (to be split into conf_str + mister_ini) |

---

### Phase 3 — Hardware Emulation (FPGA Plane)

#### 06 — FPGA Subsystem
| File | Topic |
|---|---|
| [sys_top.md](06_fpga_subsystem/sys_top.md) | Top-level hardware abstraction: PLL tree, bridge wiring, GPO/GPI, HDMI/VGA muxing |
| [hps_io_module.md](06_fpga_subsystem/hps_io_module.md) | `hps_io.sv` deep dive: port map, parameters, opcode dispatch |
| [hps_bus.md](06_fpga_subsystem/hps_bus.md) | HPS_BUS[48:0] — the 49-bit parallel bus |
| [spi_protocol.md](06_fpga_subsystem/spi_protocol.md) | SPI-over-GPO: software-emulated SPI via FPGA Manager registers |
| [fpga_loading.md](06_fpga_subsystem/fpga_loading.md) | RBF bitstream loading: FPGA Manager state machine |
| [sdram_controller.md](06_fpga_subsystem/sdram_controller.md) | `sdram.sv`: bank interleaving, refresh, dual-port |
| [ddr3_architecture.md](06_fpga_subsystem/ddr3_architecture.md) | F2H AXI bridge, DDR3 port allocation, bandwidth budget |
| [gpo_gpi_registers.md](06_fpga_subsystem/gpo_gpi_registers.md) | GPO/GPI register interface |

#### 07 — FPGA Cores Architecture
| File | Topic |
|---|---|
| [template_walkthrough.md](07_fpga_cores_architecture/template_walkthrough.md) | Step-by-step guide to building a core from `Template_MiSTer` |
| [build/overview.md](07_fpga_cores_architecture/build/overview.md) | FPGA build pipeline: QPF → Synthesis → Fitter → Assembler → RBF |

#### 08 — FPGA Cores Catalog
| File | Topic |
|---|---|
| [minimig.md](08_fpga_cores_catalog/minimig.md) | Amiga OCS/ECS/AGA — → [Amiga Bootcamp](https://github.com/alfishe/amiga-bootcamp) |
| [ao486.md](08_fpga_cores_catalog/ao486.md) | 486SX soft CPU, IDE/CD-ROM DMA, VGA, Sound Blaster |
| [nes.md](08_fpga_cores_catalog/nes.md) | NES: mappers, PPU/APU, iNES/NES 2.0 |
| [snes.md](08_fpga_cores_catalog/snes.md) | SNES: GSU/SA-1/SDD-1/DSP coprocessors |
| [genesis.md](08_fpga_cores_catalog/genesis.md) | Genesis: 68000+Z80, VDP, YM2612, Mega CD, 32X |
| [psx.md](08_fpga_cores_catalog/psx.md) | PSX: R3000A, GTE, GPU, MDEC, CD-ROM, CHD |
| [n64.md](08_fpga_cores_catalog/n64.md) | N64: VR4300, RDP/RSP, 8MB RDRAM |
| [arcade_and_mra.md](08_fpga_cores_catalog/arcade_and_mra.md) | Arcade cores: MRA XML, ROM merging, DIP switches |

---

### Phase 4 — Peripherals & I/O

#### 09 — Video & Audio
| File | Topic |
|---|---|
| [hdmi_scaler.md](09_video_audio/hdmi_scaler.md) | `ascal.vhd`: polyphase scaler, DDR3 framebuffer |
| [audio_pipeline.md](09_video_audio/audio_pipeline.md) | Sigma-delta DAC, I2S, HDMI audio, filters |
| [analog_video.md](09_video_audio/analog_video.md) | IO board analog output, direct video, CRT |

#### 10 — Input Devices
| File | Topic |
|---|---|
| [keyboard.md](10_input_devices/keyboard.md) | PS/2 keyboard emulation via USB HID |
| [mouse.md](10_input_devices/mouse.md) | PS/2 mouse emulation |
| [joystick.md](10_input_devices/joystick.md) | Joystick/gamepad handling, analog sticks, autofire |
| [snac_llapi.md](10_input_devices/snac_llapi.md) | SNAC & LLAPI: direct controller wiring to FPGA fabric |

#### 11 — Storage
| File | Topic |
|---|---|
| [floppy_emulation.md](11_storage/floppy_emulation.md) | Floppy disk emulation: ADF/DSK images |
| [hdd_ide_emulation.md](11_storage/hdd_ide_emulation.md) | HDD/IDE emulation: VHD images, DMA path |
| [file_transfer.md](11_storage/file_transfer.md) | File I/O protocol: ROM loading, save file upload/download |

#### 12 — Networking
| File | Topic |
|---|---|
| [wifi_setup.md](12_networking/wifi_setup.md) | USB WiFi, wpa_supplicant, SSH, FTP, Samba |

---

### Phase 5 — Features, Ecosystem & Reference

#### 13 — Save States
| File | Topic |
|---|---|
| [save_state_architecture.md](13_save_states/save_state_architecture.md) | DDR3 snapshot/restore, SSPI protocol, rewind |

#### 14 — Extensions
| File | Topic |
|---|---|
| [retroachievements.md](14_extensions/retroachievements.md) | RetroAchievements: odelot fork, DDRAM bridge, rcheevos |
| [cheats.md](14_extensions/cheats.md) | Cheat engine, .cht format |
| [mra_format.md](14_extensions/mra_format.md) | MRA XML schema, ROM merging |

#### 15 — Ecosystem
| File | Topic |
|---|---|
| [update_scripts.md](15_ecosystem/update_scripts.md) | `update_all`, `downloader`, custom database URLs |

#### 16 — Advanced Topics
| File | Topic |
|---|---|
| [custom_linux.md](16_advanced_topics/custom_linux.md) | Building custom MiSTer Linux images from scratch |
| [mistex.md](16_advanced_topics/mistex.md) | MiSTeX: porting MiSTer framework to alternative FPGA boards via SBC+FPGA architecture |
| [analogue_comparison.md](16_advanced_topics/analogue_comparison.md) | Analogue Pocket openFPGA vs MiSTer comparison |

#### 17 — References
| File | Topic |
|---|---|
| [uio_command_reference.md](17_references/uio_command_reference.md) | Complete UIO/FIO opcode table with bit-level detail |
