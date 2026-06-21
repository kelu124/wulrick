# Wulrick: pic0rick vs WULPUS — Comparison Research Plan

## Overview

This repo tracks a structured comparison between two open-source ultrasound platforms:

- **pic0rick** — RP2040-based pulse-echo acquisition board by kelu124 (kelu124/pic0rick)
- **WULPUS** — Wearable Ultra Low-Power Ultrasound probe by ETH Zurich IIS/PULP group (pulp-bio/wulpus)

The goal is to map the **strengths, weaknesses, and specificities** of each design — not to rank one above the other. The two devices solve different problems under different constraints; understanding each on its own terms is the primary objective.

### Methodology

All analysis is derived from: schematics and PCB files in the respective GitHub repositories, component datasheets, and published papers/documentation. No physical hardware is available; performance figures come from datasheet specs and published measurements only.

### Output Format

Each phase produces a mix of **markdown prose** (design decision rationale, qualitative analysis) and **comparison tables** (spec-by-spec, block-by-block). The final synthesis consolidates these into a master document.

---

## Phase 1: Independent Technical Design Analysis — by Functional Block

This phase compares the two devices function by function, using schematics and datasheets. For each functional block: identify the components used in each design, understand the design decision (why this approach), and note what it implies for performance, flexibility, and constraints.

### 1.1 Pulser (Transmit Excitation)

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Topology | Three-level bipolar (MD1210 + TC6320) | Unipolar +15V (MOSFET driver) |
| Voltage | ±24V | +15V |
| Programmability | Fixed pulse width via RP2040 PIO | Programmable excitation via MSP430 USS |

**Research tasks:**
- [ ] Read MD1210 and TC6320 datasheets — understand three-level pulse generation
- [ ] Identify WULPUS MOSFET driver part from schematic — read its datasheet
- [ ] Compare bipolar vs unipolar excitation: bandwidth, transducer efficiency, ringing
- [ ] Assess voltage level tradeoffs: ±24V vs +15V — penetration depth implications
- [ ] Document design decision rationale for each approach

### 1.2 TX/RX Switch and Multiplexing

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| TX/RX isolation | HV protection on receive path (identified in main board) | Dedicated TX/RX switch on HV PCB |
| Channel count | 1 | 8 (time-multiplexed HV mux) |
| Mux component | N/A | HV multiplexer (identify part from schematic) |

**Research tasks:**
- [ ] Identify pic0rick RX protection components from schematic
- [ ] Identify WULPUS HV multiplexer part — read datasheet
- [ ] Identify WULPUS TX/RX switch — read datasheet
- [ ] Compare single-channel vs 8-channel multiplexed approach: what imaging/sensing modes each enables
- [ ] Analyze switching speed and implications for pulse repetition frequency

### 1.3 Time-Gain Compensation / Amplifier

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Amplifier | AD8331 (VGA/TGC) | Fixed-gain amplifier (identify part) |
| Gain range | 7.5–55.5 dB, continuously variable | 40.8 dB fixed |
| Gain control | SPI via MCP4812 DAC | None (fixed) |

**Research tasks:**
- [ ] Read AD8331 datasheet: noise figure, bandwidth, gain curve, input impedance
- [ ] Identify WULPUS amplifier part from schematic — read datasheet
- [ ] Analyze MCP4812 DAC role in pic0rick gain curve shaping
- [ ] Compare TGC (dynamic) vs fixed gain: depth compensation, SNR at depth, simplicity tradeoff
- [ ] Assess whether WULPUS fixed gain is appropriate given its target transducer frequencies and imaging depths

### 1.4 Analog-to-Digital Conversion

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| ADC | External (identify part), 60 Msps, 10-bit | Integrated in MSP430FR5043, 8 Msps, 12-bit |
| Interface to MCU | Parallel/SPI (verify from schematic) | Internal (on-chip) |
| Effective resolution | 10-bit at 60 Msps | 12-bit at 8 Msps |

**Research tasks:**
- [ ] Identify pic0rick ADC part from schematic — read datasheet (bandwidth, ENOB, interface)
- [ ] Read MSP430FR5043 datasheet: USS ADC specs, ENOB, integrated anti-aliasing
- [ ] Calculate theoretical axial resolution for each: c/(2 × fs) for typical tissue velocity
- [ ] Compare 10-bit @ 60 Msps vs 12-bit @ 8 Msps: dynamic range vs temporal resolution tradeoff
- [ ] Assess anti-aliasing filter requirements and whether each design addresses them

### 1.5 Power Supply

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| HV generation | Separate HV board (±24V) | Integrated HV PCB |
| Logic supply | USB 5V → onboard regulation | Battery → onboard regulation |
| Total consumption | Unknown (USB-tethered, uncharacterized) | <25 mW |

**Research tasks:**
- [ ] Identify pic0rick HV generation topology from HV board schematic (boost, flyback, charge pump?)
- [ ] Identify WULPUS HV generation topology from HV PCB schematic
- [ ] Read relevant power IC datasheets for both
- [ ] Analyze HV modular (pic0rick) vs integrated (WULPUS) tradeoffs: noise coupling, assembly complexity, flexibility
- [ ] Characterize WULPUS power budget from published figures and datasheet quiescent currents
- [ ] Estimate pic0rick power from RP2040, ADC, and AD8331 datasheet figures

### 1.6 Digital Control and Timing

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Timing engine | RP2040 PIO state machines | MSP430FR5043 USS peripheral |
| MCU ecosystem | General-purpose (MicroPython, C/C++) | Ultrasound-specific (TI USS library) |
| Wireless | None | nRF52832 BLE SoC |

**Research tasks:**
- [ ] Read RP2040 PIO documentation: flexibility, timing resolution, limitations for ultrasound
- [ ] Read MSP430FR5043 USS peripheral documentation: dedicated ultrasound sequencer, what it automates
- [ ] Compare general-purpose PIO flexibility vs dedicated USS peripheral efficiency
- [ ] Read nRF52832 datasheet: BLE throughput, power consumption, coexistence with MSP430
- [ ] Analyze WULPUS dual-MCU split: why separate BLE MCU, power gating strategy

### 1.7 Connectivity and Data Output

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Host interface | USB (via RP2040) | BLE 320 kbps (nRF52832) + nRF52840 USB dongle |
| Raw data rate | ~600 Mbps theoretical (USB 2.0), decimated in practice | 320 kbps BLE |
| Tethered | Yes | No |

**Research tasks:**
- [ ] Determine actual USB throughput used by pic0rick (from firmware/examples)
- [ ] Determine WULPUS data compression or decimation to fit 8 Msps into 320 kbps BLE
- [ ] Compare data pipeline latency: USB vs BLE for real-time display
- [ ] Assess implications of BLE bandwidth constraint on what WULPUS can transmit in real time

---

## Phase 2: Hardware Performance Characterization

Using datasheet specs and published measurements (no hardware testing).

**Research tasks:**
- [ ] Compile key performance figures for each device in a single comparison table
- [ ] Identify figures available from datasheets vs figures only available from published papers
- [ ] Flag any specs that are uncharacterized / not publicly available
- [ ] Collect SNR, dynamic range, imaging depth, axial/lateral resolution from published papers

---

## Phase 3: Software and Firmware Ecosystem

### 3.1 Programming Model

**pic0rick:** MicroPython on RP2040, PIO for timing, pyUn0-lib Python library
**WULPUS:** C firmware for MSP430 + nRF52832 + nRF52840 dongle; Python host tools with GUI

**Research tasks:**
- [ ] Survey pyUn0-lib: API coverage, documentation quality, example scripts
- [ ] Survey WULPUS firmware repo structure: how to configure acquisitions, modify timing
- [ ] Compare customization ease: MicroPython/Python scripting vs C embedded toolchain
- [ ] Evaluate WULPUS Python GUI: what parameters are configurable, what's fixed
- [ ] Assess signal processing tools: kelu124/us_rf_processing (B-mode), WULPUS Python tools

### 3.2 Data Pipeline

**Research tasks:**
- [ ] Trace pic0rick pipeline: RP2040 ADC capture → USB → host PC
- [ ] Trace WULPUS pipeline: MSP430 ADC → nRF52832 BLE → dongle → host PC
- [ ] Identify buffering, decimation, and framing in each pipeline from firmware source

---

## Phase 4: Strengths, Weaknesses, and Specificities

A per-device assessment — not comparative ranking but clear-eyed characterization.

### 4.1 pic0rick

**Strengths:**
- High sampling rate (60 Msps) → fine axial resolution, compatibility with high-frequency transducers
- Variable TGC (7.5–55.5 dB) → depth compensation, flexible imaging scenarios
- PMOD modular architecture → extensible for custom experiments
- MicroPython/Python → low barrier to scripting and rapid prototyping
- OSHWA certified, available pre-built on Tindie
- Part of mature un0rick ecosystem with prior published work

**Weaknesses and limits (to verify/quantify):**
- [ ] USB-tethered: no wireless or battery operation
- [ ] Single channel: no simultaneous multi-element acquisition
- [ ] Power consumption uncharacterized
- [ ] Not a medical-grade device

**Specificities to investigate:**
- [ ] PIO-based ultrasound timing: flexibility ceiling and resolution limits
- [ ] Three-level bipolar pulser: transducer compatibility implications

### 4.2 WULPUS

**Strengths:**
- Ultra-low power (<25 mW) → genuine multi-day wearable operation
- 8 time-multiplexed channels → multi-site sensing, probe array support
- BLE wireless → untethered real-time monitoring
- 13g, 46×25 mm → body-worn deployment
- Published biomedical results (carotid, muscle, gesture, EMG coupling)

**Weaknesses and limits (to verify/quantify):**
- [ ] 8 Msps ADC: limits usable transducer frequency range and axial resolution
- [ ] Fixed gain: no depth compensation
- [ ] BLE 320 kbps: raw data throughput ceiling
- [ ] C toolchain: higher embedded development barrier
- [ ] Altium PCB source files: replication requires expensive EDA tool

**Specificities to investigate:**
- [ ] MSP430FR5043 USS peripheral: what it automates, what it constrains
- [ ] Dual-MCU architecture: power gating strategy, synchronization overhead

### 4.3 Published Benchmarks

- [ ] Extract quantitative figures from Zenodo paper (pic0rick/un0rick)
- [ ] Extract quantitative figures from IEEE UFFC 2022 paper (WULPUS)
- [ ] Build comparison table of measured performance figures

---

## Phase 5: Openness, Community, and Cost

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| License | Open hardware (OSHWA FR000023) | Permissive open-source |
| PCB tool | Verify (KiCad?) | Altium (proprietary) |
| BOM | Yes | Yes |
| Pre-built | Yes (Tindie) | DIY only |

**Research tasks:**
- [ ] Verify pic0rick PCB tool; check if Gerbers are exported
- [ ] Check what Altium export formats WULPUS provides (Gerbers, PDFs, step files)
- [ ] Confirm pic0rick kit price (Tindie) and estimate component BOM separately
- [ ] Estimate WULPUS BOM from shared files and PCBWay listing
- [ ] GitHub activity audit: stars, forks, open/closed issues, last commit
- [ ] Find community channels for each project (forums, Discord, issues)

---

## Phase 6: Synthesis

**Deliverables:**
- [ ] Master comparison table (all functional blocks + performance + ecosystem)
- [ ] Per-device strengths/weaknesses/specificities summary (prose + table)
- [ ] Key design decisions that most differentiate the platforms
- [ ] Gap analysis: what neither device covers well
- [ ] Cross-pollination notes: design ideas each could borrow from the other

---

## Phase 7: Array Multiplexers

*Comparing the MAX14866 8-channel HV analog switch (pic0rick PMOD) with the WULPUS HV mux approach. Addresses what each architecture enables and limits for multi-element array imaging.*

---

### 7.1 MAX14866 (pic0rick PMOD)

The MAX14866 is an 8-channel high-voltage analog switch IC used in pic0rick's mux PMOD board.

| Parameter | Value | Notes |
|-----------|-------|-------|
| Manufacturer | Maxim/Analog Devices | Part of the MAX14xxx HV switch family |
| Channel count | 8 | Single IC; cascadable via SPI |
| HV tolerance | ±80V | Protects receive inputs from transmit transients |
| ON resistance (RON) | ~100–250 Ω | Depends on supply voltage; higher than signal-path VGAs |
| Switching speed | <500 ns | Time from SPI command to channel active |
| Control interface | SPI (3-wire: CSB, SDI, SCLK) | 16-bit shift register |
| Supply voltage | Up to ±40V logic + HV supply | Separate HV and logic rails |
| Package | 28-pin TSSOP | Board area ~30 mm² |
| Unit price (est.) | ~$8–12 (qty 1) | Premium for HV tolerance |

**How pic0rick uses it:** The MAX14866 PMOD plugs into pic0rick's PMOD header. The RP2040 controls the mux via SPI, selecting which of the 8 channels is active for TX/RX. This enables sequential single-element firing across up to 8 transducers, or multi-element switching for basic synthetic aperture experiments. The wide ±80V tolerance accommodates pic0rick's ±24V transmit pulses with substantial margin.

**Cascading:** Multiple MAX14866 PMODs can be chained via PMOD daisy-chain or additional SPI chip selects. SPI daisy-chaining two boards yields 16 channels. Each additional board adds 8 channels at the cost of one more SPI transaction per frame. At 16 MHz SPI, four boards (32 channels) can be configured in <10 µs — well within any inter-acquisition window.

---

### 7.2 WULPUS HV Mux (Probable HV2707T-C/R8X or equivalent)

WULPUS's HV PCB uses an SPI-controlled 8-channel HV multiplexer. The part has been provisionally identified as the Supertex/Microchip HV2707T-C/R8X based on functional analysis of the schematic, but has not been confirmed from the Altium BOM (binary format, not directly readable).

| Parameter | Value (HV2707T-C/R8X, est.) | Notes |
|-----------|------------------------------|-------|
| Manufacturer | Supertex / Microchip | Acquired from Supertex 2014 |
| Channel count | 8 | 8:1 multiplexer |
| HV tolerance | Up to ±150V | Designed for medical ultrasound drive voltages |
| ON resistance (RON) | ~100–200 Ω (typical HV mux) | Datasheet-dependent |
| Switching speed | ~100 ns typical | SPI-triggered, fast HV switching |
| Control interface | SPI (16-bit shift register) | 8MHz max SPI clock |
| Supply voltage | Logic: 3.3–5V; HV: up to ±150V | Separate HV rail from logic |
| Package | ~28–32 pin SOIC/SSOP | Depends on specific variant |

*Note: All parameters marked (est.) are inferred from the HV2707 family datasheet and WULPUS schematic context, not directly confirmed.*

**How WULPUS uses it:** The WULPUS HV mux is integrated on the HV PCB alongside the MOSFET pulser and +15V boost circuit. The MSP430FR5043 controls mux channel selection via SPI. During acquisition, the mux switches between TX and RX states — an important distinction: it does not just select between 8 transducers, it also switches the selected transducer's connection from the transmit driver (during the pulse) to the receive amplifier (during echo capture). This TX/RX switching function is a critical role beyond simple channel selection.

**WULPUS mux timing:** The `HV MUX RX start time` parameter (tuned per transducer, as documented in Discussion #20) controls when the mux switches from TX to RX after the excitation pulse ends. This parameter must be tuned to avoid receiving the transmit pulse itself as a false echo while also not delaying so long that near-field echoes are missed.

---

### 7.3 Side-by-Side Comparison

| Parameter | MAX14866 (pic0rick PMOD) | HV2707T/equiv. (WULPUS) |
|-----------|--------------------------|--------------------------|
| Channel count | 8 per IC | 8 per IC |
| HV tolerance | ±80V | ±150V (est.) |
| pic0rick TX voltage | ±24V | N/A |
| WULPUS TX voltage | N/A | +15V |
| TX/RX switching | External (MD0100 switch on main board) | Integrated in mux function |
| SPI interface | 16-bit shift register | 16-bit shift register |
| Max SPI clock | ≥10 MHz | 8 MHz (HV2707) |
| Cascading | Via PMOD daisy-chain | Via SPI shift register chaining |
| Board integration | PMOD (modular, pluggable) | On HV PCB (fixed, integrated) |
| BOM position | Optional expansion | Core subsystem |
| Approximate cost | $8–12 (IC only) | $5–10 (est.) |

---

### 7.4 Architecture Implications

**pic0rick's modular PMOD approach:**

The MAX14866's position as a pluggable PMOD means array capability is **opt-in** — the base pic0rick board is single-channel, and multi-channel is added when needed. This creates design flexibility: users who only need a single transducer avoid the mux cost and complexity. Users who want to experiment with arrays can add the PMOD without redesigning the main board.

The ±80V tolerance provides margin beyond pic0rick's ±24V transmit pulses, protecting the MAX14866 from transients and allowing headroom for future higher-voltage pulser experiments. A PMOD-to-PMOD cascade enables 16, 24, or 32 channels from the same RP2040 SPI bus.

**Constraint:** The MAX14866 does not include integrated TX/RX switching. pic0rick handles T/R isolation via the MD0100N8-G passive limiter on the main board. This architecture means all 8 mux inputs share the same transmit path — only one element fires at a time, but all inputs are exposed to the full transmit pulse during firing. The MD0100 limits the voltage that reaches the receive path, but does not isolate individual channels from each other during transmission.

**WULPUS's integrated HV PCB approach:**

WULPUS's mux is not modular — it is a fixed subsystem on the HV PCB. This means: (a) the mux cannot be replaced without redesigning the PCB, (b) the mux is specifically matched to the MOSFET pulser's voltage and timing requirements, (c) the TX/RX switching and channel selection are co-designed, which reduces the risk of timing mismatches.

The higher HV tolerance (~±150V vs ±80V) is appropriate given that WULPUS's HV PCB can deliver +15V pulses — and WULPUS-Pro targets +30V. The extra HV headroom future-proofs the mux for higher-voltage drive without board redesign.

**Constraint:** Fixed 8-channel count with no modular expansion path in the current design. Expanding to 32 channels requires a full PCB redesign (as discussed in the Vermon adapter analysis). The integrated approach trades flexibility for space efficiency and design coherence.

---

### 7.5 What Each Architecture Enables

| Imaging mode | pic0rick + MAX14866 PMOD | WULPUS + integrated mux |
|--------------|--------------------------|--------------------------|
| Single-element A-mode | ✓ (base, no mux) | ✓ (any channel) |
| 8-ch multiplexed A-mode | ✓ (with PMOD) | ✓ (standard config) |
| 8-ch synthetic aperture | ✓ (with PMOD) | ✓ |
| 16-ch SA (2 PMODs) | ✓ (SPI cascade) | ✗ (requires new PCB) |
| 32-ch SA (4 PMODs) | ✓ (SPI cascade) | ✗ (Vermon adapter, new PCB) |
| B-mode image | ✓ offline (host beamforming) | ✓ offline (host beamforming) |
| Real-time B-mode | △ USB-limited throughput | ✗ BLE bandwidth cap |
| HV replacement with >80V pulser | ✗ Exceeds MAX14866 rating | ✓ (HV2707 tolerates >80V) |
| HV replacement with >150V pulser | N/A | ✗ Would exceed mux rating |

---

### 7.6 Design Decision Summary

**pic0rick MAX14866 choice:** Appropriate for a modular open-hardware platform. The PMOD format makes channel expansion incremental, and ±80V tolerance comfortably exceeds the ±24V pulser. The main limitation is that T/R isolation is handled upstream (MD0100), not within the mux, meaning that per-channel isolation is not available — all channels share the transmit bus.

**WULPUS integrated mux choice:** Appropriate for a fixed, integrated wearable. The co-design of pulser + mux + T/R switching on one PCB achieves the timing coordination needed for 450 nA standby power. The higher HV tolerance reflects a design intent toward future WULPUS-Pro voltages. The limitation is inflexibility — adding channels requires PCB modification.

**For researchers considering channel expansion:** pic0rick's PMOD architecture is the more accessible path to >8 channels. Adding a second MAX14866 PMOD is a firmware change (extended SPI addressing) and a hardware addition, not a board redesign. For WULPUS, 32-channel expansion requires the Vermon adapter work documented in the discussions section.

---

## Key References

- pic0rick main repo: https://github.com/kelu124/pic0rick
- pic0rick product page: https://un0rick.cc/pic0rick
- pic0rick Zenodo paper: https://zenodo.org/records/10968504
- pic0rick Tindie: https://www.tindie.com/products/kelu124/pic0rick-a-pico-ultrasound-pulse-echo-system/
- WULPUS main repo: https://github.com/pulp-bio/wulpus
- WULPUS IEEE paper: https://ieeexplore.ieee.org/document/9958156/
- WULPUS IEEE UFFC post: https://ieee-uffc.org/post/news/wulpus-open-source-wearable-ultra-low-power-ultrasound-probe
