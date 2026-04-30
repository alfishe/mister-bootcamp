[← Overview](README.md) · [↑ Knowledge Base](../README.md)

# Getting Started with MiSTer FPGA

MiSTer is an open-source platform that recreates classic hardware in programmable silicon. Whether you're here to play retro games, learn computer architecture, build an arcade cabinet, or write FPGA cores — this page will get you oriented and point you to the right resources.

> **New to MiSTer entirely?** Read the [Platform Overview](README.md) first for the full story — what it is, how it evolved, and why it matters.

---

## Find Your Path

| You are... | What you'll find here | Jump to |
|---|---|---|
| 🎮 **Retro gamer** | Cycle-accurate classics with zero input lag — the way they were meant to be played | [Retro Gaming →](#-retro-gaming) |
| 🕹️ **Arcade builder** | Drop MiSTer into a real JAMMA cabinet with authentic controls | [Arcade →](#️-arcade-builder) |
| 📺 **CRT / analog enthusiast** | Real 15kHz scanlines — not shader approximations | [Analog Video →](#-crt--analog-video) |
| 🎓 **CS / EE student** | Study real CPU architectures (68000, Z80, R3000A) in synthesizable RTL you can read | [Student →](#-student--learner) |
| 💻 **Software developer** | Hack on a custom ARM Linux stack — C++ control plane, USB HID, video scaling | [Software Dev →](#-software-developer) |
| ⚡ **FPGA engineer** | Write cores on a mature open-source framework — video, input, storage handled for you | [FPGA Dev →](#-fpga--hdl-engineer) |
| 🔧 **Tinkerer / maker** | Build custom cases, solder addon boards, 3D-print mounts, mod everything | [Tinkerer →](#-tinkerer--maker) |
| 📡 **Preservationist** | Cycle-accurate hardware means bit-perfect preservation and verification | [Preservation →](#-preservationist--archivist) |

---

## Hardware: What You Need

*(Shared across all paths — everyone starts here)*

### Minimum Kit

| Component | Purpose | Approx. Cost |
|---|---|---|
| **DE10-Nano** board | The FPGA platform itself (Cyclone V 5CSEBA6 SoC) | ~$225 |
| **128 MB SDRAM** module | Required by most cores for fast scratch memory | ~$20–45 |
| **MicroSD card** (16 GB+) | Stores Linux OS, cores, ROMs, save files | ~$10 |
| **5V / 2A USB-C power supply** | Powers the board | ~$10 |
| **USB keyboard or gamepad** | Basic input | (you have one) |
| **HDMI display** | Video output | (you have one) |

**Total minimum: ~$265–290** (DE10-Nano path)

> **Pricing note**: Costs above are actual as of April 2026 and will fluctuate. Always verify current pricing at vendor sites. See the [Platform Overview](README.md) for vendor links.

### Budget Alternatives

| Board | Cost | Trade-off |
|---|---|---|
| **QMTech Cyclone V** | ~$90–110 | Same FPGA, different GPIO layout — needs QMTech-specific I/O boards |
| **MiSTer Pi** (Taki Udon) | ~$80–100 | Raspberry Pi form factor, full core compatibility |

**Budget total: ~$120–155** + SDRAM + SD card

### Pre-Built Bundles

If you want plug-and-play, see the [vendor table in the Platform Overview](README.md#-affordable-powerful-off-the-shelf-hardware) — pre-assembled kits run **~$400–500** with everything included.

### Optional Addons

| Addon | What it unlocks |
|---|---|
| **I/O Board** | Analog VGA/component output for CRTs, SNAC connector for native controllers |
| **USB Hub** | Multiple controllers, WiFi adapter, keyboard + gamepad simultaneously |
| **Direct Video adapter** | Cheap CRT output without the I/O board (active HDMI-to-VGA) |
| **MT32-Pi** | Roland MT-32 synthesizer emulation for DOS/PC games via Raspberry Pi |

> For full hardware details → [DE10-Nano Reference](../02_hardware_platforms/de10_nano.md) · [Addon Boards](../02_hardware_platforms/addon_boards.md)

---

## First Boot: Universal Setup

This is the same regardless of your persona — everyone runs through these steps once.

### 1. Flash the SD Card

1. Download the latest **Mr. Fusion** image from [GitHub releases](https://github.com/MiSTer-devel/mr-fusion/releases)
2. Flash it to your MicroSD card using [balenaEtcher](https://etcher.balena.io/), `dd`, or similar
3. Insert the SD card into the **bottom slot** of the DE10-Nano

### 2. First Power-On

1. Connect HDMI and USB keyboard/gamepad
2. Power on — Mr. Fusion will automatically partition the SD card, install the base MiSTer OS, and reboot
3. You'll land at the **MiSTer main menu** — a text-based OSD over a blue background

### 3. Run Update All

The `update_all` script (by [Theypsilon](https://github.com/theypsilon/Update_All_MiSTer)) downloads all cores, firmware updates, and support files automatically.

1. Press **F12** (or your OSD button) → navigate to **Scripts**
2. If `update_all` isn't there yet:
   - Download `update_all.sh` from the [GitHub repo](https://github.com/theypsilon/Update_All_MiSTer)
   - Copy it to the `/Scripts/` folder on the SD card
3. Run it — first run takes **15–30 minutes** over Ethernet
4. Press **UP** during the countdown to enter settings (enable BIOS downloads, arcade ROMs, etc.)

### 4. Configure Your Controller

From the main menu, select **Define Joystick Buttons** and follow the prompts. MiSTer supports virtually any USB HID gamepad — 8BitDo, Xbox, PlayStation, fight sticks, etc.

### 5. Network Access (Optional but Recommended)

| Method | How |
|---|---|
| **Ethernet** | Plug in — DHCP auto-configures |
| **WiFi** | Run the `wifi` script from the Scripts menu |
| **SSH** | `ssh root@<mister-ip>` (default password: `1`) |
| **Samba/SMB** | Access `\\mister\` from any machine on your network — drag and drop ROMs |
| **FTP** | Standard FTP on port 21 |

> For advanced networking → [WiFi & Network Setup](../12_networking/wifi_setup.md)

---

## 🎮 Retro Gaming

**You want to**: Play classic consoles and computers with perfect accuracy — the real hardware experience on a modern display.

### Why MiSTer Over Software Emulation?

Software emulators run your game inside a program. MiSTer **rebuilds the original hardware** in programmable silicon. The practical differences:

- **Zero additional input lag** — FPGA processes inputs in the same clock cycle, no frame buffering
- **Cycle-accurate timing** — games that rely on precise hardware quirks (raster effects, audio sync, coprocessor handshakes) just work
- **No host OS overhead** — no Windows update interrupting your speedrun

### Getting Playing

1. After running `update_all`, you'll have **200+ cores** available — from Atari 2600 to PlayStation
2. Navigate the OSD: cores are organized as `_Console/`, `_Computer/`, `_Arcade/`
3. Load a core → load a ROM via the OSD file browser → play

### Recommended First Cores

| Core | Why start here |
|---|---|
| **SNES** | Near-perfect cycle accuracy, excellent coprocessor support (GSU, SA-1, DSP) |
| **Genesis / Mega Drive** | Complete with Mega CD and 32X support |
| **NES** | Massive mapper support, rock-solid |
| **Game Boy / GBA** | Portables run beautifully |
| **PSX** | The "watershed" 32-bit core — CHD support, memory card saves |
| **ao486** | A full 486 PC — run DOS games with Sound Blaster and MIDI |

### Key Features for Gamers

- **Save states** — snapshot/restore at any point (most cores)
- **RetroAchievements** — earn achievements on real hardware ([details](../14_extensions/retroachievements.md))
- **OSD-based configuration** — change video modes, filters, cheats without leaving the game
- **Per-core INI overrides** — different video settings for each system

> **Deep dive**: [Video & Audio Pipeline](../09_video_audio/hdmi_scaler.md) · [Save States](../13_save_states/save_state_architecture.md) · [Configuration Guide](../05_configuration/mister_ini_guide.md)

---

## 🕹️ Arcade Builder

**You want to**: Put MiSTer inside a real arcade cabinet with JAMMA wiring and authentic controls.

### The MiSTercade Approach

The **MiSTercade** board (from MiSTer Addons) is a purpose-built carrier that turns the DE10-Nano into a JAMMA-compatible drop-in replacement for original arcade PCBs:

- Standard JAMMA edge connector — plugs directly into your cabinet's wiring harness
- Active-low button mapping matches arcade conventions
- Supports 15kHz CRT output natively
- DIP switch and game configuration via OSD

### MRA Files: Arcade ROM Management

Arcade cores on MiSTer use **MRA (MiSTer ROM Archive)** XML files to describe how ROM dumps map to the FPGA core:

- Each MRA file specifies ROM merge order, byte swaps, and interleaving
- One FPGA core can serve dozens of game variations (e.g., the CPS1 core runs 50+ games)
- DIP switch settings are configured per-MRA

### SNAC: Native Controller Wiring

The **Serial Native Accessory Converter** (SNAC) connects original arcade controls (joysticks, spinners, trackballs, light guns) directly to the FPGA fabric — bypassing USB entirely for true zero-lag input.

> **Deep dive**: [Arcade Cores & MRA](../08_fpga_cores_catalog/arcade_and_mra.md) · [SNAC & LLAPI](../10_input_devices/snac_llapi.md)

---

## 📺 CRT / Analog Video

**You want to**: Connect MiSTer to a real CRT or PVM for authentic scanlines and phosphor glow — not shader approximations.

### Output Options

| Method | What you need | Signal |
|---|---|---|
| **I/O Board** → VGA cable → CRT | I/O Board (~$50) | 15kHz RGB or 31kHz VGA |
| **I/O Board** → component transcoder | I/O Board + transcoder | YPbPr component |
| **Direct Video** → HDMI-to-VGA adapter | Active adapter (~$10) | 15kHz RGB (active adapter required — passive won't work) |
| **HDMI → OSSC/RetroTINK** | HDMI output → external scaler → CRT | Flexible, but adds a processing step |

### Key MiSTer.ini Settings for CRT

```ini
composite_sync=1        ; Enable csync for arcade monitors
vga_scaler=0            ; Bypass the HDMI scaler for analog output
forced_scandoubler=0    ; Keep native 15kHz for CRT
vsync_adjust=2          ; Lock to original refresh rate (critical for smooth scrolling)
```

### Why It Matters

Software emulators output video at the host display's refresh rate (typically 60.00 Hz). Original hardware ran at non-standard rates (59.94 Hz NTSC, 50.08 Hz PAL, 57.44 Hz arcade). MiSTer's FPGA generates the **exact original timing** — so CRTs display the signal identically to original hardware, with correct aspect ratio, refresh rate, and blanking intervals.

> **Deep dive**: [Analog Video Output](../09_video_audio/analog_video.md) · [HDMI Scaler](../09_video_audio/hdmi_scaler.md) · [INI Reference](../05_configuration/mister_ini_guide.md)

---

## 🎓 Student / Learner

**You want to**: Understand how real CPUs, GPUs, and sound chips work — not from textbook block diagrams, but from **actual synthesizable implementations** you can read, modify, and run.

### What MiSTer Teaches You

Textbooks give you block diagrams. MiSTer gives you the blocks — real, synthesizable, running actual software. It's a **controlled descent** from high-level concepts to the deep end of digital design, rather than jumping into cold water from academic abstractions.

**Classic hardware — the retro cores:**

| Topic | What you'll see in MiSTer cores |
|---|---|
| **CPU architecture** | Pipelined 68000, microcoded Z80, MIPS R3000A — with real hazard handling, bus arbitration, and exception logic |
| **Memory systems** | SDRAM controllers with bank interleaving, refresh scheduling, multi-port access patterns |
| **Video generation** | Tile engines, sprite multiplexers, raster IRQ timing, horizontal/vertical blanking |
| **Audio synthesis** | FM synthesis (YM2612), wavetable (SNES SPC700), PSG, sigma-delta DACs |
| **Bus protocols** | Shared-bus arbitration, DMA controllers, coprocessor handshakes (68000 ↔ Z80 in Genesis) |
| **Real-time constraints** | Meeting exact cycle counts in 180ns windows — not "close enough" |

**Modern SoC — the platform itself:**

The Cyclone V SoC on the DE10-Nano *is* a modern heterogeneous system-on-chip. Building and working with MiSTer teaches you the same concepts used in industrial embedded design:

| Topic | Where you see it on MiSTer |
|---|---|
| **ARM + FPGA co-design** | HPS (dual Cortex-A9) ↔ FPGA fabric bridging — the exact paradigm used in Xilinx Zynq, Intel Agilex, and every modern FPGA SoC |
| **AXI / Avalon bus interconnect** | F2H (FPGA-to-HPS) and H2F bridges carry DDR3 traffic, status registers, and file I/O — standard on-chip interconnect protocols |
| **PLL / clock tree design** | `sys_top.v` contains multi-output PLLs generating pixel clocks, CPU clocks, SDRAM clocks — real clock domain crossing |
| **DDR3 memory controllers** | The HPS DDR3 interface and FPGA-side SDRAM controllers show two different approaches to high-speed memory |
| **Embedded Linux BSP** | U-Boot → kernel → userspace on ARM — cross-compilation, device tree, FPGA Manager driver |
| **Hardware-software co-processing** | The `Main_MiSTer` C++ binary and `hps_io.sv` Verilog module form a tightly-coupled HW/SW interface — register-level handshaking, interrupt signaling, shared memory |
| **Video scaling pipelines** | `ascal.vhd` is a production polyphase scaler — the same algorithm used in broadcast video hardware |
| **USB HID stack** | Full USB host implementation — descriptor parsing, HID reports, device enumeration — on a real ARM Linux stack |

This is the same technology stack you'd encounter designing automotive ECUs, industrial controllers, or aerospace systems — just wrapped in a more approachable (and more fun) package.

### Suggested Learning Path

| Step | Core | Complexity | What you'll learn |
|---|---|---|---|
| 1 | **ZX Spectrum** | Low | Simple Z80, ULA timing, memory-mapped video, trivial bus |
| 2 | **NES** | Medium | 6502 + PPU + APU interaction, mapper switching, sprite evaluation pipeline |
| 3 | **Genesis** | High | Dual-CPU (68000 + Z80), VDP with multi-layer scrolling, YM2612 FM synthesis |
| 4 | **SNES** | High | 5A22 CPU + S-PPU + S-SMP + coprocessors (GSU, SA-1, DSP) |
| 5 | **PSX** | Advanced | MIPS R3000A + GTE (geometry transform engine) + GPU + DMA channels |

### Tools You'll Need

- **Intel Quartus Prime Lite 17.0.2** (free) — synthesis, simulation, timing analysis
- **ModelSim** (included with Quartus) — RTL simulation
- **GTKWave** — open-source waveform viewer
- A text editor that handles Verilog/SystemVerilog syntax (VS Code + Verilog extension)

> **Deep dive**: [Platform Architecture](../01_system_architecture/platform_architecture.md) · [Template Walkthrough](../07_fpga_cores_architecture/template_walkthrough.md) · [FPGA Knowledge Base](https://github.com/alfishe/fpga-bootcamp)

---

## 💻 Software Developer

**You want to**: You know Linux, C/C++, and embedded systems. MiSTer looks like a small ARM board running Linux — and it is — but the platform offers far more than just another embedded target.

### The Software Stack

MiSTer's "host" side runs on the ARM Cortex-A9 inside the Cyclone V SoC. It's a minimal Linux system:

| Component | What it does | Language |
|---|---|---|
| **Main_MiSTer** | The main binary — USB input, OSD rendering, video scaling config, core lifecycle, file I/O | C++ (~50K SLOC) |
| **Linux kernel** | Patched 5.x kernel with FPGA Manager driver, HPS bridge support | C |
| **U-Boot** | Bootloader — handles warm/cold reboot, FPGA pre-loading, OCRAM handoff | C |
| **System scripts** | Startup, WiFi, update management, core switching | Bash |

### Beyond the Familiar Linux Surface

If you've worked on Raspberry Pi, BeagleBone, or any ARM SBC, the Linux side will feel familiar. What makes MiSTer unique is everything *below* and *around* it:

| Domain | What's different here |
|---|---|
| **FPGA co-processing** | The ARM CPU talks to live FPGA fabric via F2H/H2F AXI bridges. `Main_MiSTer` writes configuration registers, reads status, and transfers files directly to hardware that's being dynamically reconfigured. This isn't GPIO toggling — it's register-level HW/SW co-design. |
| **Live hardware reconfiguration** | The FPGA Manager driver lets you load entirely different hardware (a SNES, a Genesis, an arcade board) at runtime via `ioctl()`. No reboot. The "peripheral" changes its identity. |
| **Peripheral ecosystem** | I/O board (analog video DAC, audio, SNAC), USB hub, SDRAM modules — each with its own driver or userspace interface. The expansion bus is physical, not virtual. |
| **Video pipeline control** | The software side configures a hardware polyphase scaler (`ascal.vhd`) — setting filter coefficients, aspect ratios, and scan modes via mapped registers. You're programming a real video processing chain. |
| **Storage and file I/O** | ROM loading, VHD mounting, save files — all marshaled from Linux userspace to FPGA fabric over the HPS bridge. The file system is your interface to the hardware. |

### Getting Set Up for Development

1. **Cross-compilation toolchain**: `arm-linux-gnueabihf-gcc` (Linaro or Ubuntu packages)
2. Clone `Main_MiSTer`: `git clone https://github.com/MiSTer-devel/Main_MiSTer.git`
3. Build: `make CROSS_COMPILE=arm-linux-gnueabihf-`
4. Deploy: Copy the resulting binary to `/media/fat/MiSTer` on the SD card (via SSH/Samba)
5. Reboot or run directly from SSH

### What's Interesting to Hack On

**Platform infrastructure:**
- **USB HID input pipeline** — controller mapping, analog stick handling, autofire
- **Video scaling configuration** — INI parsing, scaler parameter setup
- **OSD rendering** — menu system, file browser, configuration UI
- **Custom Linux images** — Buildroot-based, stripped-down system builds
- **Networking stack** — WiFi management, remote access, web-based control

**Extensions and integrations:**
- **RetroAchievements integration** — the `odelot/Main_MiSTer` fork that adds rcheevos via DDRAM bridge
- **mrext (Wizzomafizzo)** — NFC game launching, web remote, system management tools
- **MT32-Pi integration** — MIDI routing from ao486/DOS cores to external Raspberry Pi synth

**Developing *for* the emulated systems:**
This is the angle most people miss. MiSTer gives you cycle-accurate recreations of classic computers (ao486, Amiga, Atari ST, MSX, ZX Spectrum) that are *better development targets than the originals*:
- Write 68000 assembly for the Amiga core and test on hardware that behaves identically to a real A500
- Develop DOS TSRs or VGA demos on ao486 with instant reboot and save states
- Cross-compile C for retro targets and deploy via Samba — no floppy disks, no serial cables
- Debug timing-sensitive code on a platform that doesn't have the variability of aging hardware

### Contribution Workflow

1. Fork on GitHub → feature branch → PR against `MiSTer-devel/Main_MiSTer`
2. Test on real hardware (no x86 simulator — ARM binary on real DE10-Nano)
3. Sorgelig reviews and merges — he's responsive but thorough

> **Deep dive**: [HPS Binary Architecture](../04_hps_binary/architecture.md) · [Build Toolchain](../04_hps_binary/build/overview.md) · [Linux Stack](../03_hps_linux/kernel/) · [Custom Linux Images](../16_advanced_topics/custom_linux.md)

---

## ⚡ FPGA / HDL Engineer

**You want to**: Write new cores, port existing ones, or contribute to the MiSTer framework itself. But MiSTer offers more than just a sandbox for isolated RTL — it's a complete SoC design platform where your core lives inside a real heterogeneous system.

### The Development Model

MiSTer uses a standardized framework that handles all the "plumbing" — you focus on the core logic:

```
Your core (emu module)
    ↕
sys/ framework (video scaling, audio mixing, input routing, HPS communication)
    ↕
sys_top.v (top-level: PLL tree, DDR3 bridge, HDMI TX, GPIO)
    ↕
DE10-Nano hardware
```

**You write the `emu` module.** The framework gives you:
- HDMI output with polyphase scaling (just provide pixel clock + RGB + sync)
- Audio output (just provide left/right samples)
- USB input (gamepad buttons arrive as a simple bit vector)
- File I/O (ROM loading, save files — handled over HPS bus)
- OSD configuration (define your menu in a `CONF_STR` parameter string)

### Beyond Isolated Cores — The Full SoC

Writing a core teaches you one system's architecture. But the **platform surrounding your core** is where production-grade SoC design skills live:

| What you'll encounter | Where | Why it matters |
|---|---|---|
| **HPS↔FPGA bridge protocols** | `hps_io.sv` | AXI/Avalon register-mapped communication between ARM and FPGA fabric — the same paradigm used in Zynq, Versal, and every Intel FPGA SoC |
| **SDRAM controller design** | `sdram.sv` | Bank interleaving, refresh scheduling, multi-port arbitration — you'll see (and eventually need to optimize) a real memory controller under tight timing budgets |
| **PLL and clock domain management** | `sys_top.v` | Multi-output PLLs generating independent clock domains (pixel clock, CPU clock, SDRAM clock) with proper CDC (clock domain crossing) |
| **DDR3 via F2H bridge** | Framework internals | Cores that need large memory (PSX, N64, ao486) route through the HPS DDR3 — AXI burst transactions, cache coherency, bandwidth sharing with Linux |
| **Video pipeline internals** | `ascal.vhd` | A polyphase scaler written in VHDL — if you want to understand real-time image processing in hardware, read this |
| **Analog output generation** | I/O Board interface | DAC driving, sync generation, 15kHz/31kHz mode switching — analog signal integrity in a digital design |
| **GPIO and expansion** | `sys_top.v`, I/O Board | Direct FPGA pin mapping to physical connectors — SNAC controllers, buttons, LEDs, accent lighting |
| **Timing closure** | Quartus Fitter | Fitting a complex core into the Cyclone V's 41K ALMs under real timing constraints teaches you more about FPGA design than any tutorial |

This is the difference between writing Verilog in a simulator and shipping a product. Your core doesn't exist in a vacuum — it negotiates with a memory controller, shares bus bandwidth with an ARM CPU, drives real analog and digital outputs, and has to meet timing on real silicon.

### Quick Start: Your First Core

1. Clone `Template_MiSTer`: `git clone https://github.com/MiSTer-devel/Template_MiSTer.git`
2. Install **Intel Quartus Prime Lite 17.0.2** — this is the community-standard version
3. Open the `.qpf` project file in Quartus
4. Modify the `emu` module in the template — start with a simple test pattern generator
5. Compile → generates an `.rbf` file
6. Copy `.rbf` to the appropriate folder on the SD card (`_Console/`, `_Computer/`, or `_Arcade/`)
7. Load from the MiSTer menu → see your output on screen

### The `sys/` Framework — Your Black Box (Until It Isn't)

**Don't modify `sys/`** — it's maintained by Sorgelig and updated globally. Treat it as a library. But *do* read it:

| Framework Module | What it provides | What you'll learn by reading it |
|---|---|---|
| `hps_io.sv` | HPS↔FPGA communication — input, file I/O, config, status | Register-mapped bus protocol design, SPI tunneling |
| `ascal.vhd` | Polyphase video scaler — native resolution → HDMI | Real-time image processing, line buffer management |
| `sys_top.v` | Top-level wiring — PLL, DDR3, GPIO, HDMI TX | SoC-level integration, constraint-driven design |
| `sdram.sv` | SDRAM controller — bank interleaving, refresh, multi-port | Memory controller architecture, arbitration |

### Jotego's jtframe (Arcade Developers)

If you're writing arcade cores, look at [jtframe](https://github.com/jotego/jtframe) — Jose Tejada's framework built on top of `sys/` that provides reusable arcade components: Yamaha FM synths, CPS tile engines, Z80 integration, and a standardized game loop.

### Community and Resources

- **MiSTer Discord** `#dev` channels — the most active place for core development discussion
- **pram0d's blog** — detailed FPGA core development walkthroughs
- Existing cores on GitHub — the best learning resource is reading working code

> **Deep dive**: [Template Walkthrough](../07_fpga_cores_architecture/template_walkthrough.md) · [sys_top.v](../06_fpga_subsystem/sys_top.md) · [hps_io.sv](../06_fpga_subsystem/hps_io_module.md) · [Build Pipeline](../07_fpga_cores_architecture/build/overview.md)

---

## 🔧 Tinkerer / Maker

**You want to**: Build, solder, 3D-print, and customize the hardware — MiSTer is an open platform that invites modification.

### The Addon Ecosystem

MiSTer's modular stacking design means there's always something to build or upgrade:

| Project | Difficulty | What you get |
|---|---|---|
| **SDRAM module** soldering | Beginner | Required for most cores — kits available, or buy pre-assembled |
| **USB Hub board** | Beginner | Multi-controller support — simple header connection |
| **I/O Board** | Intermediate | Analog video output, SNAC connector, audio jack |
| **Custom case** (3D print) | Intermediate | Dozens of designs on Thingiverse/Printables — console-style, vertical tower, bartop |
| **SNAC adapters** | Intermediate | Build your own adapters for NES, SNES, Genesis, Neo Geo controllers |
| **Direct Video mod** | Beginner | Cheap CRT output via active HDMI-to-VGA adapter |
| **MiSTer Multisystem** | Advanced | Full console-style motherboard — replaces the stacking design entirely |
| **Bartop arcade build** | Advanced | Integrate MiSTer + MiSTercade into a custom cabinet |

### Resources

- **Thingiverse / Printables** — search "MiSTer FPGA" for case designs
- **MiSTer Addons** — commercial cases, boards, and accessories
- **MiSTer FPGA Discord** `#hardware` — community builds, troubleshooting
- **GitHub: MiSTer-devel** — schematics and KiCad files for official addon boards

> **Deep dive**: [Addon Boards](../02_hardware_platforms/addon_boards.md) · [DE10-Nano Reference](../02_hardware_platforms/de10_nano.md)

---

## 📡 Preservationist / Archivist

**You want to**: Use cycle-accurate hardware recreation for serious preservation work — verification, analysis, and tool-assisted research.

### Why FPGA Matters for Preservation

Software emulators are invaluable preservation tools, but they have inherent limitations:

- **Timing approximation** — software can't reproduce sub-cycle analog interactions (e.g., open-bus behavior, capacitor decay, analog audio mixing)
- **Frame-based execution** — software processes one frame at a time; real hardware is continuously parallel
- **Host dependency** — emulator behavior can vary with host CPU, OS scheduler, driver version

FPGA recreation eliminates these variables. The hardware behaves identically every time because it *is* the hardware — just built from different transistors.

### What You Can Do

| Activity | How MiSTer helps |
|---|---|
| **TAS (Tool-Assisted Speedruns/Superplay)** | SNAC + cycle-accurate cores = tournament-legal hardware for TAS verification |
| **Accuracy testing** | Run test ROMs (Blargg, TomHarte, Acid2) against real FPGA output to validate core fidelity |
| **Analog capture** | I/O Board analog output feeds directly into capture cards for archival-quality video |
| **Behavioral analysis** | Study undocumented hardware quirks — open-bus reads, uninitialized RAM patterns, PPU race conditions |
| **Software verification** | Confirm that dumped ROMs boot and run identically to original cartridges/discs |

### Tools and References

- **Test ROM suites** — Blargg (NES/GB), TomHarte (Z80/6502/68000), Acid2 (SNES PPU)
- **SNAC** — direct controller connection for input-deterministic testing
- **Save states** — snapshot mid-execution for analysis
- **SSH access** — script automated test runs against multiple cores

> **Deep dive**: [Platform Overview — Fidelity Milestones](README.md#fidelity-milestones) · [SNAC & LLAPI](../10_input_devices/snac_llapi.md)

---

## Read Also

| Document | What it covers |
|---|---|
| [Platform Overview](README.md) | What MiSTer is, historical evolution (Minimig → MiST → MiSTer), preservation philosophy |
| [Platform Architecture](../01_system_architecture/platform_architecture.md) | Deep technical reference: Cyclone V SoC, HPS↔FPGA bridges, SDRAM architecture, data flow pipelines |
| [Ecosystem & Community Tools](../15_ecosystem/README.md) | Derived projects, alternative platforms, hardware accessories, community scripts |
| [Knowledge Base Index](../README.md) | Full documentation suite — all sections and articles |
