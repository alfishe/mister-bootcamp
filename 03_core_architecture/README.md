[← Knowledge Base](../README.md)

# Core Architecture

How to build a MiSTer core — from the Template through to a compiled RBF bitstream. Covers the emu module pattern, Quartus project structure, and build tooling across all platforms.

## Articles

| File | Topic | Quality |
|---|---|---|
| [template_walkthrough.md](template_walkthrough.md) | Step-by-step guide to building a core from `Template_MiSTer` | 📋 Target: Deep |

## Build Tooling

| File | Topic | Quality |
|---|---|---|
| [build/overview.md](build/overview.md) | FPGA build pipeline: QPF → Synthesis → Fitter → Assembler → RBF | 📋 Target: Deep |
| [build/native_linux.md](build/native_linux.md) | Quartus 17.1 Lite on Ubuntu/Debian | 📋 Target: Adequate |
| [build/native_windows.md](build/native_windows.md) | Quartus 17.1 on Windows (GUI + CLI) | 📋 Target: Adequate |
| [build/docker_x86.md](build/docker_x86.md) | Docker build with `raetro/quartus:17.1` | 📋 Target: Adequate |
| [build/docker_apple_silicon.md](build/docker_apple_silicon.md) | Quartus on Apple Silicon via Rosetta 2 (`quartus-on-docker`) | 📋 Target: Deep |
| [build/ci_automation.md](build/ci_automation.md) | GitHub Actions CI for core compilation | 📋 Target: Adequate |

## Cross-References

- [Framework](../02_framework/README.md) — the `sys/` modules every core instantiates
- [HPS Binary](../04_hps_binary/README.md) — the ARM-side binary that communicates with cores
- [FPGA KB — Design Flow](https://github.com/alfishe/fpga-bootcamp/blob/main/03_design_flow/overview.md)
