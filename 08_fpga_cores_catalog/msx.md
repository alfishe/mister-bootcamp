[← Core Catalog](README.md) · [↑ Knowledge Base](../README.md)

# MSX / MSX2 / MSX2+ / MSX Turbo R

> Microsoft's standardized home computer architecture — hugely popular in Japan, Europe, and South America. The MiSTer core covers the full MSX generation span from MSX1 through Turbo R, with PSG, SCC, OPLL audio, and VDP evolution from TMS9918 to V9958.

Sources: [`MSX_MiSTer`](https://github.com/MiSTer-devel/MSX_MiSTer) · Original KdL MSX OCM core

---

## Architecture Overview

```mermaid
flowchart TD
    CPU["Z80 CPU<br/>3.58 MHz (MSX1/2)<br/>5.37 MHz (MSX2+)<br/>R800 + Z80 (TurboR)"]
    SLOT["Slot Manager<br/>Primary/Secondary slot expand"]
    VDP["VDP<br/>TMS9918 → V9938 → V9958"]
    VRAM["VRAM<br/>16 KB → 128 KB → 192 KB"]
    PSG["YM2149 PSG<br/>3 square + noise + I/O"]
    SCC["SCC/SCC+<br/>5 channels wave"]
    OPLL["YM2413 OPLL<br/>9 FM + 5 rhythm"]
    RAM["Main RAM<br/>64 KB – 4 MB (mapper)"]
    FMPAC["FM-PAC<br/>YM2413 + SRAM"]
    SUB["Sub ROM<br/>MSX2+ / TurboR BIOS"]
    DISK["Disk Controller<br/>720 KB 3.5\" FDD"]
    MIDI["MIDI<br/>Optional interface"]
    RTC["RTC<br/>RP5C01 calendar + RAM"]
    SD["SD Card<br/>FAT16, SDSC/SDHC"]

    CPU --- SLOT
    SLOT --- VDP
    SLOT --- RAM
    SLOT --- PSG
    SLOT --- SCC
    SLOT --- OPLL
    SLOT --- FMPAC
    SLOT --- SUB
    SLOT --- DISK
    SLOT --- MIDI
    SLOT --- RTC
    SLOT --- SD
    VDP --- VRAM
```

---

## Hardware Specifications by Generation

| Component | MSX1 (1983) | MSX2 (1985) | MSX2+ (1988) | Turbo R (1990) |
|---|---|---|---|---|
| **CPU** | Z80A @ 3.58 MHz | Same | Z80 @ 5.37 MHz | R800 (14.32 MHz) + Z80 |
| **VDP** | TMS9918A | V9938 | V9958 | V9958 |
| **VRAM** | 16 KB | 128 KB | 128 KB | 128 KB |
| **Text** | 40×24, 32×24 | 80×24 | 80×24 | 80×24 |
| **Max colors** | 16 | 512 (256 on screen) | 19,268 (YJK) | Same |
| **Sprites** | 32, 4 per line | 32, 8 per line | Same | Same |
| **Sound** | YM2149 PSG | PSG + optional OPLL | PSG + OPLL + SCC | Same |
| **RAM** | 8–64 KB | 64–512 KB | 64 KB–4 MB | 256 KB–4 MB |
| **Disk** | Optional | 3.5" FDD | Same | Same + IDE |

---

## Audio Subsystem

The MSX platform has one of the richest audio ecosystems of any 8/16-bit computer:

| Sound Chip | Type | Channels | Notes |
|---|---|---|---|
| **YM2149 PSG** | Square/noise | 3 + noise | Standard across all MSX |
| **SCC / SCC+** | Waveform | 5 | Konami cartridge — programmable 32-byte waveforms |
| **YM2413 OPLL** | FM synthesis | 9 + 5 rhythm | MSX-MUSIC / FM-PAC |
| **MSX-AUDIO** | FM synthesis | 9 + 5 | Yamaha Y8950 — rare, separate expansion |
| **VDP noise** | Click | 1 | TMS9918 click output (MSX1) |

---

## Slot Architecture

MSX uses a slot-based memory mapping system instead of fixed addresses:

- **4 primary slots** (0–3), each up to 64 KB
- **Secondary slots** — each primary can be expanded to 8 sub-slots
- **Memory mapper** — bank-switched RAM in 16 KB pages (up to 4 MB)
- Cartridges map into slot space — BIOS auto-detects and initializes

---

## MiSTer Core Features

Source: [`MSX_MiSTer` README](https://github.com/MiSTer-devel/MSX_MiSTer)

| Feature | Detail |
|---|---|
| **MSX generations** | MSX2 / MSX2+ / MSX3 (Turbo R) |
| **Sound** | YM2149 PSG + YM2413 OPLL + SCC/SCC+ |
| **Virtual cartridges** | Various sound and memory expansions |
| **Turbo mode** | CPU speed boost, F11 to change speed |
| **Mouse** | Supported |
| **Joystick** | Supported |
| **RTC** | Real-time clock |
| **VHD images** | Hard disk emulation |
| **SD card** | SDSC (< 4 GB) and SDHC/SDXC (> 4 GB) support |
| **File manager** | `mm` utility for loading ROM/DSK files |
| **Warm/cold reset** | Short press = warm, 2+ seconds = cold |

### SD Card Setup

Use the `sdcreate` utility (requires admin rights) to format and create a bootable MSX SD card. `OCM-BIOS.DAT` must be the first file copied after format. Only FAT16 is supported. Partitions < 4 GB regardless of card size.

---

## Cross-References

| Topic | Article |
|---|---|
| Z80 CPU family | [C64](c64.md) (C128 Z80 mode), [MSX](msx.md) |
| VDP evolution | TMS9918 lineage → V9938 → V9958 |
| Floppy disk emulation | [Floppy Emulation](../11_storage/floppy_emulation.md) |
| HDD/IDE emulation | [HDD/IDE](../11_storage/hdd_ide_emulation.md) |
| Audio pipeline | [Audio Pipeline](../09_video_audio/audio_pipeline_deep_dive.md) |
| SNAC joystick wiring | [SNAC & LLAPI](../10_input_devices/snac_llapi.md) |
