[← Input Devices](README.md) · [↑ Knowledge Base](../README.md)

# Controller Setup & Mapping

> Everything you need to know about connecting, configuring, and troubleshooting controllers on MiSTer — from 8bitdo Bluetooth pads to SNAC OEM hardware, lightguns, mice, and arcade encoders.

Sources: [MiSTer Input Documentation](https://mister-devel.github.io/MkDocs_MiSTer/basics/input/)

---

## Player Assignment

MiSTer uses a press-to-assign system for multi-player:

1. **Start a core** — all controllers are unassigned
2. **Press a button** on controller 1 → assigned as Player 1
3. **Press a button** on controller 2 → assigned as Player 2
4. Continue for Players 3–6 (framework supports up to 6)
5. **Reset**: OSD secondary menu (press Right in OSD) → "Reset player assignment"

> [!NOTE]
> If a controller doesn't respond, reboot into the Menu core and define its button mapping first. Two controllers with identical Vendor ID / Product ID may conflict.

### Advanced (Manual) Assignment

For permanent player assignment (e.g., arcade cabinets), configure `MiSTer.ini` by device identifier:

| Method | Use Case |
|---|---|
| **Bluetooth MAC address** | Wireless controllers |
| **VendorID + ProductID** | Wired USB controllers |
| **USB port path** | Multiple identical controllers |

Enable `debug=1` in `MiSTer.ini`, SSH in as root, run `killall MiSTer; /media/fat/MiSTer` and watch the debug output for device identifiers.

---

## Supported Device Types

| Type | Examples | Notes |
|---|---|---|
| **USB HID gamepads** | DualSense, Xbox, 8bitdo, SN30 Pro | Plug and play |
| **Bluetooth controllers** | 8bitdo, DualShock 4, DualSense | Pair via `Scripts/setup_bluetooth.sh` |
| **USB retro adapters** | Raphnet, Daemonbite | HID-compatible, auto-detected |
| **Arcade encoders** | Zero Delay, IPAC | USB HID, treat as joystick/keyboard |
| **SNAC** | Original OEM controllers | Sub-µs latency via IO board USER_IO pins |
| **USB lightguns** | GUN4IR, GunCon 2/3, Wiimote | Core-specific mapping required |
| **Mouse** | USB, Bluetooth, Wiimote | Emulates lightgun, paddle, or pointing device |
| **Keyboard** | Any USB HID keyboard | PS/2 emulation, F-keys for core hotkeys |
| **Spinners / trackballs** | USB HID spinner, TurboTwist | Mouse-mode or analog axis |

---

## Button Definition

Controllers must be defined in the **Menu core** before use:

1. Boot into Menu core
2. Hold a button on the controller — MiSTer detects it
3. OSD prompts you to map each function: Up, Down, Left, Right, A, B, X, Y, L, R, Start, Select, etc.
4. Definitions are saved and persist across cores

---

## Auto Fire

Any button (except D-pad) supports configurable auto fire:

| Action | How |
|---|---|
| **Enable** | Hold target button + press BUTTON OSD |
| **Cycle rate** | Repeat enable to step through rates |
| **Disable** | Step through until "disabled" message |

### Custom Rates (`MiSTer.ini`)

```ini
; Preset rates in Hz
autofire_rates=7.5,10,15,20,30

; Or binary pattern (0=release, 1=hold, per frame)
autofire_rates=0b000011111
; 5 frames held, 4 frames released, repeat
```

---

## Mouse Emulation from Controller

Any gamepad with enough buttons can emulate a mouse:

| Action | Control |
|---|---|
| **Activate** | Hold "Mouse Emu" button |
| **Move pointer** | Left analog stick (while Mouse Emu held) |
| **Left/Right/Middle click** | Defined mouse buttons |
| **Toggle permanent** | Hold Mouse Emu + press BUTTON OSD |
| **Sniper mode** | In permanent mode, Mouse Emu becomes precision/slow |

> [!NOTE]
> Only mouse-mapped buttons switch. Remaining gamepad buttons still function as joystick — useful for games like Amiga's *Walker* that need both.

---

## Lightgun Setup

### USB / Bluetooth Lightguns

Supported: GUN4IR, GunCon 2, GunCon 3, Wiimote (as mouse/lightgun)

**Calibration** (required per core):
1. Open OSD → press **F10** to show calibration screen
2. Shoot each edge of the image as prompted
3. Calibrate to game screen edges, not full widescreen TV

**Player assignment** — match lightgun to core's expected port:

| System | Lightgun Port | Notes |
|---|---|---|
| NES (Zapper) | Joy 2 | Some games require P1 = regular controller |
| SNES (Super Scope) | Joy 2 | Set "Super Scope Btn: Joy" in input options |
| Genesis (Menacer/Justifier) | Joy 2 | Set "Gun Control: Joy2, Gun Fire: Joy" |
| Master System (Phaser) | Joy 1 | Set "Gun Port: Port 1" |
| PSX (GunCon) | Joy 1 or Joy 2 | Pad1: GunCon in OSD |
| Atari 7800 (XG-1) | Joy 1 | Enable "Show Overscan: Yes" |

### Wiimote as Lightgun

Pair via Bluetooth → treated as mouse → used as positional gun with sensor bar. Requires a powered sensor bar.

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---|---|---|
| Controller not detected | Not defined in Menu core | Boot to Menu core, define buttons |
| 8bitdo won't pair on USB hub | USB hub power issue | Try different port or powered hub |
| Two identical controllers conflict | Same VID/PID | Use USB port path in `MiSTer.ini` |
| Bluetooth pairing fails | Wrong pairing mode | Use `Scripts/setup_bluetooth.sh` |
| Buttons wrong in-game | Wrong core mapping | Redefine in Menu core or check core OSD |
| Lightgun off-target | Not calibrated | Calibrate per core (F10 in OSD) |
| D-pad acts as analog | Controller mode switch | Toggle controller between D-pad/X-input mode |

---

## Cross-References

| Topic | Article |
|---|---|
| SNAC direct controller wiring | [SNAC & LLAPI](snac_llapi.md) |
| Joystick protocol (hps_io.sv) | [HPS IO Module](../06_fpga_subsystem/hps_io_module.md) |
| Input latency analysis | [Input Latency & SNAC](../06_fpga_subsystem/input_latency_and_snac.md) |
| Keyboard PS/2 emulation | [Keyboard](keyboard.md) |
| Mouse PS/2 emulation | [Mouse](mouse.md) |
| OSD menu system | [OSD](../05_configuration/osd.md) |
