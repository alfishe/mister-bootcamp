[← Section Index](README.md) · [↑ Knowledge Base](../README.md)

# MiSTer FPGA Audio Pipeline: Deep Dive

A source-grounded architectural analysis of how audio flows from an emulation core's output through filtering, mixing, and DAC conversion to the three physical output interfaces. Every code reference is traced to `Template_MiSTer/sys/audio_out.v`.

## Table of Contents

1. [Top-Level Architecture](#1-top-level-architecture)
2. [Clock Enable Generation](#2-clock-enable-generation)
3. [Input Synchronizer](#3-input-synchronizer)
4. [IIR Filter](#4-iir-filter)
5. [DC Blocker](#5-dc-blocker)
6. [Mixer and Attenuator (`aud_mix_top`)](#6-mixer-and-attenuator-aud_mix_top)
7. [Output Interfaces](#7-output-interfaces)
8. [Startup Muting Sequence](#8-startup-muting-sequence)
9. [Antipatterns and Common Pitfalls](#9-antipatterns-and-common-pitfalls)
10. [Platform Context](#10-platform-context)

---

## 1. Top-Level Architecture

Source: [`audio_out.v`](https://github.com/MiSTer-devel/Template_MiSTer/blob/master/sys/audio_out.v)

The primary orchestration of the audio pipeline occurs within `audio_out.v`. The module takes two audio sources — core audio (`core_l`/`core_r`) and Linux ALSA audio (`alsa_l`/`alsa_r`) — processes them through a filtering and mixing chain, and fans out to three physical outputs.

```mermaid
flowchart TD
    subgraph "Inputs"
        CL["core_l[15:0]"]:::core
        CR["core_r[15:0]"]:::core
        AL["alsa_l[15:0]"]:::linux
        AR["alsa_r[15:0]"]:::linux
    end

    subgraph ProcessingChain ["Processing Chain (audio_out.v)"]
        SYNC[Input Synchronizer<br/>double-flop + stable check]:::proc
        IIR[IIR_filter<br/>Configurable 2nd-order]:::proc
        DCB[DC_blocker<br/>High-pass ~1 Hz]:::proc
        MIX_L[aud_mix_top (L)<br/>ALSA mix + crossfeed + attenuate + clamp]:::proc
        MIX_R[aud_mix_top (R)<br/>ALSA mix + crossfeed + attenuate + clamp]:::proc
    end

    subgraph Outputs
        I2S[i2s.v<br/>I2S to HDMI TX]:::out
        SPDIF[spdif.v<br/>TOSLINK Optical]:::out
        DAC_L[sigma_delta_dac (L)<br/>3.5mm Analog]:::out
        DAC_R[sigma_delta_dac (R)<br/>3.5mm Analog]:::out
    end

    CL & CR --> SYNC
    SYNC --> IIR
    IIR --> DCB
    DCB -->|adl| MIX_L
    DCB -->|adr| MIX_R
    AL --> MIX_L
    AR --> MIX_R
    MIX_L -->|"al[15:0]"| I2S
    MIX_R -->|"ar[15:0]"| I2S
    MIX_L -->|al| SPDIF
    MIX_R -->|ar| SPDIF
    MIX_L -->|al| DAC_L
    MIX_R -->|ar| DAC_R
```

### Module Parameters and Ports

```verilog
// audio_out.v:L1-43
module audio_out #(parameter CLK_RATE = 24576000) (
    input        reset,
    input        clk,
    input        sample_rate,      // 0=48kHz, 1=96kHz
    input [31:0] flt_rate,         // IIR filter clock enable rate
    input [39:0] cx,               // IIR x-coefficients (combined)
    input  [7:0] cx0, cx1, cx2,   // IIR x-coefficient scale factors
    input [23:0] cy0, cy1, cy2,   // IIR y-coefficients
    input  [4:0] att,              // Volume attenuation (0=max, 31=mute)
    input  [1:0] mix,              // Stereo crossfeed mode
    input        is_signed,        // Core output format: 0=unsigned, 1=signed
    input [15:0] core_l, core_r,   // Core audio input
    input [15:0] alsa_l, alsa_r,   // Linux ALSA audio input
    output       i2s_bclk, i2s_lrclk, i2s_data,  // I2S output
    output       spdif,            // SPDIF output
    output       dac_l, dac_r      // Sigma-Delta analog output
);
```

---

## 2. Clock Enable Generation

Source: `audio_out.v:L45-141`

The audio pipeline runs entirely on a single clock domain (`clk`, typically 24.576 MHz from `pll_audio`), using clock-enable pulses rather than separate clock domains. This eliminates clock-domain crossing issues.

### 2.1 Master Clock Enable (`mclk_ce`)

The master clock enable runs at the bit clock rate for SPDIF (48 kHz × 32 bits × 2 channels = 3.072 MHz at 48 kHz, or 6.144 MHz at 96 kHz):

```verilog
// audio_out.v:L53-63
localparam CE_RATE = AUDIO_RATE * AUDIO_DW * 8;  // 48000*16*8 = 6144000
wire [31:0] real_ce = sample_rate ? {CE_RATE[30:0],1'b0} : CE_RATE[31:0];

reg mclk_ce;
always @(posedge clk) begin
    reg [31:0] cnt;
    mclk_ce = 0;
    cnt = cnt + real_ce;
    if(cnt >= CLK_RATE) begin
        cnt = cnt - CLK_RATE;
        mclk_ce = 1;
    end
end
```

This is a fractional accumulator — it generates `mclk_ce` pulses at the exact rate required, even though `CLK_RATE` is not an integer multiple of `CE_RATE`. The accumulator is self-correcting: any jitter from one cycle is compensated in the next.

### 2.2 I2S Clock Enable (`i2s_ce`)

```verilog
// audio_out.v:L65-73
reg i2s_ce;
always @(posedge clk) begin
    reg div;
    i2s_ce <= 0;
    if(mclk_ce) begin
        div <= ~div;
        i2s_ce <= div;  // toggle: i2s_ce = mclk_ce / 2
    end
end
```

`i2s_ce` is half the rate of `mclk_ce`, producing one I2S bit clock per two `mclk_ce` pulses.

### 2.3 Sample Rate Clock Enable (`sample_ce`)

```verilog
// audio_out.v:L117-129
reg sample_ce;
always @(posedge clk) begin
    reg [8:0] div = 0;
    reg [1:0] add = 0;
    div <= div + add;
    if(!div) begin
        div <= 2'd1 << sample_rate;  // 48kHz: div=1, 96kHz: div=2
        add  <= 2'd1 << sample_rate;
    end
    sample_ce <= !div;
end
```

At 48 kHz (`sample_rate=0`): `div` counts by 1, giving a `sample_ce` pulse every 256 `mclk_ce` cycles (6.144 MHz / 256 / 2 = 48 kHz... approximately). At 96 kHz (`sample_rate=1`): counts by 2, doubling the sample rate.

### 2.4 Filter Clock Enable (`flt_ce`)

```verilog
// audio_out.v:L131-141
reg flt_ce;
always @(posedge clk) begin
    reg [31:0] cnt = 0;
    flt_ce = 0;
    cnt = cnt + {flt_rate[30:0],1'b0};
    if(cnt >= CLK_RATE) begin
        cnt = cnt - CLK_RATE;
        flt_ce = 1;
    end
end
```

The IIR filter runs at a configurable rate set by `flt_rate`, using the same fractional accumulator technique. This allows the filter to process audio at rates other than the output sample rate, enabling oversampled filtering.

---

## 3. Input Synchronizer

Source: `audio_out.v:L143-153`

Core audio inputs are asynchronous to `clk`. The synchronizer uses a **double-flop with stability check** to prevent metastability:

```verilog
// audio_out.v:L143-153
reg [15:0] cl, cr;
always @(posedge clk) begin
    reg [15:0] cl1, cl2;
    reg [15:0] cr1, cr2;

    cl1 <= core_l; cl2 <= cl1;
    if(cl2 == cl1) cl <= cl2;   // only update when stable

    cr1 <= core_r; cr2 <= cr1;
    if(cr2 == cr1) cr <= cr2;   // only update when stable
end
```

This is a three-stage design:
1. `cl1` captures the asynchronous input (first flop, metastability risk)
2. `cl2` re-synchronizes (second flop, clean signal)
3. The comparison `cl2 == cl1` ensures the value has been stable for at least one `clk` period before passing it to the processing chain

> [!WARNING]
> The stability check means the core audio can be delayed by up to 2 clock cycles. For a 24.576 MHz clock, this is ~81 ns — well within the 20.8 µs sample period at 48 kHz. However, if the core changes `core_l` rapidly (faster than 12 MHz), the synchronizer may miss transitions.

---

## 4. IIR Filter

Source: `audio_out.v:L179-200`

The IIR (Infinite Impulse Response) filter is a configurable second-order filter used to simulate the analog low-pass characteristics of original hardware. It operates on the sign-converted, synchronized core audio.

### 4.1 Sign Conversion

```verilog
// audio_out.v:L196-197
.input_l({~is_signed ^ cl[15], cl[14:0]}),
.input_r({~is_signed ^ cr[15], cr[14:0]}),
```

The `is_signed` parameter controls the input format:
- `is_signed = 1`: Core outputs signed 16-bit audio (two's complement, −32768 to +32767). The XOR with `1` flips the MSB, converting to offset binary.
- `is_signed = 0`: Core outputs unsigned audio (0 to 65535). The XOR with `0` leaves the MSB intact.

The result is always **offset binary** (0x0000 = most negative, 0x8000 = zero, 0xFFFF = most positive), which the IIR filter processes uniformly.

### 4.2 Filter Architecture

The `IIR_filter` module is parameterized with `use_params=0`, meaning it uses the external coefficient ports rather than internal hardcoded values. The coefficients are:

| Port | Width | Purpose |
|---|---|---|
| `cx` | 40 bits | Combined x-coefficient data |
| `cx0`, `cx1`, `cx2` | 8 bits each | X-coefficient scale/shift factors |
| `cy0`, `cy1`, `cy2` | 24 bits each | Y-coefficient (feedback) terms |

The filter runs on `flt_ce & a_en1` — it only processes when both the filter clock enable is asserted and the startup delay has completed (see [§8](#8-startup-muting-sequence)).

### 4.3 Configuration from HPS

The HPS configures the filter coefficients through the SPI status word. Core developers can specify custom filter profiles in their CONF_STR to simulate specific hardware characteristics (e.g., the NES's characteristic low-pass, or the C64's SID filter curve).

---

## 5. DC Blocker

Source: `audio_out.v:L202-222`

Two instances of `DC_blocker` remove DC offset from the filtered audio:

```verilog
// audio_out.v:L202-211
wire [15:0] adl;
DC_blocker dcb_l (
    .clk(clk),
    .ce(sample_ce),
    .sample_rate(sample_rate),
    .mute(~a_en2),     // mute during startup
    .din(acl),
    .dout(adl)
);
```

The `mute` input is tied to `~a_en2` — the blocker outputs silence until the startup delay completes. This prevents pops and clicks during core loading.

The DC blocker is a first-order high-pass IIR filter with a very low cutoff frequency (~1 Hz). It removes the constant DC offset that some cores produce (where the audio oscillates between 0 and +MAX instead of −MAX/2 to +MAX/2). Without DC blocking, the mix with ALSA audio (which is centered at zero) would produce asymmetric clipping.

---

## 6. Mixer and Attenuator (`aud_mix_top`)

Source: `audio_out.v:L258-296`

The mixer combines core audio with Linux ALSA audio, applies stereo crossfeed, attenuates the volume, and clamps the output to prevent overflow.

### 6.1 Architecture

```verilog
// audio_out.v:L258-296
module aud_mix_top (
    input             clk,
    input             ce,
    input       [4:0] att,        // volume attenuation
    input       [1:0] mix,        // crossfeed mode
    input      [15:0] core_audio, // DC-blocked core audio
    input      [15:0] linux_audio,// ALSA audio
    input      [15:0] pre_in,     // crossfeed from opposite channel
    output reg [15:0] pre_out = 0,// output before crossfeed (fed to other channel)
    output reg [15:0] out = 0     // final output
);
```

The left and right mixers are cross-connected: the left mixer's `pre_out` feeds the right mixer's `pre_in`, and vice versa. This enables stereo crossfeed.

### 6.2 Mixing Math

```verilog
// audio_out.v:L274-293
reg signed [16:0] a1, a2, a3, a4;
always @(posedge clk) if (ce) begin

    // Step 1: Sign-extend core audio to 17 bits
    a1 <= {core_audio[15], core_audio};

    // Step 2: Add Linux audio (sign-extended)
    a2 <= a1 + {linux_audio[15], linux_audio};

    // Step 3: Crossfeed output (before crossfeed processing)
    pre_out <= a2[16:1];   // divide by 2 to prevent overflow

    // Step 4: Apply crossfeed mode
    case(mix)
        0: a3 <= a2;                                                    // Hard stereo
        1: a3 <= $signed(a2) - $signed(a2[16:3]) + $signed(pre_in[15:2]);  // 12.5% crossfeed
        2: a3 <= $signed(a2) - $signed(a2[16:2]) + $signed(pre_in[15:1]);  // 25% crossfeed
        3: a3 <= {a2[16], a2[16:1]} + {pre_in[15], pre_in};            // Full mono
    endcase

    // Step 5: Attenuation
    if(att[4]) a4 <= 0;            // att >= 16: mute
    else a4 <= a3 >>> att[3:0];    // arithmetic right shift by att[3:0]

    // Step 6: Clamping (overflow protection)
    out <= ^a4[16:15] ? {a4[16], {15{a4[15]}}} : a4[15:0];
end
```

### 6.3 Crossfeed Modes

| `mix` | Mode | Formula | Description |
|---|---|---|---|
| 0 | Hard stereo | `a3 = a2` | No mixing between channels |
| 1 | Light crossfeed | `a3 = a2 − a2/8 + pre_in/4` | ~12.5% opposite channel |
| 2 | Medium crossfeed | `a3 = a2 − a2/4 + pre_in/2` | ~25% opposite channel |
| 3 | Full mono | `a3 = a2/2 + pre_in` | Equal L+R mix |

The crossfeed is implemented using **shift-and-add** arithmetic — no DSP multipliers are used. The subtraction of the local channel's contribution (`a2[16:3]` etc.) compensates for the added opposite channel, preventing the total energy from increasing.

### 6.4 Attenuation

The `att[4:0]` input provides 32 volume levels:
- `att = 0`: No attenuation (full volume)
- `att = 1–15`: Arithmetic right shift by 1–15 bits (−6 dB to −90 dB in 6 dB steps)
- `att ≥ 16`: Hard mute (`a4 = 0`)

The arithmetic right shift (`>>>`) preserves the sign bit, ensuring proper attenuation of negative values.

### 6.5 Clamping

```verilog
out <= ^a4[16:15] ? {a4[16], {15{a4[15]}}} : a4[15:0];
```

The XOR of bits 16 and 15 detects overflow (when the sign bits disagree). On overflow:
- Positive overflow → `0111_1111_1111_1111` (+32767)
- Negative overflow → `1000_0000_0000_0000` (−32768)

This is a standard saturating clamp that prevents wraparound distortion.

---

## 7. Output Interfaces

### 7.1 I2S (Digital Audio to HDMI)

Source: `audio_out.v:L75-88`

```verilog
i2s i2s (
    .reset(reset),
    .clk(clk),
    .ce(i2s_ce),          // bit clock enable
    .sclk(i2s_bclk),      // serial clock output
    .lrclk(i2s_lrclk),    // left/right clock output
    .sdata(i2s_data),     // serialized data output
    .left_chan(al),        // left sample
    .right_chan(ar)        // right sample
);
```

The `i2s.v` module takes parallel 16-bit words and serializes them:
- **Bit clock**: Generated from `i2s_ce` (half the `mclk_ce` rate)
- **LR clock**: Alternates every 32 bit clocks (one sample per channel)
- **Data**: MSB-first, synchronized to the falling edge of `sclk`

At 48 kHz: bit clock = 3.072 MHz (48k × 32 × 2). At 96 kHz: 6.144 MHz.

### 7.2 SPDIF (TOSLINK Optical)

Source: `audio_out.v:L90-99`

```verilog
spdif toslink (
    .rst_i(reset),
    .clk_i(clk),
    .bit_out_en_i(mclk_ce),     // bit-level clock enable
    .sample_i({ar, al}),         // stereo sample packed MSB-first
    .spdif_o(spdif)              // BMC-encoded output
);
```

The SPDIF module packs 16-bit samples into 32-bit subframes with preambles, validity bits, user bits, channel status, and parity. The output is BMC (Biphase Mark Code) encoded — self-clocking and DC-free.

### 7.3 Sigma-Delta DAC (Analog 3.5mm)

Source: `audio_out.v:L101-115`

```verilog
sigma_delta_dac #(15) sd_l (
    .CLK(clk),
    .RESET(reset),
    .DACin({~al[15], al[14:0]}),  // convert signed to offset binary
    .DACout(dac_l)
);

sigma_delta_dac #(15) sd_r (
    .CLK(clk),
    .RESET(reset),
    .DACin({~ar[15], ar[14:0]}),
    .DACout(dac_r)
);
```

Key observations:
1. **15-bit input** — The `#(15)` parameter reduces the input from 16 to 15 bits. This limits the effective SNR to ~90 dB (15 × 6 dB/bit) instead of the theoretical 96 dB of 16-bit audio.
2. **Sign conversion** — `{~al[15], al[14:0]}` converts signed two's complement to offset binary (0x0000 = most negative, 0x7FFF = zero, 0xFFFF = most positive).
3. **First-order modulator** — The `sigma_delta_dac` module is a first-order ΣΔ modulator that pushes quantization noise into high frequencies (noise shaping). The external RC low-pass filter on the IO board removes this high-frequency noise.

> [!NOTE]
> The analog DAC output quality is limited by the 15-bit depth and the first-order noise shaping. For the highest audio quality, use the HDMI (I2S) or TOSLINK (SPDIF) digital outputs instead.

---

## 8. Startup Muting Sequence

Source: `audio_out.v:L155-177`

The pipeline includes a two-stage startup delay that prevents pops and clicks during core loading:

```verilog
// audio_out.v:L155-177
reg a_en1 = 0, a_en2 = 0;
always @(posedge clk, posedge reset) begin
    reg  [1:0] dly1 = 0;
    reg [14:0] dly2 = 0;

    if(reset) begin
        dly1 <= 0; dly2 <= 0; a_en1 <= 0; a_en2 <= 0;
    end
    else begin
        if(flt_ce) begin
            if(~&dly1) dly1 <= dly1 + 1'd1;  // count up to 3
            else a_en1 <= 1;                   // IIR filter enable after 3 flt_ce cycles
        end

        if(sample_ce) begin
            if(!dly2[13+sample_rate]) dly2 <= dly2 + 1'd1;
            else a_en2 <= 1;                   // DC blocker enable after delay
        end
    end
end
```

| Stage | Signal | Delay | Purpose |
|---|---|---|---|
| 1 | `a_en1` | 3 `flt_ce` cycles | Wait for IIR filter to settle before enabling output |
| 2 | `a_en2` | ~8192–16384 sample clocks | Wait for DC blocker to converge before un-muting |

The DC blocker's `mute` input is tied to `~a_en2`, so it outputs silence until the startup delay expires. This eliminates the initial DC offset transient that would otherwise produce an audible pop.

---

## 9. Antipatterns and Common Pitfalls

### 9.1 Unsigned vs Signed Audio Confusion

The `is_signed` parameter must correctly reflect the core's output format. If a core outputs unsigned audio but `is_signed=1`, the sign conversion will produce severely distorted audio (the MSB is incorrectly flipped).

### 9.2 Overdriving the Mixer

If both core audio and ALSA audio are at full scale (0x7FFF), their sum exceeds the 17-bit range. The `pre_out` division by 2 prevents immediate overflow, but with crossfeed enabled, the signal can still clip. The clamping circuit prevents wraparound but causes distortion. Core developers should ensure their audio output stays below ~75% of full scale to leave headroom for ALSA mixing.

### 9.3 Sigma-Delta DAC Resolution Loss

The 15-bit input to the DAC discards the LSB of the 16-bit audio. For most retro applications this is imperceptible (~90 dB SNR), but for 16-bit audio purists, the digital outputs (HDMI/TOSLINK) should be preferred.

### 9.4 Filter Rate Mismatch

The `flt_rate` parameter controls the IIR filter's processing rate independently of the output sample rate. If `flt_rate` is set too low, the filter won't process samples fast enough and the output will be aliased or distorted. If set too high, the filter processes more frequently than necessary, wasting FPGA resources.

---

## 10. Platform Context

| Aspect | MiSTer | Software Emulation | Analogue Pocket |
|---|---|---|---|
| Clock generation | Fractional accumulator (jitter-free) | N/A (software resampling) | Similar clock-enable approach |
| Filter | Configurable IIR with HPS-set coefficients | FIR/IIR in software | Fixed filter |
| ALSA mixing | Hardware mixer in FPGA | Software mixer (ALSA/PulseAudio) | Not applicable |
| DAC | 1st-order ΣΔ, 15-bit | Host sound card | I2S to built-in speaker |
| Digital output | I2S + SPDIF | Host digital out | I2S only |
| Analog output | ΣΔ DAC via IO board RC filter | Host analog out | Built-in speaker |

---

Source: `Template_MiSTer/sys/audio_out.v` (297 lines), `Template_MiSTer/sys/i2s.v`, `Template_MiSTer/sys/spdif.v`, `Template_MiSTer/sys/sigma_delta_dac.v`
