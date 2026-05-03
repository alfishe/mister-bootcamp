# Video & Audio Subsystem

This directory contains deep-dive documentation and reference implementations for the MiSTer Video and Audio signal processing pipelines.

## 1. Video Processing & Scaling

| Document | Quality | Description |
|----------|---------|-------------|
| [ASCAL Deep Dive](ascal_deep_dive.md) | **Deep** | Complete architectural analysis of the Avalon Scaler — clock domains, CDC, data flow, interpolation algorithms, triple buffering, and all 17 functional blocks |
| [Polyphase Scaling Theory](polyphase_scaling_theory.md) | Deep | Mathematical foundations of video scaling, FIR filters, and coefficient vectors |
| [ascal Architecture](ascal_architecture.md) | Adequate | Overview of the VHDL and SystemVerilog implementations |
| [Video Audio Pipelines](../06_fpga_subsystem/video_audio_pipelines.md) | Adequate | How A/V signals flow through the FPGA framework |

## 2. Audio Processing

| Document | Quality | Description |
|----------|---------|-------------|
| [Audio Pipeline Overview](audio_pipeline_overview.md) | Adequate | Sigma-Delta modulation, I2S interfacing, and audio mixing |

## 3. Reference Implementations

| File | Lines | Description |
|------|-------|-------------|
| `ascal.sv` | 2414 | SystemVerilog port of the polyphase scaler |
| `ascal.vhd` | 3057 | Original MiSTer VHDL polyphase scaler (TEMLIB) |

## 4. Cross-References

- [FPGA KB — Intel HPS-FPGA Integration](https://github.com/alfishe/fpga-knowledge-base) — Cyclone V SoC architecture
- [MiSTer Wiki — Video Modes](https://github.com/MiSTer-devel/Main_MiSTer/wiki/Video-modes) — User-facing configuration
