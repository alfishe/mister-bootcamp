[← Knowledge Base](../README.md)

# Framework (`sys/`)

The MiSTer framework layer — shared SystemVerilog modules that every core instantiates. This is the contract between the platform and individual cores: `sys_top.v` provides the hardware abstraction, `hps_io.sv` handles all HPS↔FPGA communication, and the SDRAM/DDR3 controllers manage external memory.

## Articles

| File | Topic | Quality |
|---|---|---|
| [sys_top.md](sys_top.md) | Top-level hardware abstraction: PLL tree, bridge wiring, GPO/GPI, HDMI/VGA muxing | 📋 Target: Deep |
| [hps_io_module.md](hps_io_module.md) | `hps_io.sv`: port map, parameters, opcode dispatch, status register | 📋 Target: Deep |
| [hps_bus.md](hps_bus.md) | HPS_BUS[48:0] — 49-bit parallel bus between sys_top and hps_io | 📋 Target: Deep |
| [spi_protocol.md](spi_protocol.md) | SPI-over-GPO: software-emulated SPI via FPGA Manager registers | 📋 Target: Deep |
| [fpga_loading.md](fpga_loading.md) | RBF bitstream loading: FPGA Manager, bridge teardown/restore, warm reboot | 📋 Target: Deep |
| [sdram_controller.md](sdram_controller.md) | `sdram.sv`: bank interleaving, refresh, dual-port, board detection | 📋 Target: Deep |
| [ddr3_architecture.md](ddr3_architecture.md) | F2H AXI bridge, DDR3 port allocation, bandwidth budget | 📋 Target: Deep |

## Cross-References

- [Hardware Platform](../01_hardware_platform/README.md) — physical board, memory map
- [Core Architecture](../03_core_architecture/README.md) — how cores use this framework
- [FPGA KB — HPS↔FPGA Bridges](https://github.com/alfishe/fpga-bootcamp/blob/main/10_embedded_linux/hps_fpga_bridges_intel_soc.md)
