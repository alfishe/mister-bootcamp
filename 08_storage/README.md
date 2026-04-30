[← Knowledge Base](../README.md)

# Storage

Disk and file I/O emulation — how MiSTer presents SD card images as floppy drives, hard disks, and CD-ROMs to FPGA cores, plus the ROM/file loading protocol.

## Articles

| File | Topic | Quality |
|---|---|---|
| [floppy_emulation.md](floppy_emulation.md) | Floppy disk emulation: ADF/DSK images, sector I/O via UIO protocol | 📋 Target: Deep |
| [hdd_ide_emulation.md](hdd_ide_emulation.md) | HDD/IDE emulation: VHD images, DMA path, hps_ext extensions | 📋 Target: Deep |
| [file_transfer.md](file_transfer.md) | File I/O protocol: ROM loading (FIO_FILE_TX), save file upload/download | 📋 Target: Adequate |

## Cross-References

- [Framework — hps_io.sv](../02_framework/hps_io_module.md) — SD card and file I/O opcodes
- [Configuration — CONF_STR](../09_configuration/conf_str.md) — `S`, `F`, `FC`, `FS` slot declarations
