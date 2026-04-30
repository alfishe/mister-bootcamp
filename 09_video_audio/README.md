[← Knowledge Base](../README.md)

# Video & Audio

The complete video and audio pipeline from core output through to HDMI/VGA/composite connectors. Covers the HDMI polyphase scaler (`ascal.vhd`), analog video output via IO board, and the audio mixing/filtering chain.

## Articles

| File | Topic | Quality |
|---|---|---|
| [hdmi_scaler.md](hdmi_scaler.md) | `ascal.vhd`: polyphase scaler, DDR3 framebuffer, filter coefficient loading | 📋 Target: Deep |
| [audio_pipeline.md](audio_pipeline.md) | Sigma-delta DAC, I2S, HDMI audio embedding, filters, 96 kHz mode | 📋 Target: Deep |
| [analog_video.md](analog_video.md) | IO board analog output, direct video, composite/S-Video, CRT considerations | 📋 Target: Adequate |

## Cross-References

- [Configuration — MiSTer.ini](../05_configuration/mister_ini.md) — video mode, scaler, filter settings
- [Framework — sys_top.v](../06_fpga_subsystem/sys_top.md) — PLL tree, HDMI/VGA muxing
