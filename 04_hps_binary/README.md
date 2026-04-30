[← Knowledge Base](../README.md)

# HPS Binary (`Main_MiSTer`)

The ARM-side C++ binary that runs on the Cyclone V HPS under Linux. It handles core loading, OSD menu, input routing, SD card I/O, video scaling, audio mixing, and all user-facing functionality.

## Articles

| File | Topic | Quality |
|---|---|---|
| [architecture.md](architecture.md) | Module map, main loop, scheduler, key data structures | 📋 Target: Deep |

## Build Tooling

| File | Topic | Quality |
|---|---|---|
| [build/overview.md](build/overview.md) | ARM cross-compilation toolchain, Makefile structure | 📋 Target: Deep |
| [build/native_linux.md](build/native_linux.md) | `arm-linux-gnueabihf-g++` on Ubuntu | 📋 Target: Adequate |
| [build/docker_cross.md](build/docker_cross.md) | Docker-based ARM cross-compile (all platforms) | 📋 Target: Adequate |
| [build/windows_wsl2.md](build/windows_wsl2.md) | WSL2 cross-compilation workflow | 📋 Target: Adequate |
| [build/macos.md](build/macos.md) | macOS + Apple Silicon ARM cross-compile | 📋 Target: Adequate |

## Cross-References

- [Framework](../02_framework/README.md) — FPGA-side modules the HPS communicates with
- [Configuration](../09_configuration/README.md) — CONF_STR, MiSTer.ini, .CFG files
- [References — UIO Opcodes](../15_references/uio_command_reference.md) — full command protocol
