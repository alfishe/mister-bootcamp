[← Ecosystem](README.md) · [↑ Knowledge Base](../README.md)

# Ecosystem Software & Tools

> The MiSTer ecosystem extends well beyond the core framework — scripts, frontends, ports of emulators, attract modes, and community utilities that make the platform more usable and fun.

Sources: [`wizzomafizzo/mrext`](https://github.com/wizzomafizzo/mrext) · [MiSTer-devel Scripts](https://github.com/MiSTer-devel/mister-scripts)

---

## mrext — MiSTer Extensions

A collection of scripts by wizzomafizzo, installable via Update All or downloader.

| Tool | Purpose |
|---|---|
| **Remote** | Web-based control from any browser — OSD, file browser, screen capture |
| **BGM** | Background music player in the MiSTer menu (pauses during gameplay) |
| **Favorites** | Create game shortcuts in the main menu |
| **GamesMenu** | Auto-index all games and generate browseable menu structure |
| **LaunchSync** | Shareable/subscribable game playlists |
| **LastPlayed** | Dynamic shortcuts for recently played games |
| **PlayLog** | Track and report play history |
| **Random** | Launch a random game from your collection |
| **Search** | Fast game search and launch |
| **NFC** | Launch games by tapping NFC tags |

Install all at once via `downloader.ini`:
```ini
[mrext/all]
db_url = https://github.com/wizzomafizzo/mrext/raw/main/releases/all.json
```

---

## Frontend & Display Tools

| Tool | Description |
|---|---|
| **MiSTer SAM** (Super Attract Mode) | Idle attract screen — randomly plays games, jump in and play |
| **Insert Coin** | Alternative layout for the Arcade folder |
| **MiSTerWallpapers** | Auto-download wallpaper collection for HDMI display |
| **MiSTer-CRT-Wallpapers** | Same, optimized for 4:3 CRT screens |
| **MiSTress** | RSS reader — display latest core updates on wallpaper |
| **VIDEO PRESETS by Robby** | Curated video presets per core for optimal display |

---

## Software Ports

Native Linux applications running on the ARM HPS alongside FPGA cores:

| Port | Description |
|---|---|
| **MiSTer ScummVM** | Point-and-click adventure engine — runs well, covers games beyond ao486 reach |
| **MiSTer DOSBox** | DOS emulator — complements ao486 for edge-case games |
| **MiSTer PrBoom-Plus** | Enhanced Doom engine with massive expansion support |
| **MiSTer Basilisk II** | 68k Macintosh emulator |
| **DevilutionX** | Diablo engine port (via YARMUS installer) |
| **Cave Story** | Doukutsu port (via YARMUS installer) |

---

## System Utilities

| Tool | Description |
|---|---|
| **MiSTer Batch Control** | CLI for low-level functions not accessible via scripting |
| **MiSTerTools** | Aspect ratio calculator, modeline↔video_mode converter, INI profile switcher, MRA parser |
| **Migrate SD** | Clone your entire SD card to a new one from MiSTer itself |
| **Overclock Scripts** | Overclock ARM for full-speed Munt (MT-32) and extra ScummVM performance |
| **reMiSTer** | Use your PC keyboard over the network |
| **Official Scripts** | Miscellaneous system task scripts from MiSTer-devel |

---

## PC-Side Tools

| Tool | Platform | Description |
|---|---|---|
| **AMMiSTer** | Windows/Mac/Linux | Arcade game manager — updates, bulk management, favorites, metadata |
| **PC Launcher** | Any | Download all MiSTer files on PC for offline transfer |
| **mistercon** | Android | Browse collection and launch games from phone |
| **YARMUS** | MiSTer | One-shot installer for many tools listed above |

---

## ROM Management

| Tool | Description |
|---|---|
| **MGL Core Setnames** | Modified core shortcuts for automatic alternate configs |
| **mister-boot-roms** | MiSTer-themed boot screens for cores with loadable boot ROMs |
| **DOS Shareware Updater** | Update Flynn's DOS Shareware pack for ao486 |
| **Top 300 Updater** | Update Flynn's Top 300 DOS pack for ao486 |

---

## Cross-References

| Topic | Article |
|---|---|
| Update All & Downloader | [Update Scripts](update_scripts.md) |
| Remote web control | [Remote Management](../12_networking/remote_management.md) |
| Alternative distributions | [Alternative Distros](../03_hps_linux/alternative_distros.md) |
| MiSTer.ini configuration | [INI Guide](../05_configuration/mister_ini_guide.md) |
