[← Networking](README.md) · [↑ Knowledge Base](../README.md)

# Remote Management

> Control your MiSTer from any device on your network — phone, tablet, laptop. The primary tool is **mrext/Remote** by wizzomafizzo, a full-featured web interface for OSD navigation, core launching, file management, and screen capture.

Sources: [`wizzomafizzo/mrext`](https://github.com/wizzomafizzo/mrext) · [MiSTer Remote (Google Play)](https://play.google.com/store/apps/details?id=net.mrext.remote)

---

## Remote — Web Interface

A web-based control panel accessible from any browser on the same network.

### Features

| Feature | Description |
|---|---|
| **OSD navigation** | Full on-screen display control from browser |
| **Core launching** | Browse and launch cores and games |
| **Screen capture** | Live screenshot of current display |
| **File browser** | Navigate SD card, manage ROMs and saves |
| **Search** | Find and launch games from your collection |
| **Configuration** | Edit MiSTer.ini and other config files |
| **System info** | CPU temp, uptime, disk usage |

### Installation

Add to `downloader.ini`:

```ini
[mrext/all]
db_url = https://github.com/wizzomafizzo/mrext/raw/main/releases/all.json
```

Or install individual scripts from the [releases page](https://github.com/wizzomafizzo/mrext/releases).

### Access

Open a browser on any device and navigate to:
```
http://<mister-ip-address>
```

The web interface auto-starts with the MiSTer system.

---

## MiSTer Remote — Android App

A dedicated Android client for MiSTer control.

| Feature | Description |
|---|---|
| **OSD control** | Virtual D-pad and buttons |
| **Core/game launching** | Browse and launch from phone |
| **Keyboard input** | Type text into MiSTer from phone keyboard |
| **Settings editor** | Modify MiSTer settings remotely |

Available on [Google Play](https://play.google.com/store/apps/details?id=net.mrext.remote).

---

## Other Remote Access Methods

### SSH

Full shell access for advanced users:

```bash
ssh root@<mister-ip>
# Default password: 1
```

Common tasks via SSH:
- Edit configuration files
- Run update scripts manually
- Debug controller/input issues
- Monitor system logs

### FTP / SFTP

File transfer for ROM management:

| Protocol | Port | Client |
|---|---|---|
| **FTP** | 21 | FileZilla, WinSCP |
| **SFTP** | 22 | FileZilla, WinSCP, Cyberduck |

### Samba (SMB)

Network file share — MiSTer's SD card appears as a network drive:

```ini
; Enable in MiSTer.ini
smbshare=sdf
```

### reMiSTer

Use your PC keyboard over the network to control MiSTer — useful for typing in computer cores.

Source: [reMiSTer](https://github.com/wizzomafizzo/mrext)

### TCP Input Server

A server daemon that monitors TCP commands and emulates keystrokes — useful for automation and home theater integration.

---

## Headless Operation

MiSTer can run entirely without a display connected to HDMI:

1. Connect via Ethernet or WiFi
2. Use Remote web interface or SSH for all management
3. Audio can be routed to HDMI (if display supports it) or analog output

> [!NOTE]
> Some cores may behave differently without an HDMI display connected. The HDMI scaler initializes based on detected EDID. Use a HDMI dummy plug (EDID emulator) if you experience issues.

---

## Cross-References

| Topic | Article |
|---|---|
| WiFi setup | [WiFi Setup](wifi_setup.md) |
| Update scripts | [Update Scripts](../15_ecosystem/update_scripts.md) |
| OSD architecture | [OSD](../05_configuration/osd.md) |
| MiSTer.ini settings | [INI Guide](../05_configuration/mister_ini_guide.md) |
