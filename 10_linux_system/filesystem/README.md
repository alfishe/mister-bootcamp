[← Linux System](../README.md) · [↑ Knowledge Base](../../README.md)

# Filesystem

MiSTer's SD card layout and directory conventions. The system uses a FAT32 partition at `/media/fat/` as the primary user-facing storage, with a separate ext4 Linux root.

## Planned Articles

| File | Topic |
|---|---|
| `partition_layout.md` | SD card partitions: exFAT/FAT32 data, ext4 root, raw U-Boot area |
| `directory_conventions.md` | `/media/fat/` layout: `_Cores/`, `games/`, `config/`, `linux/`, `Scripts/` |
| `config_files.md` | Per-core config persistence: `.CFG`, `.f0`, `.s0` files |
