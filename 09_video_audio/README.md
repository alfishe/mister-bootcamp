# Video & Audio Subsystem

This directory contains deep-dive documentation and reference implementations for the MiSTer Video and Audio signal processing pipelines.

## 1. Video Processing & Scaling

| Document | Quality | Description |
|----------|---------|-------------|
| [Video Mixer Deep Dive](video_mixer_deep_dive.md) | **Deep** | Source-grounded analysis of `video_freezer`, `gamma_corr` (3-cycle LUT + `gamma_fast`), scandoubler, Hq2x, and output selection |
| [ASCAL Deep Dive](ascal_deep_dive.md) | **Deep** | Complete architectural analysis of the Avalon Scaler — clock domains, CDC, data flow, interpolation algorithms, triple buffering, and all 17 functional blocks |
| [Analog & Direct Video](analog_direct_video_architecture.md) | **Deep** | Direct Video (HDMI bypass), RGB→YPbPr (`vga_out.sv`), NTSC/PAL composite (`yc_out.sv`), sync orchestration, IO board DAC |
| [Polyphase Scaling Theory](polyphase_scaling_theory.md) | Deep | Mathematical foundations of video scaling, FIR filters, and coefficient vectors |
| [ascal Architecture](ascal_architecture.md) | Adequate | Overview of the VHDL and SystemVerilog implementations |
| [Video Audio Pipelines](../06_fpga_subsystem/video_audio_pipelines.md) | Adequate | How A/V signals flow through the FPGA framework |

## 2. Audio Processing

| Document | Quality | Description |
|----------|---------|-------------|
| [Audio Pipeline Deep Dive](audio_pipeline_deep_dive.md) | **Deep** | Source-grounded analysis of `audio_out.v`: clock-enable generation, input synchronizer, IIR filter, DC blocker, `aud_mix_top` mixer (crossfeed + attenuation + clamp), I2S/SPDIF/ΣΔ DAC, startup muting |

## 3. Reference Implementations

| File | Lines | Description |
|------|-------|-------------|
| `ascal.sv` | 2414 | SystemVerilog port of the polyphase scaler |
| `ascal.vhd` | 3057 | Original MiSTer VHDL polyphase scaler (TEMLIB) |

## 4. Cross-References

- [FPGA KB — Intel HPS-FPGA Integration](https://github.com/alfishe/fpga-knowledge-base) — Cyclone V SoC architecture
- [MiSTer Wiki — Video Modes](https://github.com/MiSTer-devel/Main_MiSTer/wiki/Video-modes) — User-facing configuration
