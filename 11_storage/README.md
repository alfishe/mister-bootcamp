# 11 — Storage

[← Back to Index](../README.md)

Disk emulation and file transfer mechanisms between HPS Linux and FPGA cores.

---

## Articles

| File | Topic | Quality |
|---|---|---|
| [file_transfer.md](file_transfer.md) | ROM/file download stream via ioctl_* (FIO engine), upload for save states | Adequate |
| [floppy_emulation.md](floppy_emulation.md) | Floppy disk block request model, SD card sector I/O, Minimig ADF | Adequate |
| [hdd_ide_emulation.md](hdd_ide_emulation.md) | ATA/ATAPI state machine, IDE register protocol, CUE/TOC for CD images | Adequate |

---

## Related Sections

* [06 — FPGA Subsystem](../06_fpga_subsystem/README.md) — `hps_io.sv` File I/O engine
* [08 — Cores Catalog](../08_fpga_cores_catalog/README.md) — Core-specific storage usage (ao486, Minimig)
* [13 — Save States](../13_save_states/README.md) — Save state persistence via file I/O
* [17 — References](../17_references/README.md) — UIO opcode tables for SD/FIO commands
