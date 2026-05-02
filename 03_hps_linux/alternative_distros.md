[← HPS Linux Index](README.md) · [↑ MiSTer Knowledge Base](../../README.md)

# Alternative Linux Distributions on MiSTer

MiSTer's stock `linux.img` is a Buildroot-generated initramfs: minimal, fast-booting, and immutable. This article covers how to replace it with a full Linux distribution — Debian, Ubuntu, or Arch Linux ARM — while preserving MiSTer's kernel, U-Boot, device tree, and FPGA bridge functionality. Two proven strategies are documented: an in-place loopback replacement (minimal invasiveness) and a separate ext4 root partition (full distribution experience, as implemented by MiSTerArch).

> [!NOTE]
> This article assumes you understand the MiSTer boot chain (SPL → U-Boot → kernel → rootfs) and the SD card partition layout. See [U-Boot](../uboot/) and [Image Build](../image_build/) for background.

---

## 1. What Is Actually MiSTer-Specific

The boot chain on the DE10-Nano (Cyclone V SoC) is:

```mermaid
flowchart LR
    ROM["Boot ROM"] --> SPL["SPL<br/>(A2 partition)"]
    SPL --> UB["U-Boot"]
    UB --> K["zImage + DTB"]
    K --> R["rootfs<br/>(Buildroot / Debian / Arch)"]
```

MiSTer-specific logic lives in **three places only**. Everything else is a standard ARM Linux rootfs:

| Component | What it does | Can it be replaced? |
|---|---|---|
| **U-Boot** (`MiSTer-devel/u-boot-socfpga`) | Bootloader with MiSTer-specific reboot-to-RBF logic — when you select a core in the menu, U-Boot loads the new bitstream and optionally an alternate kernel/rootfs | **No** — keep the MiSTer U-Boot |
| **Kernel** (`MiSTer-devel/Linux-Kernel_MiSTer`) | Patched framebuffer driver with integrated scaler, custom HDMI/I2S/SPDIF audio, FPGA bridge drivers, IO board drivers, USB hub drivers | **No** — keep the MiSTer kernel |
| **Device Tree** (`socfpga_cyclone5_de10_nano.dtb`) | Describes SoC, HPS-FPGA bridges, peripherals | **No** — keep the MiSTer DTB |
| **Rootfs** (`linux.img`) | BusyBox, init scripts, `Main_MiSTer` binary, update scripts | **Yes** — this is what you replace |

> **Key insight:** The kernel and U-Boot expose a standard Linux ABI to userspace. Any `armhf` rootfs with the correct kernel modules can boot, provided it mounts `/media/fat/` and can launch the `Main_MiSTer` binary.

---

## 2. Strategy A — In-Place `linux.img` Replacement

The least invasive approach: replace the Buildroot loopback image with a Debian (or Arch) loopback image, leaving partitions and U-Boot untouched.

### 2.1 Build the Replacement Image

On an x86_64 Linux host:

```bash
# Create a 4 GB loopback image
dd if=/dev/zero of=linux.img bs=1M count=4096
mkfs.ext4 linux.img

# Mount it
mkdir -p /mnt/mister-root
mount -o loop linux.img /mnt/mister-root

# Bootstrap Debian armhf
qemu-debootstrap --arch=armhf bookworm /mnt/mister-root \
    http://deb.debian.org/debian/
```

### 2.2 Install MiSTer Kernel Modules

The new rootfs needs the kernel modules from the MiSTer kernel build:

```bash
# Copy modules for the running kernel version
KVER=$(make -C /path/to/mister-kernel kernelrelease 2>/dev/null)
cp -a /path/to/mister-kernel/lib/modules/${KVER} \
    /mnt/mister-root/lib/modules/
```

### 2.3 Install MiSTer Userspace

```bash
# Copy the Main_MiSTer binary and support files
cp /path/to/MiSTer /mnt/mister-root/usr/local/bin/
cp -r /path/to/mister-scripts /mnt/mister-root/media/fat/

# Ensure /media/fat/ exists and will be mounted
mkdir -p /mnt/mister-root/media/fat
```

### 2.4 Configure Init

For Debian with `systemd`:

```bash
# Create a systemd service that starts MiSTer on boot
cat > /mnt/mister-root/etc/systemd/system/mister.service << 'EOF'
[Unit]
Description=MiSTer FPGA Main Binary
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/MiSTer
Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF

# Enable it
arch-chroot /mnt/mister-root systemctl enable mister.service
```

For `sysvinit`, add to `/etc/inittab`:

```
::respawn:/usr/local/bin/MiSTer
```

### 2.5 Install and Boot

```bash
umount /mnt/mister-root

# Copy the new linux.img to the FAT partition on the SD card
# (replace the existing linux.img)
```

> [!WARNING]
> The exFAT driver must be available (built-in or as a module) or `/media/fat/` will not mount. MiSTer's kernel 5.x exports `exfat` as a module — verify it is present in `lib/modules/${KVER}/fs/exfat/`.

---

## 3. Strategy B — Separate ext4 Root Partition

The MiSTerArch approach: repartition the SD card to add a dedicated ext4 root partition, keeping the exFAT `MiSTer_Data` partition intact.

### 3.1 Partition Layout

| Partition | Type | Size | Contents |
|---|---|---|---|
| `mmcblk0p1` | FAT32 | ~500 MB | Boot files: `zImage`, `socfpga_cyclone5_de10_nano.dtb`, `u-boot.img` |
| `mmcblk0p2` | `0xA2` (raw) | ~3 MB | SPL + U-Boot |
| `mmcblk0p3` | ext4 | Remainder | Full Arch/Debian rootfs |
| `mmcblk0p4` | exFAT | Optional | `MiSTer_Data` (ROMs, cores, configs) |

> [!CAUTION]
> The `0xA2` partition type and offsets are fixed by the Cyclone V BootROM. Do not resize or move this partition.

### 3.2 Boot Script

U-Boot needs a `boot.scr` that points the kernel to the new root partition:

```bash
# boot.cmd
setenv mmcroot /dev/mmcblk0p3
setenv mmcargs setenv bootargs console=ttyS0,115200 root=${mmcroot} rw rootwait
run mmcargs
load mmc 0:1 ${loadaddr} zImage
load mmc 0:1 ${fdtaddr} socfpga_cyclone5_de10_nano.dtb
bootz ${loadaddr} - ${fdtaddr}
```

Compile it with `mkimage`:

```bash
mkimage -A arm -O linux -T script -C none \
    -n "MiSTer boot" -d boot.cmd boot.scr
```

Source: [zangman/de10-nano](https://github.com/zangman/de10-nano)

### 3.3 MiSTerArch Reference Implementation

[MiSTerArch](https://github.com/MiSTerArch/PKGBUILDs) is the canonical working example of Strategy B. It provides:

- Pre-built SD card images with Arch Linux ARM
- `PKGBUILD`s for MiSTer-specific packages (`uboot-mister`, `linux-mister`, `mister-bin`, `mister-menu`)
- `pacman` repository at `http://misterarch.hypertriangle.com/repo`
- Systemd integration for launching the MiSTer binary

Installation flow:

```bash
# Initialize pacman keys
pacman-key --init && pacman-key --populate archlinuxarm

# Add MiSTerArch repository
cat >> /etc/pacman.conf << 'EOF'
[misterarch]
SigLevel = Optional TrustedOnly
Server = http://misterarch.hypertriangle.com/repo
EOF

# Install MiSTer packages
pacman -Syu uboot-mister linux-mister mister-bin mister-menu
systemctl enable MiSTer
```

Source: [MiSTerArch/PKGBUILDs/INSTALL.md](https://github.com/MiSTerArch/PKGBUILDs/blob/main/INSTALL.md)

### 3.4 MOnSieurFPGA

[MOnSieurFPGA](https://github.com/noemiabril/MOnSieurFPGA-SD_Image_Builds) provides pre-built SD images combining MiSTerFPGA userspace binaries with Arch Linux ARM. Key characteristics:

- `pacman`-based package management
- Pre-installed: `rsync`, `git`, `go-ipfs`, `networkmanager`, `bluez`, `imlib2`, `freetype2`
- The `MiSTer` binary lives at `/usr/bin/MiSTer`
- `menu.rbf` located in `/boot/`
- Fast USB polling (1000 Hz) enabled by default in `/boot/uboot.txt`

Source: [noemiabril/MOnSieurFPGA-SD_Image_Builds](https://github.com/noemiabril/MOnSieurFPGA-SD_Image_Builds)

---

## 4. Critical Gotchas

### 4.1 Kernel Modules Lock

If you use the MiSTer kernel binary, **do not install distribution kernel packages**. Their modules are built against a different kernel configuration and will not load.

```bash
# Debian — hold kernel packages
apt-mark hold linux-image-* linux-headers-*

# Arch — ignore kernel packages in pacman
echo 'IgnorePkg = linux linux-headers' >> /etc/pacman.conf
```

> [!WARNING]
> If you accidentally install a distribution kernel, its modules will populate `/lib/modules/` but the running MiSTer kernel will fail to load them. Symptoms: missing `/dev/` nodes, non-working WiFi, HDMI audio failure.

### 4.2 MiSTer Updater Compatibility

The official `update.sh` and `downloader` expect to overwrite `linux.img`, the kernel, and the bootloader. In a custom distro setup:

- Set `update_linux = false` in `downloader.ini` (new updater)
- For the legacy `update.sh`, either modify it or run updates manually for cores only
- MiSTerArch handles this by keeping kernel/U-Boot under pacman's control and blocking `linux.img` updates

### 4.3 `/media/fat/` Is Hardcoded

The `Main_MiSTer` binary and many scripts expect `/media/fat/` to be the exFAT data partition. In Strategy B:

```bash
# /etc/fstab entry for the exFAT partition
/dev/mmcblk0p4  /media/fat  exfat  defaults,noatime  0  0
```

Systemd with `udev` usually creates the mount automatically, but an explicit fstab entry ensures it is available before `mister.service` starts.

### 4.4 FPGA Bridges and `/dev/mem`

MiSTer userspace communicates with the FPGA via memory-mapped registers exposed by the kernel. The device nodes (`/dev/MiSTer_*`) are created by `udev` rules. On systemd-based distros, verify:

```bash
ls -la /dev/MiSTer*
# Expected: /dev/MiSTer, /dev/MiSTer_fb, etc.
```

If missing, copy the `udev` rules from the stock Buildroot rootfs or from the MiSTerArch PKGBUILDs.

### 4.5 exFAT in Kernel

The MiSTer kernel must have exFAT support. Modern MiSTer kernels (5.x+) include `exfat.ko` as a module. Ensure it is loaded at boot:

```bash
# Check
lsmod | grep exfat

# If missing, load manually
modprobe exfat
```

### 4.6 Boot Time

Full distributions boot slower than Buildroot. Community-reported times:

| Distro | Boot to MiSTer menu | Notes |
|---|---|---|
| **Stock Buildroot** | ~3–5 seconds | Minimal initramfs |
| **MiSTerArch** | ~10–15 seconds | systemd, services, journal |
| **Debian with systemd** | ~15–25 seconds | Full service startup |

Source: MiSTer FPGA Forum — [MiSTerArch thread](https://misterfpga.org/viewtopic.php?t=4264)

---

## 5. Comparison: Buildroot vs Full Distro

| Aspect | Buildroot (Stock) | Debian/Arch (Custom) |
|---|---|---|
| **Boot time** | ~3–5 seconds | ~10–25 seconds |
| **Package manager** | None (static binaries + rebuild) | `apt` / `pacman` |
| **Rootfs mutability** | Read-only initramfs (ephemeral) | Persistent ext4 |
| **Development tools** | Cross-compile only | Native GCC, Python, git, etc. |
| **Systemd/services** | BusyBox init | Full systemd |
| **Journal/logging** | `dmesg` only | `journalctl`, persistent logs |
| **Kernel updates** | Rebuild `linux.img` | Hold kernel, manual rebuild |
| **Updater compatibility** | Full | Partial (disable `update_linux`) |
| **Best for** | Gaming, stability, fast boot | Development, experimentation, package flexibility |

---

## 6. Platform Context

| Project | Distro | Strategy | Package Manager | Status |
|---|---|---|---|---|
| **Stock MiSTer** | Buildroot | Initramfs | None | Official, maintained |
| **MiSTerArch** | Arch Linux ARM | Separate ext4 root | `pacman` | Community, active |
| **MOnSieurFPGA** | Arch Linux ARM | Separate ext4 root | `pacman` | Community, active |
| **zangman/de10-nano** | Arch Linux ARM | Separate ext4 root | `pacman` | Educational guide |
| **Custom Debian** | Debian armhf | In-place or separate ext4 | `apt` | DIY |

---

## 7. Cross-References

- [Buildroot Overview](buildroot/buildroot_overview.md) — Stock rootfs architecture
- [Package Management](buildroot/package_management.md) — Entware, chroot, and cross-compilation alternatives
- [HPS Binary — ARM Cross-Compilation]( ../../04_hps_binary/build/toolchain.md) — Building `Main_MiSTer` and utilities
- [HPS Binary — Main_MiSTer Overview]( ../../04_hps_binary/build/overview.md) — Main_MiSTer binary architecture and capabilities
- [Image Build](image_build/) — SD card partition layout and `linux.img` creation
- [U-Boot](uboot/) — Bootloader patches and boot sequence
- [Kernel](kernel/) — MiSTer kernel patches and build

---

## 8. References

| Source | Path / URL |
|---|---|
| MiSTerArch PKGBUILDs | [`MiSTerArch/PKGBUILDs`](https://github.com/MiSTerArch/PKGBUILDs) |
| MiSTerArch Install Guide | [`PKGBUILDs/INSTALL.md`](https://github.com/MiSTerArch/PKGBUILDs/blob/main/INSTALL.md) |
| MOnSieurFPGA SD Images | [`noemiabril/MOnSieurFPGA-SD_Image_Builds`](https://github.com/noemiabril/MOnSieurFPGA-SD_Image_Builds) |
| zangman DE10-Nano Guide | [`zangman/de10-nano`](https://github.com/zangman/de10-nano) — Archlinux ARM RootFS walkthrough |
| MiSTerArch Forum Thread | [misterfpga.org/viewtopic.php?t=4264](https://misterfpga.org/viewtopic.php?t=4264) |
| MiSTer Linux Kernel | [`MiSTer-devel/Linux-Kernel_MiSTer`](https://github.com/MiSTer-devel/Linux-Kernel_MiSTer) |
| MiSTer U-Boot | [`MiSTer-devel/u-boot_MiSTer`](https://github.com/MiSTer-devel/u-boot_MiSTer) |
| Debian debootstrap | [wiki.debian.org/Debootstrap](https://wiki.debian.org/Debootstrap) |
| Arch Linux ARM | [archlinuxarm.org](https://archlinuxarm.org/) |
