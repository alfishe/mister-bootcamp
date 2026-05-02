[← Buildroot Index](README.md) · [↑ Linux System](../README.md) · [↑ Knowledge Base](../../README.md)

# Custom Packages

How to extend the `mr-fusion` installer rootfs with custom recovery tools, why you **should not** use `mr-fusion` for runtime dependencies like WiFi or RetroAchievements, and the general pattern for adding any Buildroot package to the installer.

> [!NOTE]
> This article assumes you've read the [Buildroot Overview](buildroot_overview.md) and [MiSTer Defconfig Walkthrough](mister_defconfig.md). You should understand the defconfig model and the rootfs overlay mechanism before adding custom packages.

---

## 1. Two Approaches to Customization

Buildroot offers two mechanisms for adding content to the rootfs. Choose based on what you're adding:

| Mechanism | Use when | Example |
|---|---|---|
| **Rootfs overlay** | Adding shell scripts, config files, or pre-compiled binaries | `wifi.sh`, custom `/etc/inittab`, gamecontrollerdb.txt |
| **Buildroot package** | Compiling from source, or when the package has library dependencies | `wpa_supplicant`, `libcurl`, `libretro-common` |

### 1.1 Rootfs Overlay (Simpler)

Files placed under `BR2_ROOTFS_OVERLAY` are copied verbatim into the rootfs tree. This is how mr-fusion injects `S99install-MiSTer.sh`. To add a custom script:

```bash
mkdir -p buildroot/board/mr-fusion/rootfs-overlay/etc/init.d/
cp my-custom-script.sh buildroot/board/mr-fusion/rootfs-overlay/etc/init.d/
```

No Buildroot package definition needed. The file appears in `output/target/` and then in the final `rootfs.cpio`.

### 1.2 Buildroot Package (Compiled Dependencies)

When a package requires compilation or has library dependencies, you must use Buildroot's package infrastructure. There are two sub-approaches:

| Sub-approach | When to use |
|---|---|
| **Enable an existing Buildroot package** | The package already exists in Buildroot's `package/` tree (e.g., `wpa_supplicant`, `libcurl`) |
| **Create a new package** | The package doesn't exist in Buildroot, or you need a custom version |

Source: [Buildroot Manual — Adding Packages](https://buildroot.org/downloads/manual/manual.html#adding-packages)

---

## 2. The Runtime Modification Fallacy

It is a common architectural misconception that you should add runtime packages (like `wpa_supplicant` for WiFi, or `libcurl` for RetroAchievements) to the `mr-fusion` Buildroot configuration.

As detailed in the [Buildroot Overview](buildroot_overview.md), `mr-fusion` is **only the ephemeral installer**. Any packages or firmware you add to the `mr-fusion` defconfig will only be available during the initial ~30-second SD card partitioning phase. Once the system reboots into the actual MiSTer OS (the `linux.img` loopback ext4 filesystem), all of those `mr-fusion` packages will vanish.

If your goal is to add permanent packages (like custom WiFi firmware, VPN daemons, or RetroAchievements dependencies) to the runtime MiSTer OS, **do not use the `mr-fusion` Buildroot**. Instead, use one of the following methods:

1. **Pacman / Entware**: Use the built-in package managers on a running MiSTer system.
2. **Persistence Hooks**: Add download or setup commands to `/media/fat/linux/user-startup.sh`, which executes on every boot in the runtime OS.
3. **Direct loopback modification**: Mount the `linux.img` file on a host PC (or in a chroot), install the packages, unmount, and re-pack your custom release archive.

Modifying the `mr-fusion` Buildroot is strictly for adding tools to the **installation phase** (e.g., custom network recovery tools, specific filesystem formatters, or automated pre-install hooks).

---

## 3. General Pattern: Adding a Package to the Installer

If you do need to add a package to the `mr-fusion` installer, all Buildroot packages follow the same integration pattern. Here's the step-by-step workflow:

### 3.1 Find the Package

List available packages:

```bash
cd buildroot
make list-defconfigs | grep <package-name>
```

Or browse `package/<name>/Config.in` to find the option symbol.

### 3.2 Add to Defconfig

Append the option to your defconfig:

```bash
echo 'BR2_PACKAGE_<NAME>=y' >> configs/mr-fusion_defconfig
```

### 3.3 Resolve Dependencies

`make menuconfig` shows dependency resolution. Run it interactively to verify:

```bash
cd buildroot
make mr-fusion_defconfig
make menuconfig
```

Navigate to the package category and check that all dependencies are satisfied. Buildroot's Kconfig will gray out or hide packages with unmet dependencies.

### 3.4 Update the Defconfig

After verifying in `menuconfig`, extract only your changes:

```bash
make savedefconfig DEFCONFIG=configs/mr-fusion_defconfig
```

This rewrites the defconfig with only the non-default options — keeping it minimal.

### 3.5 Rebuild

```bash
make -j$(nproc)
```

The rootfs will now include the new package(s).

---

## 4. Installer Package Examples

### 4.1 SSH Server (Dropbear)

For remote shell access without relying on the post-install SSH setup:

```ini
BR2_PACKAGE_DROPBEAR=y
```

Dropbear is a lightweight SSH server (~200 KB) ideal for embedded systems. After boot, `dropbear` listens on port 22 with root password `1` (MiSTer default).

### 4.2 NTP Client

For accurate timestamps (useful for RetroAchievements which validate server timestamps):

```ini
BR2_PACKAGE_NTP=y
BR2_PACKAGE_NTP_NTPDATE=y
```

Alternative: `BR2_PACKAGE_CHRONY=y` (lighter weight, better for intermittent connections).

### 4.3 Game Controller Database

The SDL game controller database (`gamecontrollerdb.txt`) is not a Buildroot package — it's a data file bundled during image assembly:

```bash
curl -LsS -o gamecontrollerdb.txt \
    "https://raw.githubusercontent.com/MiSTer-devel/Distribution_MiSTer/main/linux/gamecontrollerdb/gamecontrollerdb.txt"
```

Source: `mr-fusion/Dockerfile`

---

## 5. Rootfs Size Budget

Every added package increases the compressed initramfs size. MiSTer's initramfs is loaded into DDR3 RAM alongside:

| Consumer | Approximate size |
|---|---|
| **Linux kernel (zImage)** | ~4 MB |
| **Rootfs (compressed cpio)** | ~8 MB (stock mr-fusion) |
| **FPGA core bitstream** | 2–8 MB (varies by core) |
| **Core memory (BRAM, SDRAM mapping)** | Core-dependent |
| **User data buffers** | Dynamic |

The DE10-Nano has 1 GB of DDR3. In practice, the initramfs has generous headroom, but aim to keep the compressed rootfs under **16 MB** to leave maximum room for large FPGA cores (ao486, PSX, N64).

### Size Reduction Tips

| Technique | Savings |
|---|---|
| Disable `BR2_PACKAGE_UTIL_LINUX_BINARIES` (use BusyBox applets) | ~1.5 MB |
| Use `BR2_PACKAGE_DROPBEAR_SMALL` instead of full Dropbear | ~100 KB |
| Strip debug symbols: `BR2_STRIP_strip=y` (default) | Variable |
| Use `BR2_OPTIMIZE_2=y` (`-Os` — optimize for size) | ~10–20% code reduction |

---

## 6. Testing Custom Packages

### 6.1 Verify Package Inclusion

After the Buildroot build, check that the package made it into the target tree:

```bash
ls buildroot/output/target/usr/bin/<binary-name>
# Or for libraries:
ls buildroot/output/target/lib/<library-name>.so*
```

### 6.2 Verify Initramfs Content

```bash
mkdir -p /tmp/rootfs && cd /tmp/rootfs
zcat buildroot/output/images/rootfs.cpio | cpio -idmv
ls -la etc/init.d/   # Check overlay scripts
ls -la usr/bin/       # Check package binaries
```

### 6.3 Runtime Verification

After booting the custom installer image on the DE10-Nano:

```bash
# Check package binaries
which wpa_supplicant
wpa_supplicant -v

# Check kernel modules
lsmod | grep rt2

# Check firmware loaded
dmesg | grep firmware

# Check disk usage
df -h /
free -m
```

---

## 7. Cross-References

- [Buildroot Overview](buildroot_overview.md) — Package model and output structure
- [MiSTer Defconfig Walkthrough](mister_defconfig.md) — Line-by-line defconfig reference
- [WiFi & Network Setup](../../12_networking/wifi_setup.md) — Post-install WiFi configuration
- [RetroAchievements](../../14_extensions/retroachievements.md) — RA integration architecture
- [Buildroot Manual — Adding Packages](https://buildroot.org/downloads/manual/manual.html#adding-packages)

---

## 8. References

| Source | Path / URL |
|---|---|
| MiSTer WiFi script | [`Scripts_MiSTER/other_authors/wifi.sh`](https://github.com/MiSTer-devel/Scripts_MiSTer/blob/master/other_authors/wifi.sh) |
| RetroAchievements fork | [`odelot/Main_MiSTer`](https://github.com/odelot/Main_MiSTer) |
| Game controller DB | [`Distribution_MiSTer/gamecontrollerdb.txt`](https://github.com/MiSTer-devel/Distribution_MiSTer) |
| mr-fusion Dockerfile | [`mr-fusion/Dockerfile`](https://github.com/MiSTer-devel/mr-fusion/blob/master/Dockerfile) |
| Buildroot Package Infrastructure | [buildroot.org/manual.html#adding-packages](https://buildroot.org/downloads/manual/manual.html#adding-packages) |
