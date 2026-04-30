# MiSTer FPGA Knowledge Base — Agent & Contributor Standards

> **Audience**: Any agent (human or AI) authoring, reviewing, or expanding documentation in this repository.
>
> This file defines the quality standards, formatting rules, and architectural conventions that every article must follow. Violations produce inconsistent documentation that misleads core developers and wastes contributor time.

---

## 1. Mission Statement

This repository is the **definitive technical reference** for the MiSTer FPGA platform — from the DE10-Nano hardware and Cyclone V SoC architecture through the `sys/` framework, the HPS binary, individual core implementations, and the surrounding ecosystem (RetroAchievements, build tooling, community scripts).

It serves three audiences simultaneously:
- **Retro gaming enthusiasts** who want to understand what they're running and why it matters
- **Software / Linux developers** who want to cross-compile, extend the HPS binary, or build custom Linux images
- **FPGA / HDL engineers** who want to write new cores or modify existing ones

Every article must be useful to at least one of these audiences, and the Overview section must provide clear entry points for all three.

---

## 2. Quality Tiers

### Deep (Target for all architectural articles)

A "Deep" article is the **last document you need to read** on its topic. It must contain:

| Requirement | Example |
|---|---|
| **Architectural diagram** (Mermaid) | System block diagram, data flow, state machine |
| **Register-level or code-level detail** | Verilog snippets from `hps_io.sv`, C excerpts from `user_io.cpp` |
| **Source attribution** | `Main_MiSTer/user_io.cpp:L142`, `Template_MiSTer/sys/hps_io.sv` |
| **Cross-references** | Links to related articles in this KB, the FPGA KB, or the Amiga Bootcamp KB |
| **Antipatterns / hazards** | "Do NOT use `quartus_sh --flow compile` on Apple Silicon — timing-driven synthesis crashes under Rosetta 2" |
| **Decision rationale** | Why the MiSTer framework uses software SPI over GPO instead of the Cyclone V's hard SPI peripheral |
| **Platform context** | How MiSTer's approach compares to Analogue Pocket, MiST, MiSTeX, or software emulation |

### Adequate (Acceptable for reference tables and user guides)

An "Adequate" article provides correct, complete information but may lack architectural diagrams, decision rationale, or platform comparisons. Acceptable for:
- UIO opcode reference tables
- MiSTer.ini variable reference
- Pin assignment tables
- Build quickstart guides

### Shallow (Unacceptable — must be upgraded)

A "Shallow" article is a stub, a copy-paste from a wiki, or a description without source-grounded detail. If an article is marked Shallow in `TODO.md`, it must be upgraded before the next review cycle.

---

## 3. Formatting Standards

### 3.1 Document Structure

Every article **must** begin with a navigation breadcrumb on line 1:

```markdown
[← Section Index](README.md) · [↑ Knowledge Base](../README.md)
```

Followed by a level-1 heading (`#`), then the body.

### 3.2 Hexadecimal Notation

| Context | Format | Example |
|---|---|---|
| SoC register addresses | C-style | `0xFF706010` |
| FPGA opcode references | Verilog-style | `` `h14 `` or `0x14` |
| Memory sizes | C-style with comment | `0x00800000` (8 MB) |
| Bit fields | Verilog range | `[15:0]`, `[48:0]` |
| Magic numbers | C-style | `0xBEEFB001` |

### 3.3 Source Attribution

**Every technical claim must cite its source.** Use this format:

```markdown
Source: `Main_MiSTer/user_io.cpp`, `Template_MiSTer/sys/hps_io.sv`
```

For GitHub-hosted sources, prefer clickable links to the MiSTer-devel organization:
```markdown
[`hps_io.sv`](https://github.com/MiSTer-devel/Template_MiSTer/blob/master/sys/hps_io.sv)
```

### 3.4 Code Blocks

- **Verilog/SystemVerilog**: Use ` ```verilog ` fence
- **VHDL**: Use ` ```vhdl ` fence (ascal.vhd, sigma-delta DAC, etc.)
- **C/C++**: Use ` ```c ` fence
- **Python**: Use ` ```python ` fence (build scripts, MRA tools)
- **Shell / Bash**: Use ` ```bash ` fence
- **Makefile**: Use ` ```makefile ` fence
- **INI files**: Use ` ```ini ` fence
- **Dockerfile**: Use ` ```dockerfile ` fence
- **Device Tree**: Use ` ```dts ` fence

Always include a source comment on the first line:
```verilog
// hps_io.sv — CONF_STR ROM readout
'h14: if(byte_cnt <= STRLEN) io_dout[7:0] <= conf_byte;
```

### 3.5 Diagrams

Use **Mermaid** for all diagrams. Every architectural article must contain at least one.

Preferred diagram types:
| Use case | Mermaid type |
|---|---|
| Data flow / system architecture | `flowchart LR` or `flowchart TD` |
| Protocol sequences (HPS↔FPGA) | `sequenceDiagram` |
| State machines (FPGA Manager) | `stateDiagram-v2` |
| Signal timing | ASCII art (Mermaid has no waveform support) |

### 3.6 Tables

Use tables for:
- Register bit assignments
- Opcode references
- Configuration variable references
- Feature comparison matrices

Always include column headers. Align columns for readability in source.

### 3.7 Alerts

Use GitHub-style alerts for critical information:

```markdown
> [!NOTE]
> Background context or implementation detail

> [!WARNING]
> Breaking change, compatibility issue, or common mistake

> [!CAUTION]
> Action that could brick hardware or corrupt data
```

### 3.8 Section README Files

Every numbered directory (`00_overview/`, `02_hardware_platforms/`, etc.) must contain a `README.md` with:
1. Section title and one-paragraph scope description
2. Article index table (filename, topic, quality tier)
3. Cross-references to related sections

---

## 4. Cross-Referencing Standards

### 4.1 Internal Cross-References

Use relative paths within this repository:
```markdown
See [HPS Bus Protocol](../06_fpga_subsystem/hps_bus.md) for the 49-bit bus architecture.
```

### 4.2 FPGA Knowledge Base Cross-References

When an article references general FPGA concepts (Cyclone V architecture, AXI bus, timing constraints, PLL configuration), link to the FPGA knowledge base rather than re-documenting:

```markdown
For Cyclone V SoC architecture details, see
[FPGA KB — Intel HPS-FPGA Integration](https://github.com/alfishe/fpga-bootcamp/blob/main/02_architecture/soc/hps_fpga_intel_soc.md).
```

### 4.3 Amiga Bootcamp Cross-References

When documenting the Minimig core or Amiga-specific hardware concepts, link to the Amiga Bootcamp:

```markdown
For the original Amiga chipset architecture (OCS/ECS/AGA), see
[Amiga Bootcamp — Custom Chips](https://github.com/alfishe/amiga-bootcamp/blob/main/01_hardware/common/custom_chips_overview.md).
```

### 4.4 GitHub Source References

For MiSTer source code, link to the MiSTer-devel organization:

| Repository | Content |
|---|---|
| `MiSTer-devel/Main_MiSTer` | HPS binary (C++) |
| `MiSTer-devel/Template_MiSTer` | Core template + `sys/` framework |
| `MiSTer-devel/Linux-Kernel_MiSTer` | Patched Linux kernel |
| `MiSTer-devel/u-boot_MiSTer` | Patched U-Boot |
| `MiSTer-devel/mr-fusion` | SD card image builder |
| `odelot/Main_MiSTer` | RetroAchievements fork |

---

## 5. Content Rules

### 5.1 No Wiki Copy-Paste

The [MiSTer wiki](https://github.com/MiSTer-devel/Main_MiSTer/wiki) is a user-facing reference. This KB is a **technical deep-dive** grounded in source code. If a wiki page covers a topic, our article must go deeper — showing the Verilog, the C code, the register addresses, and the design rationale.

### 5.2 No Speculation

Every claim must be grounded in:
- Source code (with file + line reference)
- Datasheet (with document number)
- Measured behavior (with methodology described)

If something is uncertain, mark it:
```markdown
> [!NOTE]
> **Unverified**: The exact DDR3 bandwidth available to the FPGA via F2H AXI has not been
> independently measured. The 800 MB/s figure is theoretical based on 32-bit @ 200 MHz.
```

### 5.3 Platform Context Sections

Architectural articles should include a brief comparison section showing how MiSTer's approach differs from:
- **Software emulation** (RetroArch, MAME, standalone emulators)
- **Analogue Pocket** (openFPGA framework)
- **MiST / SiDi** (predecessor platforms)
- **MiSTeX** (Xilinx/Zynq port)

This is not marketing — it's engineering context that helps readers understand design decisions.

### 5.4 Tone

Technically authoritative but not gatekeepy. Write for people who care about *why* CRTs look different from LCD scalers, not for people who need to be sold on the concept. No marketing fluff, no "easy as 1-2-3" condescension.

The retro community values depth and authenticity. If you can explain the polyphase filter coefficients in `ascal.vhd`, do it. If you can show why software SPI over GPO hits 150 MB/s, show the ARM assembly burst loop. Trust the reader.

---

## 6. File Naming Conventions

| Type | Convention | Example |
|---|---|---|
| Article | `snake_case.md` | `hps_bus.md`, `conf_str.md` |
| Section directory | `NN_descriptive_name/` | `06_fpga_subsystem/`, `14_extensions/` |
| Sub-directory | `snake_case/` | `build/`, `input/` |
| Image | `snake_case.png` | `system_block_diagram.png` |

---

## 7. Review Checklist

Before marking any article as "Deep" quality, verify:

- [ ] Navigation breadcrumb on line 1
- [ ] At least one Mermaid diagram (architectural articles)
- [ ] All code blocks have source attribution comments
- [ ] All register addresses use correct hex notation
- [ ] Cross-references to FPGA KB / Amiga Bootcamp where applicable
- [ ] No dead internal links
- [ ] GitHub source links point to correct MiSTer-devel repos
- [ ] Hazards / antipatterns documented with `> [!WARNING]` alerts
- [ ] Platform context section present (architectural articles)
- [ ] Article indexed in section README.md and root README.md
