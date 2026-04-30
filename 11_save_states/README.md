[← Knowledge Base](../README.md)

# Save States

MiSTer's save state system uses DDR3 as a snapshot buffer — the entire core state (CPU registers, VRAM, WRAM, audio state) is serialized to a DDR3 region and can be restored on demand. This section covers the architecture, the SSPI protocol for save/load, per-core implementation requirements, and the rewind feature.

## Planned Articles

| File | Topic | Quality |
|---|---|---|
| [save_state_architecture.md](save_state_architecture.md) | DDR3 snapshot/restore, SSPI protocol, CONF_STR `SS` declaration, per-core requirements, rewind | 📋 Target: Deep |
