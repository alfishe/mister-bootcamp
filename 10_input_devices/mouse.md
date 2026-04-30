[← Input Devices](../README.md)

# Mouse Path: Linux evdev → PS/2 Bytes → FPGA

MiSTer reads USB mouse events from Linux and encodes them as PS/2 mouse
packets delivered to the FPGA core.

Sources: `Main_MiSTer/input.cpp`, `user_io.cpp`, `hps_io.sv`

---

## End-to-End Flow

```mermaid
flowchart TD
    M[USB / BT Mouse] -->|HID report| USB[Linux USB HID driver]
    USB -->|/dev/input/eventX| evdev[evdev EV_REL + EV_KEY]
    evdev -->|read()| input_poll["input_poll()\ninput.cpp"]
    input_poll -->|buttons, dx, dy, dw| user_io_mouse["user_io_mouse()\nuser_io.cpp"]
    user_io_mouse -->|UIO_MOUSE cmd 0x04| spi["SPI-over-GPO"]
    spi -->|HPS_BUS| hps_io["hps_io.sv\n'h04 handler"]
    hps_io -->|"ps2_mouse[24:0]"| core["Core RTL"]
    hps_io -.->|PS2DIV path| ps2_dev["ps2_device\nserial PS/2 output"]
    ps2_dev -.->|ps2_mouse_clk/data| core
```

---

## Linux evdev Events

```c
// input.cpp (simplified)
// Relative axis events accumulate between polls:
if (ev.type == EV_REL) {
    if (ev.code == REL_X) mouse_dx += ev.value;
    if (ev.code == REL_Y) mouse_dy += ev.value;
    if (ev.code == REL_WHEEL) mouse_dw += ev.value;
}
if (ev.type == EV_KEY) {
    if (ev.code == BTN_LEFT)   buttons = set/clear bit 0;
    if (ev.code == BTN_RIGHT)  buttons = set/clear bit 1;
    if (ev.code == BTN_MIDDLE) buttons = set/clear bit 2;
}
// On each poll cycle, call:
user_io_mouse(buttons, mouse_dx, mouse_dy, mouse_dw);
```

---

## UIO Mouse Command (Opcode `0x04`)

```c
// user_io.cpp
void user_io_mouse(unsigned char b, int16_t x, int16_t y, int16_t w)
{
    // Clamp deltas to fit PS/2 packet
    if (x >  127) x =  127;
    if (x < -128) x = -128;
    if (y >  127) y =  127;
    if (y < -128) y = -128;

    spi_uio_cmd_cont(UIO_MOUSE);  // 0x04, assert IO CS
    spi_w(b & 0x07);              // byte 0: buttons [2:0]
    spi_w((uint8_t)x);            // byte 1: X movement
    spi_w((uint8_t)(-y));         // byte 2: Y movement (PS/2 Y axis inverted)
    if (w) spi_w((uint8_t)w);    // byte 3: wheel (optional)
    spi_w(0xFF);                  // end marker
    DisableIO();
}
```

---

## FPGA Reception (`hps_io.sv`)

Each word sent is 16-bit where `[7:0]` = PS/2 byte, `[15:8]` = end marker:

```verilog
// hps_io.sv — command 0x04
'h04: begin
    if(PS2DIV) begin
        mouse_data <= io_din[7:0];
        mouse_we   <= 1;               // feed serial PS/2 device
    end
    if(&io_din[15:8]) ps2skip <= 1;   // 0xFF = end of packet
    if(~&io_din[15:8] && ~ps2skip && !byte_cnt[MAX_W:2]) begin
        case(byte_cnt[1:0])
            1: ps2_mouse[7:0]   <= io_din[7:0];  // buttons + overflow bits
            2: ps2_mouse[15:8]  <= io_din[7:0];  // X delta
            3: ps2_mouse[23:16] <= io_din[7:0];  // Y delta
        endcase
        // Extended ext data (wheel, extra buttons):
        case(byte_cnt[1:0])
            1: ps2_mouse_ext[7:0]   <= {io_din[14], io_din[14:8]};
            2: ps2_mouse_ext[11:8]  <= io_din[11:8];
            3: ps2_mouse_ext[15:12] <= io_din[11:8];
        endcase
    end
end

// At end of transaction:
if(cmd == 4 && !ps2skip) ps2_mouse[24] <= ~ps2_mouse[24]; // toggle
```

### `ps2_mouse[24:0]` Output Format

| Bits | Meaning |
|---|---|
| `[24]` | Toggle bit — flips with every mouse event |
| `[23:16]` | Y delta (signed, PS/2 convention) |
| `[15:8]` | X delta (signed) |
| `[7]` | Y overflow |
| `[6]` | X overflow |
| `[5]` | Y sign bit |
| `[4]` | X sign bit |
| `[3]` | Always 1 (PS/2 format) |
| `[2]` | Middle button |
| `[1]` | Right button |
| `[0]` | Left button |

### `ps2_mouse_ext[15:0]`

| Bits | Meaning |
|---|---|
| `[15:12]` | Reserved (extra buttons) |
| `[11:8]` | Wheel movements (nibble 2) |
| `[7:0]` | Wheel movements (nibble 1) + button 4/5 |

---

## Keyboard-as-Mouse Emulation

MiSTer supports a keyboard emulation mode where cursor keys control the mouse:

```c
// user_io.cpp
static int emu_mode = EMU_NONE;  // EMU_MOUSE, EMU_JOY0, EMU_JOY1

void set_emu_mode(int mode) {
    emu_mode = mode;
    // NumLock + ScrollLock LEDs indicate active mode
    spi_uio_cmd16(UIO_LEDS, 0x6000 | emu_led);
}
```

When `emu_mode == EMU_MOUSE`, cursor key events are translated to
`user_io_mouse()` calls with unit deltas.

---

## Amiga / Minimig Mouse Handling

The Minimig core receives mouse data via `hps_ext.v` on the `EXT_BUS`:

```verilog
// hps_ext.v — UIO_MOUSE (0x04)
UIO_MOUSE:
    case(byte_cnt)
        1: begin
            kbd_mouse_data  <= io_din[7:0];  // byte 0: buttons
            kbd_mouse_type  <= 0;            // type=0: button data
            kbd_mouse_level <= ~kbd_mouse_level;
        end
        2: begin
            kbd_mouse_data  <= io_din[7:0];  // byte 1: X movement
            kbd_mouse_type  <= 1;            // type=1: X movement
            kbd_mouse_level <= ~kbd_mouse_level;
        end
        3: begin
            mouse_buttons <= io_din[2:0];    // latch button state
        end
        4: begin
            kbd_mouse_data  <= io_din[7:0];  // byte 3: wheel
            kbd_mouse_level <= ~kbd_mouse_level;
        end
    endcase
```

The Minimig core's internal mouse hardware reads `kbd_mouse_type` to route
data to the X/Y counters or button register, and `kbd_mouse_level` as an
edge-detect trigger.
