[← References](../README.md)

# UIO Command Reference

All HPS→FPGA communication for peripheral events uses a command-based protocol
over the SPI-over-GPO bus.  Each transaction starts with a **command opcode**
(one 16-bit word) followed by zero or more data words.

The FPGA-side handler is `hps_io.sv` (`uio_block` always block).
The HPS-side sender is `user_io.cpp` / `spi.cpp`.

Source: `Main_MiSTer/user_io.h`, `hps_io.sv`

---

## Opcode Table

### Input Events

| Opcode | Name | HPS→FPGA | FPGA→HPS | Description |
|---|---|---|---|---|
| `0x01` | `UIO_BUT_SW` | cfg[15:0] | — | System config flags (scandoubler, YPbPr, etc.) |
| `0x02` | `UIO_JOYSTICK0` | joy[15:0], joy[31:16] | — | Joystick 0 buttons (32-bit, 2 words) |
| `0x03` | `UIO_JOYSTICK1` | joy[15:0], joy[31:16] | — | Joystick 1 buttons |
| `0x04` | `UIO_MOUSE` | byte×3 | — | PS/2 mouse bytes (buttons, X, Y) |
| `0x05` | `UIO_KEYBOARD` | byte×N | — | PS/2 keyboard scan codes |
| `0x06` | `UIO_KBD_OSD` | byte | — | OSD-only key codes |
| `0x10` | `UIO_JOYSTICK2` | joy×2 words | — | Joystick 2 |
| `0x11` | `UIO_JOYSTICK3` | joy×2 words | — | Joystick 3 |
| `0x12` | `UIO_JOYSTICK4` | joy×2 words | — | Joystick 4 |
| `0x13` | `UIO_JOYSTICK5` | joy×2 words | — | Joystick 5 |
| `0x1a` | `UIO_ASTICK` | idx, xy[15:0] | — | Left analog stick (index + 16-bit XY) |
| `0x3D` | `UIO_ASTICK_2` | idx, xy[15:0] | — | Right analog stick |
| `0x3F` | `UIO_GET_RUMBLE` | joy_idx[15:8] | rumble[15:0] | Rumble motor magnitudes |

### SD Card / Disk Emulation

| Opcode | Name | HPS→FPGA | FPGA→HPS | Description |
|---|---|---|---|---|
| `0x16` | `UIO_GET_SDSTAT` | — | {1,blk_cnt,BLKSZ,sdn,wr,rd} | Poll pending SD request |
| `0x17` | `UIO_SECTOR_RD` | data words | — | Write sector data to FPGA buffer |
| `0x18` | `UIO_SECTOR_WR` | — | data words | Read sector data from FPGA buffer |
| `0x1C` | `UIO_SET_SDSTAT` | img_mounted, readonly | — | Notify image mount status |
| `0x1D` | `UIO_SET_SDINFO` | size×4 words | — | Notify image size (64-bit bytes) |
| `0x19` | `UIO_SET_SDCONF` | — | — | Send SD card CSD/CID config |

### Configuration / Status

| Opcode | Name | HPS→FPGA | FPGA→HPS | Description |
|---|---|---|---|---|
| `0x14` | `UIO_GET_STRING` | — | char bytes | Read CONF_STR from core |
| `0x15` | `UIO_SET_STATUS` | status byte | — | Set 8-bit status word (legacy) |
| `0x1E` | `UIO_SET_STATUS2` | status×8 words | — | Set 128-bit status word |
| `0x29` | `UIO_GET_STATUS` | — | status×8 words | Read back 128-bit status from core |
| `0x2E` | `UIO_GET_OSDMASK` | — | mask[15:0] | Read OSD menu mask from core |
| `0x1F` | `UIO_GET_KBD_LED` | — | led_status | Keyboard LED control state |
| `0x31` | `UIO_SET_MEMSZ` | sdram_sz | — | Inform core of SDRAM size |

### Video / Scaler

| Opcode | Name | Direction | Description |
|---|---|---|---|
| `0x20` | `UIO_SET_VIDEO` | HPS→FPGA | Send full HDMI timing (WIDTH/HFP/HS/HBP/HEIGHT/VFP/VS/VBP) |
| `0x23` | `UIO_GET_VRES` | FPGA→HPS | Video resolution parameters |
| `0x2B` | `UIO_SET_FLTNUM` | HPS→FPGA | Predefined scaler filter index |
| `0x2A` | `UIO_SET_FLTCOEF` | HPS→FPGA | Polyphase scaler coefficients |
| `0x2C` | `UIO_GET_VMODE` | FPGA→HPS | Core video mode (for Minimig hps_ext) |
| `0x2D` | `UIO_SET_VPOS` | HPS→FPGA | Video position (for Minimig overscan) |
| `0x37` | `UIO_SETWIDTH` | HPS→FPGA | Max scaled horizontal resolution |
| `0x27` | `UIO_SETHEIGHT` | HPS→FPGA | Max scaled vertical resolution |
| `0x30` | `UIO_WAIT_VSYNC` | HPS→FPGA | Block until next VSync |
| `0x40` | `UIO_GET_FB_PAR` | FPGA→HPS | Framebuffer parameters |
| `0x41` | `UIO_SET_YC_PAR` | HPS→FPGA | Y/C (composite) subcarrier config |
| `0x3E` | `UIO_SHADOWMASK` | HPS→FPGA | Shadow mask data |
| `0x42` | `UIO_GET_FR_CNT` | FPGA→HPS | Frame counter (VSync count) |

### Audio

| Opcode | Name | Description |
|---|---|---|
| `0x26` | `UIO_AUDVOL` | Digital volume attenuation (shift right N bits) |
| `0x39` | `UIO_SET_AFILTER` | IIR audio filter coefficients |
| `0x38` | `UIO_SETSYNC` | Audio sync config |

### System / RTC

| Opcode | Name | Description |
|---|---|---|
| `0x22` | `UIO_RTC` | MSM6242B-format RTC data (8 words) |
| `0x24` | `UIO_TIMESTAMP` | Unix timestamp (2 words) |
| `0x25` | `UIO_LEDS` | Control on-board LEDs |
| `0x28` | `UIO_GETUARTFLG` | UART flags |
| `0x3B` | `UIO_SET_UART` | UART mode + baud rate |
| `0x32` | `UIO_SET_GAMMA` | Enable/disable gamma correction |
| `0x33` | `UIO_SET_GAMCURV` | Gamma curve data |
| `0x2F` | `UIO_SET_FBUF` | Framebuffer parameters for HPS output |
| `0x36` | `UIO_INFO_GET` | Core info display request |
| `0x3A` | `UIO_SET_AR_CUST` | Custom aspect ratio |
| `0x3C` | `UIO_CHK_UPLOAD` | Check for upload request |
| `0x43` | `UIO_GET_F12_MOD` | F12 key modifier mode |

### File I/O (File Port — `fp_enable`)

| Opcode | Name | Description |
|---|---|---|
| `0x53` | `FIO_FILE_TX` | Start/end file transfer (download or upload) |
| `0x54` | `FIO_FILE_TX_DAT` | Transfer one data word |
| `0x55` | `FIO_FILE_INDEX` | Set file/slot index for next transfer |
| `0x56` | `FIO_FILE_INFO` | Set file extension info |

### DMA (ao486 / x86 direct memory access)

| Opcode | Name | Description |
|---|---|---|
| `0x61` | `UIO_DMA_WRITE` | Write data to FPGA via DMA path |
| `0x62` | `UIO_DMA_READ` | Read data from FPGA via DMA path |
| `0x63` | `UIO_DMA_SDIO` | Poll IDE/CD request flags |

---

## `UIO_BUT_SW` Config Bits (0x01)

The first command sent after core load configures global hardware behaviour:

| Bit | Constant | Meaning |
|---|---|---|
| 0 | `BUTTON1` | Physical button 1 state |
| 1 | `BUTTON2` | Physical button 2 state |
| 2 | `CONF_VGA_SCALER` | Enable VGA scaler |
| 3 | `CONF_CSYNC` | Composite sync on HSync |
| 4 | `CONF_FORCED_SCANDOUBLER` | Force scan-doubler on VGA |
| 5 | `CONF_YPBPR` | YPbPr output mode |
| 6 | `CONF_AUDIO_96K` | 96 kHz audio mode |
| 7 | `CONF_DVI` | DVI (no audio) output |
| 8 | `CONF_HDMI_LIMITED1` | HDMI limited range 1 |
| 9 | `CONF_VGA_SOG` | Sync-on-Green |
| 10 | `CONF_DIRECT_VIDEO` | Direct video (bypass scaler) |
| 11 | `CONF_HDMI_LIMITED2` | HDMI limited range 2 |
| 12 | `CONF_VGA_FB` | VGA framebuffer mode |

---

## `UIO_GET_STRING` / CONF_STR Format

The core's OSD configuration string is read byte-by-byte using opcode `0x14`.
The FPGA returns `conf_byte` on each strobe until the null terminator.

```verilog
// hps_io.sv
'h14: if(byte_cnt <= STRLEN) io_dout[7:0] <= conf_byte;
```

String format (semicolon-delimited entries):
```
"CoreName;;"               ← entry 0: core name
"UART,speed1,speed2,...;"  ← entry 1: hardware capabilities
"O[bit],Label,Off,On;"    ← entry N: OSD option
"S0,EXT;"                  ← SD card slot definition
"T[0],Reset;"              ← trigger (momentary)
"V,vDATE"                  ← version string
```
