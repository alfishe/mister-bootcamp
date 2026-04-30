[← Knowledge Base](../README.md)

# Linux System

The embedded Linux stack running on the Cyclone V HPS ARM core. MiSTer uses a heavily customised Buildroot-based Linux with a patched kernel and U-Boot — this section covers every layer from bootloader through kernel to userland.

## Subdirectories

| Folder | Scope |
|---|---|
| [kernel/](kernel/README.md) | Linux kernel: MiSTer-specific patches, Kconfig, Cyclone V SoC support, FPGA Manager driver |
| [uboot/](uboot/README.md) | U-Boot: boot sequence, warm/cold reboot, OCRAM handoff, Intel SoC patches |
| [buildroot/](buildroot/README.md) | Buildroot image generation: package selection, custom overlays, SD card image creation |
| [device_tree/](device_tree/README.md) | Device tree: DE10-Nano DTS, HPS↔FPGA bridge bindings, overlay compilation |
| [filesystem/](filesystem/README.md) | Filesystem layout: SD card partitions, `/media/fat/` conventions, directory structure |
| [scripts/](scripts/README.md) | System scripts: startup sequence, networking, USB mount, core switching logic |
| [drivers/](drivers/README.md) | Kernel drivers: FPGA Manager, USB gadget, I2C (HDMI), GPIO, custom MiSTer drivers |

## Cross-References

- [HPS Binary](../04_hps_binary/README.md) — the userland binary that runs on top of this Linux stack
- [Advanced Topics — Custom Linux](../16_advanced_topics/custom_linux.md) — building custom images from scratch
- [FPGA KB — Embedded Linux](https://github.com/alfishe/fpga-bootcamp/blob/main/10_embedded_linux/)
