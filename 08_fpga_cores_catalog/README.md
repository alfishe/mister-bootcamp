# 08 — FPGA Cores Catalog

[← Back to Index](../README.md)

Detailed articles on MiSTer's major console, computer, and arcade cores.

---

## Articles

| File | Topic | Quality |
|---|---|---|
| [arcade_and_mra.md](arcade_and_mra.md) | Arcade cores, MRA XML schema, ROM assembly, DIP switches, Joteco framework | Deep |
| [minimig.md](minimig.md) | Amiga OCS/ECS/AGA: dual CPU, Agnus DMA, Denise, Paula, Gary, RTG, CD32 | Deep |
| [ao486.md](ao486.md) | 486SX soft CPU, Sound Blaster 16, SVGA, IDE/CD-ROM | Deep |
| [ao486_networking.md](ao486_networking.md) | UART SLIP networking, MiSTerFS shared-memory file system | Deep |
| [nes.md](nes.md) | NES/Famicom: 2A03 CPU+APU, 2C02 PPU, 200+ mappers, FDS, expansion audio | Adequate |
| [snes.md](snes.md) | SNES: 5A22 CPU, dual S-PPU, S-SMP audio, DSP/Super FX/SA-1 coprocessors | Adequate |
| [genesis.md](genesis.md) | Mega Drive/Genesis: 68000+Z80, YM7101 VDP, YM2612 FM, SVP, Nuked MD | Adequate |
| [psx.md](psx.md) | PlayStation: R3000A, GTE, GPU, SPU, MDEC, CD-ROM | Adequate |
| [n64.md](n64.md) | Nintendo 64: VR4300i, RSP+RDP, RDRAM, 4–8 MB | Adequate |
| [gba.md](gba.md) | Game Boy Advance: ARM7TDMI, PPU 6 modes, dual SID, 2× resolution, rewind | Adequate |
| [gb_gbc.md](gb_gbc.md) | Game Boy / Color: Sharp LR35902, 4-shade LCD, MBC1–7, SGB borders, link cable | Adequate |
| [c64.md](c64.md) | Commodore 64: 6510, VIC-II, SID 6581/8580, 1541 GCR, REU, GeoRAM, dual SID | Adequate |
| [atari_st.md](atari_st.md) | Atari ST/STe: 68000, FX68K, Shifter, Blitter, YM2149, MIDI, ACSI | Adequate |
| [saturn.md](saturn.md) | Sega Saturn: dual SH-2, VDP1/VDP2, SCU DSP, SCSP audio (WIP/beta) | Adequate |
| [pc_engine.md](pc_engine.md) | PC Engine / TG-16 / SuperGrafx: HuC6280, VDC, CD-ROM², Arcade Card | Adequate |
| [msx.md](msx.md) | MSX/MSX2+/TurboR: Z80/R800, V9938/V9958, PSG+SCC+OPLL, slot architecture | Adequate |
| [neogeo.md](neogeo.md) | Neo Geo MVS/AES/CD: 68000, LSPC, YM2610, 330 Mbit ROM, Darksoft/MAME | Adequate |

---

## Related Sections

* [07 — Cores Architecture](../07_fpga_cores_architecture/README.md) — Core template, build system
* [06 — FPGA Subsystem](../06_fpga_subsystem/README.md) — `hps_io.sv`, SDRAM/DDR3, memory controllers
* [09 — Video / Audio](../09_video_audio/README.md) — HDMI scaler, analog video, audio pipeline

