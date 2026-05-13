# 10 — Input Devices

[← Back to Index](../README.md)

Input paths from USB/BT peripherals through the HPS to the FPGA core.

---

## Articles

| File | Topic | Quality |
|---|---|---|
| [snac_llapi.md](snac_llapi.md) | SNAC direct controller wiring, per-core OEM protocols, LLAPI wireless, level shifting | Deep |
| [joystick.md](joystick.md) | USB gamepad → 32-bit digital word, analog sticks, rumble, button mapping | Adequate |
| [keyboard.md](keyboard.md) | Linux evdev → PS/2 scan code emulation → FPGA keyboard port | Adequate |
| [mouse.md](mouse.md) | Linux evdev → PS/2 mouse packet emulation → FPGA mouse port | Adequate |

---

## Related Sections

* [06 — FPGA Subsystem](../06_fpga_subsystem/README.md) — `hps_io.sv` input decoders
* [17 — References](../17_references/README.md) — UIO opcode tables for input commands
