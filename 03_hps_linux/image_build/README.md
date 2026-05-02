[← Linux System](../README.md) · [↑ Knowledge Base](../../README.md)

# Image Build

This section covers the assembly of the bootable MiSTer SD card image — from compiled artifacts (Buildroot rootfs, kernel zImage, U-Boot bootloader) through partition layout, file bundling, and first-boot verification. The actual compilation of those artifacts is covered in their respective subdirectories ([buildroot/](../buildroot/), [kernel/](../kernel/), [uboot/](../uboot/)).

## Articles

| File | Topic |
|---|---|
| [overview.md](overview.md) | End-to-end pipeline overview: what artifacts feed into the image and how they relate |
| [image_generation.md](image_generation.md) | SD card partition layout (MBR, 0xA2, FAT32), manual sfdisk/dd assembly, genimage Docker pipeline, boot script compilation, first-boot lifecycle |
