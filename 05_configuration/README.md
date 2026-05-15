# 05 — Configuration Subsystem

[← Back to Index](../README.md)

This section documents the configuration and UI overlay subsystems that bridge the HPS host software and the FPGA hardware.

## Articles

| Article | Topic | Quality |
|---|---|---|
| [Core Configuration String (CONF_STR)](conf_str.md) | The developer reference for the RTL-side configuration syntax, OSD menu declarations, and the 128-bit `status` register architecture. | ✅ Deep |
| [MiSTer INI Guide](mister_ini_guide.md) | The user reference for global `MiSTer.ini` behavior, per-core section overrides, and parsed variables. | ✅ Deep |
| [OSD Architecture](osd.md) | How the FPGA renders the menu overlay over the core's video output using character bitmaps sent from the HPS. | 🔀 Adequate |
| [CRT & Analog Video Setup](crt_setup.md) | CRT connection guide: IO board vs Direct Video, cables, mister.ini settings, troubleshooting | Adequate |
