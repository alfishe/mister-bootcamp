# 06 — FPGA Subsystem


[← Back to Index](../README.md)


## Articles

* [**sys_top.v — Top-Level Hardware Abstraction**](sys_top.md) — Deep dive into the root synthesis entity: ports, submodule map, SSPI command decoder, video/audio/memory pipelines, conditional compilation
* [**hps_io.sv — Core-Side Command Demultiplexer**](hps_io_module.md) — UIO command decoder, file I/O engine, joystick/keyboard/mouse, CONF_STR, status word, SD card, EXT_BUS
* [**DDR3 Architecture: F2SDRAM, sysmem_lite & ddram**](ddr3_architecture.md) — F2SDRAM bridge, safe terminator, ddr_svc arbiter, ddram.sv wrapper, ddram_ctrl.v cached controller, HPS bridge management
* [Framework Architecture & Overview](fpga_framework_overview.md)
* [HPS Bridge Reference (SSPI & Control)](hps_bridge_reference.md)
* [Video & Audio Pipelines](video_audio_pipelines.md)
* [Memory Controllers (SDRAM & DDR3)](memory_controllers.md)
* [SDRAM Timing Theory & Phase Alignment](sdram_timing_theory.md)
* [FPGA Performance Metrics & Utilization](fpga_performance_metrics.md)
* [FPGA Compilation & Timing Closure](fpga_compilation_guide.md)
* [FPGA Debugging & Telemetry Tools](fpga_debugging_tools.md)
* [Input Latency & SNAC](input_latency_and_snac.md)
* [FPGA Loading](fpga_loading.md)
