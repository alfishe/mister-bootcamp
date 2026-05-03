# 07 — FPGA Cores Architecture

This section documents the internal architectural patterns and logic designs of specific emulation cores. While the **[FPGA Subsystem](../06_fpga_subsystem/README.md)** defines the "Shell" (I/O, scaling, memory), this section focuses on the "Core" logic.

## Articles

*   *(Coming Soon: CPU Bus Arbitration Patterns)*
*   *(Coming Soon: Video Generation Logic - Scanline vs. Buffer)*
*   *(Coming Soon: Sound Chip Implementation Hazards)*

## Reference Documentation
Before developing a core, you must understand the MiSTer framework:
*   [**Framework Overview**](../06_fpga_subsystem/fpga_framework_overview.md) — Architectural goals and Shell/Core split.
*   [**Performance Metrics**](../06_fpga_subsystem/fpga_performance_metrics.md) — Resource limits (ALMs, DSPs) for the Cyclone V.
*   [**Compilation Guide**](../06_fpga_subsystem/fpga_compilation_guide.md) — How to achieve timing closure for new cores.

## Subdirectories

*   [**build/**](build/) — Documentation for specific core build scripts and automation.
