[← Knowledge Base](../README.md)

# Core Catalog

Per-system technical documentation for MiSTer cores. Each article covers the FPGA implementation specifics — how the original hardware is mapped onto the Cyclone V, what compromises were made, and what differs from software emulation.

## Articles

| File | Topic | Quality |
|---|---|---|
| [minimig.md](minimig.md) | Amiga OCS/ECS/AGA — chipset mapping, hps_ext.v, SDRAM vs DDR3 → [Amiga Bootcamp](https://github.com/alfishe/amiga-bootcamp) | 📋 Target: Deep |
| [ao486.md](ao486.md) | 486SX soft CPU, IDE/CD-ROM DMA, VGA, Sound Blaster | 📋 Target: Deep |
| [nes.md](nes.md) | NES: mapper support, PPU/APU, iNES/NES 2.0 | 📋 Target: Adequate |
| [snes.md](snes.md) | SNES: GSU/SA-1/SDD-1/DSP coprocessors, 5A22/S-PPU/S-SMP | 📋 Target: Adequate |
| [genesis.md](genesis.md) | Genesis/Mega Drive: 68000+Z80, VDP, YM2612, Mega CD, 32X, SVP | 📋 Target: Adequate |
| [psx.md](psx.md) | PlayStation: R3000A, GTE, GPU, MDEC, CD-ROM, CHD | 📋 Target: Adequate |
| [n64.md](n64.md) | N64: VR4300, RDP/RSP, 8MB RDRAM, accuracy status | 📋 Target: Adequate |
| [arcade_and_mra.md](arcade_and_mra.md) | Arcade cores: MRA XML format, ROM merging, DIP switches, MAME compatibility | 📋 Target: Deep |

## Cross-References

- [Framework](../06_fpga_subsystem/README.md) — shared sys/ modules all cores use
- [Extensions — RetroAchievements](../14_extensions/retroachievements.md) — per-core RA implementation status
