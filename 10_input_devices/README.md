[← Knowledge Base](../README.md)

# Input Devices

Input handling from USB HID devices through the HPS binary to FPGA core registers. Covers keyboard, mouse, joystick/gamepad, and low-latency direct-wired controllers (SNAC/LLAPI).

## Articles

| File | Topic | Quality |
|---|---|---|
| [keyboard.md](keyboard.md) | PS/2 keyboard emulation via USB HID translation | 📋 Target: Deep |
| [mouse.md](mouse.md) | PS/2 mouse emulation, sensitivity, sniper mode | 📋 Target: Adequate |
| [joystick.md](joystick.md) | Joystick/gamepad: USB HID → 32-bit button word, analog sticks, autofire | 📋 Target: Adequate |
| [snac_llapi.md](snac_llapi.md) | SNAC & LLAPI: direct DB9/NES/SNES controller wiring to FPGA pins | 📋 Target: Deep |

## Cross-References

- [Framework — hps_io.sv](../06_fpga_subsystem/hps_io_module.md) — UIO opcodes for input events
- [References — UIO Commands](../17_references/uio_command_reference.md) — joystick/keyboard/mouse opcodes
