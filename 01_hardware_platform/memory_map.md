[← Hardware Platform Index](README.md) · [↑ Knowledge Base](../README.md)

# Cyclone V SoC Physical Memory Map

Key physical addresses used by MiSTer, derived from `fpga_base_addr_ac5.h`
and the Cyclone V HPS Technical Reference Manual.

Source: [`fpga_base_addr_ac5.h`](https://github.com/MiSTer-devel/Main_MiSTer/blob/master/fpga_base_addr_ac5.h), [`fpga_io.cpp`](https://github.com/MiSTer-devel/Main_MiSTer/blob/master/fpga_io.cpp), [`shmem.cpp`](https://github.com/MiSTer-devel/Main_MiSTer/blob/master/shmem.cpp)

> [!NOTE]
> For the GPO/GPI strobe protocol that uses these registers, see
> [SPI-over-GPO Protocol](../02_framework/spi_protocol.md). For the Cyclone V
> SoC architecture, see [FPGA KB — Intel Cyclone V](https://github.com/alfishe/fpga-bootcamp).

---

## Memory Map Table

| Address Range | Name | Used by MiSTer |
|---|---|---|
| `0x00000000–0x3FFFFFFF` | DDR3 SDRAM (1 GB) | Linux, core DDR3, HDMI scaler buffer |
| `0x20000000–0x3FFFFFFF` | DDR3 (upper half) | FPGA-facing portion (F2H AXI) |
| `0x20000000` | HDMI scaler VBUF base | `ascal.vhd` frame buffer (`RAMBASE`) |
| `0x1FFFF000` | OCRAM window | Reboot flags, core environment block |
| `0xFF000000–0xFFFFFFFF` | HPS Peripheral Space | All peripherals below |
| `0xFF200000` | LW-HPS-to-FPGA (2 MB) | `fpga_core_read/write()` — IDE regs, etc. |
| `0xFF400000` | LW-HPS-to-FPGA regs | — |
| `0xFF500000` | HPS-to-FPGA regs | — |
| `0xFF600000` | FPGA-to-HPS regs | — |
| `0xFF700000` | EMAC0 | — |
| `0xFF702000` | EMAC1 | — |
| `0xFF704000` | SDMMC | — |
| `0xFF706000` | FPGA Manager Registers | Core loading, GPO/GPI |
| `0xFF706010` | **GPO Register** | SSPI data + strobe + CS |
| `0xFF706014` | **GPI Register** | FPGA response + status |
| `0xFF707000` | ACP ID Map | — |
| `0xFF708000–0xFF70A000` | GPIO 0/1/2 | — |
| `0xFF800000` | NIC-301 (L3 Switch) | Bridge remap control |
| `0xFFB00000` | USB0 | USB host controller |
| `0xFFB40000` | USB1 | USB host controller |
| `0xFFB80000` | NAND regs | — |
| `0xFFB90000` | **FPGA Manager Data** | RBF bitstream write target |
| `0xFFC00000` | CAN0 | — |
| `0xFFC02000` | UART0 | — |
| `0xFFC04000–0xFFC07000` | I2C 0–3 | — |
| `0xFFC08000` | SP Timer 0 | — |
| `0xFFC20000` | SDRAM Controller | DDR3 port configuration |
| `0xFFD00000` | OSC1 Timer 0 | — |
| `0xFFD04000` | Clock Manager | PLL configuration |
| `0xFFD05000` | Reset Manager | Warm/cold reset control |
| `0xFFD08000` | System Manager | Bridge group enable |
| `0xFFE00000` | DMA non-secure | — |
| `0xFFEC0000` | MPU SCU | ARM Cortex-A9 SCU |
| `0xFFEF0000` | L2 Cache | — |
| `0xFFFF0000` | **On-Chip RAM** (OCRAM, 64 KB) | Reboot flags, fast scratch |

---

## Key Regions Used by MiSTer

### GPO / GPI Registers

```c
// fpga_io.cpp
#define SOCFPGA_MGR_ADDRESS 0xff706000

static inline void fpga_gpo_write(uint32_t value) {
    *MAP_ADDR(SOCFPGA_MGR_ADDRESS + 0x10) = value;  // 0xFF706010
}
static inline int fpga_gpi_read() {
    return (int)*MAP_ADDR(SOCFPGA_MGR_ADDRESS + 0x14); // 0xFF706014
}
```

### LW-AXI Slave Window (FPGA registers)

```c
// fpga_io.cpp
#define SOCFPGA_LWFPGASLAVES_ADDRESS 0xff200000

void fpga_core_write(uint32_t offset, uint32_t value) {
    // Maps directly into FPGA fabric (core-defined registers)
    *MAP_ADDR(SOCFPGA_LWFPGASLAVES_ADDRESS + offset) = value;
}
```

Used by:
- IDE controllers (ao486, x86 cores): register file at core-defined offsets
- Direct-video framebuffer configuration

### FPGA Manager Data Port

```c
// fpga_io.cpp — bitstream write target
#define SOCFPGA_FPGAMGRDATA_ADDRESS 0xffb90000

uint32_t dst = (uint32_t)MAP_ADDR(SOCFPGA_FPGAMGRDATA_ADDRESS);
// Written with ARM ldmia/stmia assembly burst
```

See [FPGA Loading](../02_framework/fpga_loading.md) for the full bitstream loading sequence.

### OCRAM (On-Chip RAM)

```c
// shmem.cpp / fpga_io.cpp
void* buf = shmem_map(0x1FFFF000, 0x1000);
// Used to pass: reboot type flag, core name, CFG content
// Survives warm reset; read by MiSTer on next boot
```

> [!NOTE]
> OCRAM is the mechanism for passing state across warm reboots (core switching).
> When a user selects a new core, MiSTer writes the core filename and reboot type
> to OCRAM, then triggers a warm reset. On next boot, U-Boot reads the OCRAM flags
> and loads the requested core. See [U-Boot](../10_linux_system/uboot/README.md).

### DDR3 HDMI Buffer

```verilog
// sys_top.v → ascal.vhd
parameter RAMBASE = 32'h20000000;  // 512 MB boundary
parameter RAMSIZE = 32'h00800000;  // 8 MB HDMI frame buffer
```

See [HDMI Scaler](../06_video_audio/hdmi_scaler.md) for the `ascal.vhd` framebuffer architecture.

---

## mmap Access Pattern

All access is via a single `mmap()` of the entire 16 MB HPS peripheral space:

```c
// shmem.cpp / fpga_io.cpp
int fd = open("/dev/mem", O_RDWR | O_SYNC);
map_base = mmap(0, FPGA_REG_SIZE, PROT_READ|PROT_WRITE,
                MAP_SHARED, fd, FPGA_REG_BASE);
// FPGA_REG_BASE = 0xFF000000
// FPGA_REG_SIZE = 0x01000000 (16 MB)

// Access: index by (addr & 0xFFFFFF) >> 2
#define MAP_ADDR(x) (volatile uint32_t*)(&map_base[((uint32_t)(x) & 0xFFFFFF) >> 2])
```

This single mapping covers GPO/GPI, FPGA Manager, LW-AXI window,
NIC-301, Reset Manager, and System Manager — all within the 16 MB window.

> [!CAUTION]
> The `/dev/mem` access requires root privileges. MiSTer's Linux image runs
> the binary as root. On a standard Linux installation, the `O_SYNC` flag
> is critical — without it, writes to GPO may be buffered by the CPU cache,
> breaking the SPI protocol timing.
