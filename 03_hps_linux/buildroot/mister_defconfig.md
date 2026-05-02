[← Buildroot Index](README.md) · [↑ Linux System](../README.md) · [↑ Knowledge Base](../../README.md)

# MiSTer Buildroot Defconfig Walkthrough

A line-by-line walkthrough of the mr-fusion Buildroot defconfig — what every option does, why it's set that way for MiSTer, and what breaks if you change it.

> [!NOTE]
> This article assumes familiarity with [Buildroot Overview](buildroot_overview.md). Read that first if you're new to Buildroot's defconfig model, package system, or output structure.

---

## 1. Defconfig Source

The canonical mr-fusion Buildroot defconfig lives at:

```
mr-fusion/builder/config/buildroot-defconfig
```

It is a **minimal defconfig** — only non-default options are listed. Running `make mr-fusion_defconfig` expands it into a full `.config` with all Kconfig defaults resolved. The target Buildroot version is **2024.08 or newer** (tested through 2026.02).

> [!IMPORTANT]
> Keep in mind that `mr-fusion` is **only the installer environment**. It is not the runtime MiSTer OS. Therefore, this defconfig only defines the packages needed to bootstrap the SD card, not the packages you would use while playing games.

Source: [`mr-fusion`](https://github.com/MiSTer-devel/mr-fusion/blob/master/builder/config/buildroot-defconfig)

---

## 2. Target Architecture

```ini
BR2_arm=y
BR2_cortex_a9=y
BR2_ARM_ENABLE_NEON=y
BR2_ARM_FPU_NEON=y
```

### 2.1 `BR2_arm=y`

Selects the 32-bit ARM architecture. The Cyclone V SoC's HPS contains dual ARM Cortex-A9 cores — these are ARMv7-A, not ARMv8 (AArch64). There is no 64-bit option on this silicon.

> [!WARNING]
> Do not change to `BR2_aarch64=y`. The DE10-Nano's Cortex-A9 is strictly 32-bit. A 64-bit kernel and rootfs will not boot.

### 2.2 `BR2_cortex_a9=y`

Selects the specific ARM core variant. This controls:
- **Instruction scheduling** in GCC: the Cortex-A9 has a dual-issue, out-of-order pipeline with specific latencies.
- **`-mcpu=cortex-a9`** flag passed to the compiler.

Related: The Cyclone V's Cortex-A9 runs at 800 MHz on MiSTer (configurable via the SPL/PLL initialization in U-Boot).

### 2.3 `BR2_ARM_ENABLE_NEON=y` and `BR2_ARM_FPU_NEON=y`

NEON is the ARM Advanced SIMD extension — 128-bit wide vector operations. For MiSTer:
- **NEON is used extensively by `Main_MiSTer`** for video scaling, color space conversion, and audio filtering in the HPS binary.
- Hard-float (`hf`) ABI passes floating-point arguments in VFP registers rather than the stack — measurably faster for the scaler pipeline.

These two options also determine the toolchain triplet:
- `BR2_ARM_FPU_NEON=y` → `arm-buildroot-linux-gnueabihf-` (hard-float)
- `BR2_ARM_FPU_VFPV2=y` (legacy) → `arm-buildroot-linux-gnueabi-` (soft-float)

> [!NOTE]
> The MiSTer kernel defconfig also enables `CONFIG_VFP=y` and `CONFIG_NEON=y` at the kernel level. These must match the Buildroot toolchain configuration, or user-space binaries will crash with `SIGILL` (illegal instruction) on NEON-using code paths.

Source: `mr-fusion/builder/config/kernel-defconfig`

---

## 3. Toolchain Features

```ini
BR2_CCACHE=y
BR2_TOOLCHAIN_BUILDROOT_WCHAR=y
BR2_TOOLCHAIN_BUILDROOT_CXX=y
```

### 3.1 `BR2_CCACHE=y`

Enables `ccache` — a compiler cache that speeds up repeated builds by caching object files. Essential for CI/CD (mr-fusion's Docker pipeline) where the Buildroot build runs frequently. Without ccache, a full `make clean; make` rebuild compiles everything from scratch (20–45 minutes on modern hardware).

### 3.2 `BR2_TOOLCHAIN_BUILDROOT_WCHAR=y`

Enables wide-character support (`wchar_t`) in uClibc-ng. Required by:
- **util-linux** (`mount`, `fdisk`, `blkid` — used by S99install-MiSTer.sh)
- Any package that uses locale-aware string handling

Without this, `util-linux` fails to configure with a missing `wchar_t` error.

### 3.3 `BR2_TOOLCHAIN_BUILDROOT_CXX=y`

Enables C++ support in the cross-compiler toolchain. While MiSTer's rootfs packages are mostly C, having C++ available allows adding packages that require it (e.g., certain WiFi firmware tools, RetroAchievements client libraries). The `Main_MiSTer` binary itself is C++.

---

## 4. System Identity

```ini
BR2_TARGET_GENERIC_HOSTNAME="mr-fusion"
BR2_TARGET_GENERIC_ISSUE="Welcome to Mr. Fusion"
```

These set the hostname and `/etc/issue` banner on the generated rootfs. During the first-boot install phase, the hostname is `mr-fusion`. After S99install-MiSTer.sh completes and the system reboots into the final MiSTer environment, the hostname changes to `MiSTer` (set by a post-install script in the release archive).

The `ISSUE` string appears on the Linux console (serial UART or framebuffer console) before the MiSTer `Main` binary takes over the display.

---

## 5. Rootfs Overlay

```ini
BR2_ROOTFS_OVERLAY="board/mr-fusion/rootfs-overlay"
```

Points Buildroot to the directory tree that gets copied into the rootfs after all packages are installed. For MiSTer, this contains **only** the first-boot install script:

```
board/mr-fusion/rootfs-overlay/
└── etc/
    └── init.d/
        └── S99install-MiSTer.sh
```

> [!WARNING]
> The overlay path is relative to the Buildroot source directory. If you reorganize the build tree, update this path. A missing overlay causes a silent build failure — Buildroot completes but the first-boot script is absent, and the SD card boots to a BusyBox shell with no MiSTer installation.

Source: `mr-fusion/builder/scripts/S99install-MiSTer.sh`

---

## 6. Packages

### 6.1 Core Build

```ini
BR2_PACKAGE_P7ZIP=y
```

`p7zip` provides the `7z` command-line tool. This is **the single most critical package** — it extracts the MiSTer `release_*.7z` archive during first boot. Without p7zip, the S99install script cannot unpack the MiSTer system files (`MiSTer` binary, `menu.rbf`, `MiSTer.ini`, core `.rbf` files).

> [!CAUTION]
> Removing `BR2_PACKAGE_P7ZIP=y` produces a rootfs that boots but cannot install MiSTer. The first-boot script will fail silently (the `7z` command is not found), and the system will remain in a permanent installer state.

### 6.2 Filesystem Tools

```ini
BR2_PACKAGE_EXFATPROGS=y          # Buildroot ≥ 2024.08
# Or for older Buildroot:
# BR2_PACKAGE_EXFAT=y
# BR2_PACKAGE_EXFAT_UTILS=y
```

Provides `mkfs.exfat` and `fsck.exfat`. During first boot, S99install-MiSTer.sh repartitions the SD card and creates the exFAT `MiSTer_Data` partition. exFAT is chosen over FAT32 because:
- **No 4 GB file size limit** — PSX and Saturn CD images routinely exceed 4 GB
- **Native kernel support** — `CONFIG_EXFAT_FS=y` in the kernel, no FUSE overhead

> [!WARNING]
> Buildroot 2024.08 removed `BR2_PACKAGE_EXFAT` and `BR2_PACKAGE_EXFAT_UTILS`, replacing them with `BR2_PACKAGE_EXFATPROGS`. If you use the upstream mr-fusion defconfig (pinned to 2024.02.1), keep the old options. If you upgrade Buildroot, you **must** switch to `BR2_PACKAGE_EXFATPROGS=y`.

### 6.3 System Utilities

```ini
BR2_PACKAGE_UTIL_LINUX=y
BR2_PACKAGE_UTIL_LINUX_BINARIES=y
```

Provides `mount`, `fdisk`, `blkid`, and other system utilities. The S99install-MiSTer.sh script uses:
- `fdisk` / `sfdisk` — repartition the SD card
- `blkid` — identify partition UUIDs
- `mount` — mount the FAT32 boot partition and the new exFAT data partition

`BR2_PACKAGE_UTIL_LINUX_BINARIES=y` installs the full binary versions (not just the BusyBox applet aliases). The BusyBox versions of `fdisk` and `blkid` lack the features needed for GPT probing and exFAT detection.

### 6.4 Framebuffer Display

```ini
BR2_PACKAGE_FBV=y
BR2_PACKAGE_FBV_PNG=y
BR2_PACKAGE_FBV_JPEG=y
BR2_PACKAGE_FBV_GIF=y
```

`fbv` (Framebuffer Viewer) displays the splash screen (`splash.png`) on the Linux framebuffer during boot. The sub-options enable PNG, JPEG, and GIF decoding. The mr-fusion splash screen is a PNG file bundled from `mr-fusion/vendor/support/splash.png`.

After the splash screen displays, the MiSTer `Main` binary takes over the framebuffer for the OSD menu. `fbv` is only used during the brief boot phase.

---

## 7. Output Format

```ini
BR2_TARGET_ROOTFS_CPIO=y
BR2_TARGET_ROOTFS_CPIO_GZIP=y
# BR2_TARGET_ROOTFS_TAR is not set
```

### 7.1 cpio Archive

`BR2_TARGET_ROOTFS_CPIO=y` generates `output/images/rootfs.cpio` — a cpio archive of the entire installer rootfs. This is the **format required by the installer** because it is embedded directly into the installer's kernel `zImage` via `CONFIG_INITRAMFS_SOURCE`.

### 7.2 gzip Compression

`BR2_TARGET_ROOTFS_CPIO_GZIP=y` compresses the cpio archive with gzip. The compressed initramfs is ~8 MB — small enough to fit in DDR3 RAM alongside the FPGA core and the kernel.

### 7.3 No TAR/EXT4 Output

`# BR2_TARGET_ROOTFS_TAR is not set` explicitly disables the tar rootfs output. A tar archive would serve no purpose here — the `mr-fusion` installer only consumes the cpio archive for initramfs embedding. (The runtime MiSTer OS loopback ext4 image is built separately).

---

## 8. What Is Not Enabled (and Why)

Several Buildroot options are **deliberately absent** from the mr-fusion defconfig:

| Option not set | Why |
|---|---|
| `BR2_PACKAGE_PYTHON3` | No Python interpreter — all scripts are BusyBox ash |
| `BR2_PACKAGE_PERL` | No Perl — unnecessary for the installer environment |
| `BR2_INIT_SYSTEMD` | No systemd — BusyBox `init` (`BR2_INIT_BUSYBOX=y`, the default) |
| `BR2_PACKAGE_XORG7` | No X11 — the OSD renders directly to `/dev/fb0` |
| `BR2_PACKAGE_OPENSSL` | No OpenSSL at build time — `wget` handles HTTPS internally |
| `BR2_PACKAGE_WPA_SUPPLICANT` | No WiFi in the installer — WiFi setup happens post-install in the runtime OS |
| `BR2_TARGET_ROOTFS_EXT2` | No ext2/3/4 rootfs — The installer uses an initramfs. The runtime MiSTer OS uses an ext4 loopback image (`linux.img`), which is built separately outside this defconfig. |

---

## 9. Defconfig Compatibility Matrix

| Buildroot Version | `EXFAT` Options | `exfatprogs` | Kernel `CONFIG_EXFAT_FS` | Status |
|---|---|---|---|---|
| 2024.02.1 (mr-fusion pinned) | `BR2_PACKAGE_EXFAT=y` + `BR2_PACKAGE_EXFAT_UTILS=y` | Not available | Not required (FUSE helper) | ✅ Works |
| 2024.08 – 2024.11 | Not available | `BR2_PACKAGE_EXFATPROGS=y` | Required | ✅ Works (with patch) |
| 2025.02 – 2026.02 | Not available | `BR2_PACKAGE_EXFATPROGS=y` | Required | ✅ Works (with patch) |

> [!NOTE]
> The upstream mr-fusion defconfig pins Buildroot 2024.02.1 and does **not** use `exfatprogs`. The comprehensive guide's Dockerfile applies `sed` patches to migrate to modern Buildroot. If you're building from scratch, use Buildroot ≥ 2024.08 with the patched defconfig shown in this article.

---

## 10. Cross-References

- [Buildroot Overview](buildroot_overview.md) — Architecture, package model, output structure, cross-toolchain
- [Custom Packages](custom_packages.md) — Adding custom tools and recovery scripts to the installer
- [Image Generation](image_generation.md) — SD card partition layout and image assembly
- [Custom Image Guide](custom_image_guide.md) — End-to-end build pipeline
- [FPGA KB — Cyclone V HPS](https://github.com/alfishe/fpga-bootcamp/blob/main/02_architecture/soc/hps_fpga_intel_soc.md) — HPS architecture reference
- [mr-fusion — buildroot-defconfig](https://github.com/MiSTer-devel/mr-fusion/blob/master/builder/config/buildroot-defconfig)

---

## 11. References

| Source | Path / URL |
|---|---|
| MiSTer Buildroot defconfig | `mr-fusion/builder/config/buildroot-defconfig` |
| MiSTer Kernel defconfig | `mr-fusion/builder/config/kernel-defconfig` |
| First-boot install script | `mr-fusion/builder/scripts/S99install-MiSTer.sh` |
| Buildroot Manual — Configuration | [buildroot.org/manual.html#configure](https://buildroot.org/downloads/manual/manual.html#configure) |
| Cyclone V HPS Technical Reference Manual | [Intel 683126](https://www.intel.com/content/www/us/en/docs/programmable/683126/) |
