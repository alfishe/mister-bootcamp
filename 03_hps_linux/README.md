# 03 — HPS Linux

[← Back to Index](../README.md)

Linux running on the ARM Cortex-A9 HPS — Buildroot image generation, kernel patches, U-Boot, device tree, and filesystem layout.

---

## Articles

| File | Topic | Quality |
|---|---|---|
| [alternative_distros.md](alternative_distros.md) | Debian, Ubuntu, Arch Linux ARM; MiSTerArch and MOnSieurFPGA | Deep |
| [Buildroot Linux for DE10-Nano - Comprehensive.md](Buildroot%20Linux%20for%20DE10-Nano%20-%20Comprehensive.md) | Comprehensive Buildroot reference (migrated) | Adequate |
| [Buildroot Linux for DE10-Nano.md](Buildroot%20Linux%20for%20DE10-Nano.md) | Buildroot overview (migrated) | Adequate |

---

## Subdirectories

| Directory | Content |
|---|---|
| [buildroot/](buildroot/) | Buildroot config, defconfig, packages, overview — **4 Deep articles** |
| [device_tree/](device_tree/) | DE10-Nano device tree bindings |
| [drivers/](drivers/) | Kernel driver model: FPGA Manager, USB gadget, I2C, GPIO |
| [filesystem/](filesystem/) | SD card layout (`/media/fat/`), partition scheme |
| [image_build/](image_build/) | SD image generation with mr-fusion |
| [kernel/](kernel/) | Linux kernel patches for Cyclone V SoC |
| [scripts/](scripts/) | System scripts: startup, networking, USB mount |
| [uboot/](uboot/) | U-Boot patches, boot sequence, warm/cold reboot |
| [uboot_patches/](uboot_patches/) | Extracted U-Boot patches and Intel BSP patches |
