[← Input Devices](README.md) · [↑ Knowledge Base](../README.md)

# Keyboard Path: Linux evdev → PS/2 Emulation → FPGA

MiSTer translates physical USB or Bluetooth keyboard events all the way from
Linux HID to PS/2 scan codes delivered to the FPGA core.

Sources: `Main_MiSTer/input.cpp`, `input.h`, `user_io.cpp`, `hps_io.sv`

---

## End-to-End Flow

```mermaid
flowchart TD
    K[USB / BT Keyboard] -->|HID report| USB[Linux USB HID driver]
    USB -->|/dev/input/eventX| evdev[evdev layer]
    evdev -->|read() EV_KEY events| input_poll["input_poll()\ninput.cpp"]
    input_poll -->|Linux keycodes| ps2_xlat["PS/2 translation\nget_ps2_code()"]
    ps2_xlat -->|PS/2 scan bytes| user_io_kbd["user_io_kbd()\nuser_io.cpp"]
    user_io_kbd -->|"UIO_KEYBOARD 0x05"| spi["SPI-over-GPO"]
    spi -->|HPS_BUS| hps_io["hps_io.sv — h05 handler"]
    hps_io -->|"ps2_key[10:0]"| core["Core RTL — emu module"]
    hps_io -.->|PS2DIV path| ps2_dev["ps2_device\nserial PS/2 output"]
    ps2_dev -.->|ps2_kbd_clk/data| core
```

---

## Linux evdev Layer (`input.cpp`)

`input_poll()` opens all `/dev/input/event*` devices and uses `select()` to
multiplex them.  For keyboard events:

```c
// input.cpp (simplified)
struct input_event ev;
read(fd, &ev, sizeof(ev));

if (ev.type == EV_KEY) {
    int key  = ev.code;   // Linux keycode (KEY_A, KEY_SPACE, etc.)
    int press = ev.value; // 1=press, 0=release, 2=autorepeat

    // Pass to user_io layer
    user_io_kbd(key, press);
}
```

Modifier keys (Shift, Ctrl, Alt, GUI) are tracked separately in a bitmask
and combined into the high bits of the UIO key word.

---

## PS/2 Scan Code Translation (`get_ps2_code`)

```c
// input.cpp
uint32_t get_ps2_code(uint16_t key)
{
    // Returns: [7:0] = scan code byte, [8] = extended (0xE0 prefix)
    // Uses a static lookup table indexed by Linux keycode
}
```

The scan set used is **Set 2** (standard PC PS/2).  Special cases:
- **Extended keys** (arrows, Insert, Home, etc.) → prefix `0xE0` before the code.
- **Print Screen press**: `E0 12 E0 7C`
- **Print Screen release**: `E0 F0 7C E0 F0 12`
- **Pause/Break**: `E1 14 77 E1 F0 14 F0 77` (no release code)

---

## UIO Keyboard Command (Opcode `0x05`)

```c
// user_io.cpp
void user_io_kbd(uint16_t key, int press)
{
    static uint8_t modifier = 0;

    // Build PS/2 byte sequence
    uint8_t ps2[4];
    int len = build_ps2_sequence(key, press, ps2);

    spi_uio_cmd_cont(UIO_KEYBOARD);  // 0x05, assert IO CS
    for (int i = 0; i < len; i++)
        spi_w(ps2[i]);               // send each byte
    DisableIO();                     // trigger FPGA commit
}
```

Wire format sent to FPGA (each word is 16-bit):
- `[7:0]` = PS/2 scan code byte
- `[15:8]` = `0xFF` if this is the last byte (signals end-of-sequence)

---

## FPGA Reception (`hps_io.sv`)

```verilog
// hps_io.sv — command 0x05
'h05: begin
    if(&io_din[15:8]) ps2skip <= 1;                // 0xFF = end marker
    if(~&io_din[15:8] & ~ps2skip)
        ps2_key_raw[31:0] <= {ps2_key_raw[23:0], io_din[7:0]}; // shift in byte
    if(PS2DIV) begin
        kbd_data <= io_din[7:0];  // also feed serial PS/2 device
        kbd_we   <= 1;
    end
end

// At end of transaction (io_enable deasserted):
if(cmd == 5 && !ps2skip) begin
    ps2_key <= {~ps2_key[10],   // [10]: toggles every press/release
                pressed,         // [9]:  1=press, 0=release
                extended,        // [8]:  1=extended key
                ps2_key_raw[7:0]}; // [7:0]: scan code
    // Special sequences decoded inline:
    if(ps2_key_raw == 'hE012E07C) ps2_key[9:0] <= 'h37C; // prnscr press
    if(ps2_key_raw == 'h7CE0F012) ps2_key[9:0] <= 'h17C; // prnscr release
    if(ps2_key_raw == 'hF014F077) ps2_key[9:0] <= 'h377; // pause press
end
```

### `ps2_key[10:0]` Output Format

| Bits | Meaning |
|---|---|
| `[10]` | Toggle bit — flips on every press or release event |
| `[9]` | 1 = key pressed, 0 = key released |
| `[8]` | 1 = extended key (was prefixed by 0xE0) |
| `[7:0]` | PS/2 Set-2 scan code |

Cores should watch `ps2_key[10]` for changes (edge detect) rather than
polling the other bits, since the same key can be pressed repeatedly.

---

## Optional: Serial PS/2 Output (`PS2DIV` mode)

When the `hps_io` module parameter `PS2DIV > 0`, a serial PS/2 device is
instantiated that drives real `ps2_kbd_clk_out` / `ps2_kbd_data_out` pins:

```verilog
// hps_io.sv (PS2DIV section)
ps2_device keyboard (
    .clk_sys(clk_sys),
    .wdata(kbd_data),
    .we(kbd_we),
    .ps2_clk(clk_ps2),           // divided clock ~10-20 kHz
    .ps2_clk_out(ps2_kbd_clk_out),
    .ps2_dat_out(ps2_kbd_data_out),
    .ps2_clk_in(ps2_kbd_clk_in || !PS2WE),
    .ps2_dat_in(ps2_kbd_data_in || !PS2WE),
    .rdata(kbd_data_host),
    .rd(kbd_rd)
);
```

`ps2_device` is a FIFO-backed bidirectional PS/2 controller that
implements the open-collector PS/2 protocol in RTL.

---

## Keyboard LED Feedback

Cores can request specific LED states using `ps2_kbd_led_use` / `ps2_kbd_led_status`:

```verilog
// hps_io.sv — opcode 0x1F
'h1f: io_dout <= {|PS2WE, 2'b01,
                  ps2_kbd_led_status[2], ps2_kbd_led_use[2],   // ScrollLock
                  ps2_kbd_led_status[1], ps2_kbd_led_use[1],   // NumLock
                  ps2_kbd_led_status[0], ps2_kbd_led_use[0]};  // CapsLock
```

MiSTer polls this at 100 ms intervals and calls `set_kbdled()` which
issues a `ioctl(EVIOCSLED)` to the evdev device.

---

## Amiga / Minimig Special Handling

The Minimig core uses `hps_ext.v` which intercepts `UIO_KEYBOARD` (0x05)
on the `EXT_BUS` and drives an **Amiga-specific keyboard protocol**:

```verilog
// hps_ext.v
UIO_KEYBOARD:
    if(byte_cnt == 1) begin
        kbd_mouse_data  <= io_din[7:0];  // Amiga key code
        kbd_mouse_type  <= 2;            // type=2: keyboard
        kbd_mouse_level <= ~kbd_mouse_level; // toggle = new event
    end
```

The Amiga keyboard hardware receives the Amiga raw key codes directly —
no PS/2 emulation layer is involved on that path.

---

## Cross-References

| Topic | Article |
|---|---|
| Mouse input path | [Mouse](mouse.md) |
| Joystick input path | [Joystick](joystick.md) |
| SNAC direct wiring | [SNAC & LLAPI](snac_llapi.md) |
| UIO keyboard opcodes | [UIO Command Reference](../17_references/uio_command_reference.md) |
| hps_io keyboard decoder | [hps_io Module](../06_fpga_subsystem/hps_io_module.md) |
