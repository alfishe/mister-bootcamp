[← Knowledge Base](../README.md)

# Configuration

The two-layer configuration system: CONF_STR (compiled into the FPGA bitstream, declares the OSD menu and maps it to 128-bit status bits) and MiSTer.ini (SD card INI file controlling system-wide hardware settings). Plus per-core .CFG persistence and the OSD rendering pipeline.

## Articles

| File | Topic | Quality |
|---|---|---|
| [conf_str.md](conf_str.md) | CONF_STR: RTL-embedded OSD menu declaration, status[127:0], parse_config() | 📋 Target: Deep |
| [mister_ini.md](mister_ini.md) | MiSTer.ini: system INI file, section matching, variable reference | 📋 Target: Deep |
| [mister_ini_guide.md](mister_ini_guide.md) | User-facing guide: video modes, filters, per-core overrides, common recipes | 📋 Target: Adequate |
| [osd.md](osd.md) | OSD rendering pipeline, menu navigation, dynamic show/hide masks | 📋 Target: Adequate |

## Cross-References

- [Framework — hps_io.sv](../02_framework/hps_io_module.md) — opcode dispatch for status register
- [HPS Binary](../04_hps_binary/architecture.md) — parse_config(), cfg_parse() implementation
