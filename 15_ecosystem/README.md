# 15 — Ecosystem & Community Tools

[← Back to Index](../README.md)

MiSTer's open-source cores have spawned an entire ecosystem of derived projects — from alternative platform ports, to commercial FPGA console adaptations, to physical chip replacement boards that use MiSTer HDL as their foundation.

---

## Alternative Platforms & Ports

| Project | Architecture | Status | Description |
|---|---|---|---|
| **[MiSTeX](https://github.com/MiSTeX-devel)** | SBC + FPGA (QMTech Artix-7 + Raspberry Pi) | Active, WIP | Ports MiSTer `sys/` framework to alternative FPGA boards. SBC handles Linux/HPS role via SPI/GPIO, FPGA runs cores. |
| **[MiSTer Pi](https://retroremake.co/)** | DE10-Nano pin-compatible clone | Active | Raspberry Pi-sized form factor clone board. Full core compatibility. ~$80–100. |
| **QMTech Cyclone V** | DE10-Nano alternative | Active | Same Cyclone V FPGA, different GPIO layout. Requires QMTech-specific I/O boards and cases. ~$90–110. |
| **[MiST](https://github.com/mist-devel)** | Altera Cyclone III (EP3C25) | Legacy | MiSTer's direct predecessor. Smaller FPGA, no HPS. Still maintained but no new complex cores. |
| **[SiDi](https://github.com/ManuFerHi/SiDi-FPGA)** | MiST-compatible clone | Active | Spanish-made MiST clone with improved I/O. Runs MiST cores. |
| **[MARS FPGA](https://www.marsfpga.com/)** | Custom multi-FPGA board | Dead (2025) | Attempted "MiSTer killer" with larger FPGA. Stalled after development struggles and team departures. |

## Commercial FPGA Products

| Product | Manufacturer | Relationship to MiSTer | Description |
|---|---|---|---|
| **Analogue Pocket** | Analogue | Indirect — openFPGA uses community-ported cores | Handheld FPGA console. Some cores ported from MiSTer HDL. Closed hardware, proprietary openFPGA SDK. |
| **Analogue 3D** | Analogue | Indirect | N64-focused FPGA console (2025). 4K upscaling. Premium closed-hardware product. |
| **Analogue Duo** | Analogue | Indirect | PC Engine / TurboGrafx-16 FPGA console. |
| **Analogue Mega Sg** | Analogue | Indirect | Genesis/Mega Drive FPGA console with cartridge slot. |
| **Analogue Super Nt** | Analogue | Indirect | SNES FPGA console. Out of production. |
| **Analogue Nt Mini** | Analogue | Indirect | NES FPGA console. Discontinued. |
| **Polymega** | Playmaji | None — software emulation | Multi-system retro console. Not FPGA-based. Included for competitive context. |
| **RetroTINK** | Mike Chi | Complementary | Video scalers (4K, 5X) often paired with MiSTer for analog output → modern display chain. |

## MiSTer Hardware Ecosystem

| Product | Vendor | Description |
|---|---|---|
| **MiSTer Multisystem** | Ultimate MiSTer / Heber | All-in-one console-style motherboard. Eliminates board stacking. |
| **Multisystem 2** | Ultimate MiSTer / Heber | Second-generation all-in-one (2025). Cartridge slot expansion, improved analog. |
| **MiSTercade** | MiSTer Addons | JAMMA interface board for arcade cabinet integration. |
| **BlisSTer** | Ultimate MiSTer | Bluetooth controller adapter for MiSTer. |
| **SNAC adapters** | Various (dnotq, community) | Direct OEM controller wiring to FPGA fabric. NES, SNES, Genesis, Neo Geo, etc. |
| **I/O Board** | MiSTer-devel (open-source) | Analog video (VGA/component), audio, SNAC connector. |
| **Direct Video adapters** | Various | Active HDMI-to-VGA for cheap CRT output without I/O board. |
| **MT32-Pi** | Various | Roland MT-32 emulation via Raspberry Pi. Pairs with ao486 and other DOS/MIDI cores. |
| **Armor cases** | MiSTer Addons | Premium aluminum enclosures. |

## FPGA Chip Replacement Projects

| Project | Target Hardware | Description |
|---|---|---|
| **Amiga custom chip replacements** | Amiga 500/2000/4000 | FPGA-based drop-in replacements for failing Agnus/Denise/Paula chips. Uses HDL from Minimig/MiSTer lineage. |
| **[OpenAmiga500FastRamExpansion](https://github.com/SukkoPera/OpenAmiga500FastRamExpansion)** | Amiga 500 | Open-source trapdoor RAM expansion. Related ecosystem project. |

## Community Tools & Scripts

| Tool | Author | URL | Description |
|---|---|---|---|
| **update_all** | Theypsilon | [GitHub](https://github.com/theypsilon/Update_All_MiSTer) | All-in-one core/firmware/BIOS synchronization script |
| **Downloader** | MiSTer-devel | [GitHub](https://github.com/MiSTer-devel/Downloader_MiSTer) | Core download engine (used by update_all) |
| **mrext** | Wizzomafizzo | [GitHub](https://github.com/wizzomafizzo/mrext) | Extensions suite: NFC game launcher, remote control, overclocking, management tools |
| **MiSTer Remote** | Wizzomafizzo | (part of mrext) | Web-based remote control — access MiSTer from phone/browser |
| **MiSTerArch** | Wizzomafizzo | (part of mrext) | Alternative Arch Linux-based system image |
| **jtframe** | Jotego | [GitHub](https://github.com/jotego/jtframe) | Arcade core development framework — Yamaha FM synths, CPS engines, Z80 integration |
| **mr-fusion** | MiSTer-devel | [GitHub](https://github.com/MiSTer-devel/mr-fusion) | SD card image builder for initial setup |
| **Pocket openFPGA tools** | Community | Various | Tools for porting MiSTer cores to Analogue Pocket openFPGA |
| **MiSTer2HDMI** | Community | GitHub | Alternative HDMI output solutions |
| **Names.txt** | Community | Various repos | Custom display name files for OSD game listings |
| **OSD theme packs** | Community | Various | Custom menu themes and fonts |
| **Input presets** | Community | Various | Pre-configured controller mappings for popular gamepads |

## Content & Media

| Project | Description |
|---|---|
| **MiSTer Addons YouTube** | Hardware reviews, setup guides, core announcements |
| **RetroRGB** | Bob's coverage of MiSTer, analog video, and FPGA gaming |
| **My Life in Gaming** | High-production retro gaming content, MiSTer features |
| **Wobbling Pixels** | Technical deep-dives into MiSTer cores and setup |
| **MiSTer FPGA Bible** | Community-maintained comprehensive guide collection |
| **pram0d's blog** | FPGA core development tutorials and walkthroughs |

## Patreon / Funding

| Developer | Platform | Focus |
|---|---|---|
| **Jotego (Jose Tejada)** | Patreon | Arcade core development (CPS1/2, SNK, Sega System 16, etc.) |
| **Robert Peip (FPGAzumSpass)** | Patreon | PSX, N64, GBA, Saturn cores |
| **Sorgelig** | — | Does not accept donations. Framework maintenance is volunteer. |

---

## Articles

| File | Topic |
|---|---|
| [update_scripts.md](update_scripts.md) | `update_all`, `downloader`, custom database URLs |
