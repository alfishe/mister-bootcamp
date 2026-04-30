[← Framework](../README.md)

# GPO / GPI Registers — Raw HPS↔FPGA Signalling

The lowest-level bridge between the ARM HPS and the FPGA logic is a pair
of 32-bit General-Purpose I/O registers exposed through the **FPGA Manager**
peripheral at physical address `0xFF706000`.

Source: `Main_MiSTer/fpga_io.cpp`, `fpga_base_addr_ac5.h`

---

## Physical Addresses

| Register | Address | Access |
|---|---|---|
| GPO (output from HPS) | `0xFF706010` | Write by HPS → read by FPGA |
| GPI (input to HPS) | `0xFF706014` | Write by FPGA → read by HPS |

The MiSTer binary accesses these via `/dev/mem` with `mmap`:

```c
// fpga_io.cpp
#define FPGA_REG_BASE 0xFF000000
#define FPGA_REG_SIZE 0x01000000

map_base = (uint32_t*)shmem_map(FPGA_REG_BASE, FPGA_REG_SIZE);

#define fpga_gpo_write(value) writel((value), (void*)(SOCFPGA_MGR_ADDRESS + 0x10))
#define fpga_gpi_read()       (int)readl((void*)(SOCFPGA_MGR_ADDRESS + 0x14))
```

---

## GPO Bit Layout (HPS → FPGA)

| Bits | Name | Description |
|---|---|---|
| `[15:0]` | `io_din` | 16-bit data word to send to FPGA |
| `[16]` | (reserved) | — |
| `[17]` | `SSPI_STROBE` | Toggling this bit clocks one 16-bit word into FPGA |
| `[18]` | `io_ss0` | Chip-select 0 |
| `[19]` | `io_ss1` | Chip-select 1 |
| `[20]` | `io_ss2` | Chip-select 2 |
| `[28:21]` | (unused) | — |
| `[29]` | `led_disk` | Controls disk activity LED from HPS side |
| `[30]` | `RESET` | Assert core reset when set |
| `[31]` | `ID_MODE` | When set, FPGA returns core magic/ID on GPI instead of data |

```c
// fpga_io.cpp — key bit constants
#define SSPI_STROBE  (1<<17)

void fpga_core_reset(int reset)
{
    uint32_t gpo = fpga_gpo_read() & ~0xC0000000;
    // bit 30 = reset, bit 31 = ID query mode
    fpga_gpo_write(reset ? gpo | 0x40000000 : gpo | 0x80000000);
}
```

---

## GPI Bit Layout (FPGA → HPS)

| Bits | Name | Description |
|---|---|---|
| `[15:0]` | `io_dout` | 16-bit response word from FPGA |
| `[16]` | `io_wide` | 1 = WIDE (16-bit) file I/O mode |
| `[17]` | `io_ack` | Strobe acknowledgement (mirrors STROBE back) |
| `[18]` | (reserved) | — |
| `[19:18]` | `io_ver` | IO protocol version (current = 1) |
| `[27:20]` | (unused) | — |
| `[28]` | `io_type` | Board I/O type |
| `[29]` | (unused) | — |
| `[30:29]` | `btn_state` | Physical button states |
| `[31]` | `NOT_READY` | 1 = FPGA not in user mode (uninitialized) |

```verilog
// sys_top.v
wire [31:0] gp_in = {1'b0,           // [31] = 0 means FPGA is ready
                     btn_user|btn[1], // [30]
                     btn_osd|btn[0],  // [29]
                     io_dig,          // [28]
                     8'd0,            // [27:20]
                     io_ver,          // [19:18]
                     io_ack,          // [17]
                     io_wide,         // [16]
                     io_dout | io_dout_sys}; // [15:0]
```

---

## Core Identification Protocol

Before any commands are sent, MiSTer verifies the FPGA is loaded with a valid
core by reading the **core magic number**:

```c
// fpga_io.cpp
int fpga_core_id()
{
    uint32_t gpo = (fpga_gpo_read() & 0x7FFFFFFF);
    fpga_gpo_write(gpo);                    // bit 31 = 0: data mode
    uint32_t coretype = fpga_gpi_read();    // read whatever is on GPI
    gpo |= 0x80000000;
    fpga_gpo_write(gpo);                    // bit 31 = 1: ID mode
    // In ID mode, FPGA drives core_magic on GPI:
    //   gp_in = ~gp_out[31] ? core_magic : gp_in
    // sys_top.v: wire [31:0] core_magic = {24'h5CA623, core_type};
    if ((coretype >> 8) != 0x5CA623) return -1;
    return coretype & 0xFF;   // 0xA4 = generic, 0xA8 = dual-SDRAM
}
```

```verilog
// sys_top.v
wire [31:0] core_magic = {24'h5CA623, core_type};
cyclonev_hps_interface_mpu_general_purpose h2f_gp
(
    .gp_in({~gp_out[31] ? core_magic : gp_in}),
    .gp_out(gp_out)
);
```

---

## LW-AXI Slave Window

In addition to the GPO/GPI interface, there is a **Light-Weight AXI** slave
window at `0xFF200000` that maps directly into the FPGA fabric:

```c
// fpga_io.cpp
#define SOCFPGA_LWFPGASLAVES_ADDRESS 0xff200000

void fpga_core_write(uint32_t offset, uint32_t value)
{
    if (offset <= 0x1FFFFF)
        writel(value, (void*)(SOCFPGA_LWFPGASLAVES_ADDRESS + (offset & ~3)));
}

uint32_t fpga_core_read(uint32_t offset)
{
    if (offset <= 0x1FFFFF)
        return readl((void*)(SOCFPGA_LWFPGASLAVES_ADDRESS + (offset & ~3)));
    return 0;
}
```

This window is used for IDE register access (ao486, x86 cores) where
memory-mapped registers inside the FPGA need to be polled or written
at high speed without the SSPI protocol overhead.

---

## FPGA Ready Detection

```c
// fpga_io.cpp
int is_fpga_ready(int quick)
{
    if (quick)
        return (fpga_gpi_read() >= 0);  // GPI[31] = 0 means ready
    return fpgamgr_test_fpga_ready();   // check FPGA Manager stat register
}
```

GPI bit 31 is hardwired to `1'b0` inside `sys_top.v` after the FPGA is
in user mode.  Before loading (or during JTAG upload), the FPGA is in
configuration mode and the HPS reads an all-ones value on GPI, making
`fpga_gpi_read()` return a negative integer.
