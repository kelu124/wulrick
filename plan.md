# Wulrick: pic0rick vs WULPUS — Comparison Research Plan

## Overview

This repo tracks a structured comparison between two open-source ultrasound platforms:

- **pic0rick** — RP2040-based pulse-echo acquisition board by kelu124 (kelu124/pic0rick)
- **WULPUS** — Wearable Ultra Low-Power Ultrasound probe by ETH Zurich IIS/PULP group (pulp-bio/wulpus)

The goal is not to find common ground but to map the **strengths, weaknesses, and specificities** of each design. The two devices solve different problems with different constraints — understanding each on its own terms is more valuable than forcing a head-to-head comparison on a single axis.

---

## Phase 1: Independent Technical Design Analysis

This phase treats each device as an independent design artifact. The goal is to understand the architecture, component choices, and design philosophy from the schematics and PCB files — before any performance or use-case framing.

### 1.1 Schematic Review

**pic0rick:**
- [ ] Obtain and read the pic0rick main board schematic (kelu124/pic0rick repo)
- [ ] Obtain and read the pulser PMOD board schematic
- [ ] Obtain and read the HV supply board schematic
- [ ] Reconstruct the full signal chain: HV pulser → transducer → TGC amp → ADC → RP2040

**WULPUS:**
- [ ] Obtain and read the Acquisition PCB schematic (pulp-bio/wulpus repo, Altium files)
- [ ] Obtain and read the High-Voltage PCB schematic
- [ ] Reconstruct the full signal chain: HV multiplexer → MOSFET driver → transducer → amp → MSP430 ADC

### 1.2 Component Analysis

For each key component in both designs, document: what it does, why it was likely chosen over alternatives, and what constraints it imposes.

**pic0rick key components:**
- [ ] RP2040 — programmable logic via PIO, cost, ecosystem
- [ ] AD8331 — variable gain amp (TGC), gain range, bandwidth, noise figure
- [ ] MCP4812 DAC — SPI-controlled gain curve generation
- [ ] MD1210 + TC6320 — three-level pulser topology
- [ ] ADC chip (identify from schematic) — 60 Msps, 10-bit

**WULPUS key components:**
- [ ] MSP430FR5043 — TI ultrasound MCU with dedicated USS peripheral
- [ ] nRF52832 — BLE SoC, role in the dual-MCU architecture
- [ ] HV multiplexer (identify part) — 8-channel TX/RX switching
- [ ] MOSFET driver — unipolar +15V excitation topology
- [ ] Amplifier chain (identify parts) — fixed 40.8 dB gain

### 1.3 PCB Architecture

**pic0rick:**
- [ ] Document board count and interconnects (main + pulser PMOD + HV board)
- [ ] Identify PMOD expansion header layout and what it exposes
- [ ] Assess PCB complexity: layer count, component density, soldering difficulty

**WULPUS:**
- [ ] Document two-PCB split rationale (Acquisition PCB vs HV PCB)
- [ ] Assess miniaturization design decisions: 46×25 mm target constraints
- [ ] Identify how HV and signal paths are isolated between boards

### 1.4 Architecture Summary

- [ ] Draw or describe the system block diagram for each device
- [ ] Identify what each design explicitly optimizes for (from design choices, not stated goals)
- [ ] Note what each design explicitly trades away

---

## Phase 2: Hardware Performance Characterization

### 2.1 Core Processing

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| MCU | RP2040 (Pico W) | MSP430FR5043 + nRF52832 |
| Ultrasound timing | RP2040 PIO state machines | MSP430 dedicated USS peripheral |
| Form factor | Development board | 46×25 mm, 13g |

**Research tasks:**
- [ ] Compare RP2040 PIO flexibility vs MSP430FR5043 dedicated ultrasound peripheral
- [ ] Evaluate dual-MCU tradeoffs in WULPUS: complexity, power gating, latency

### 2.2 Analog Front-End

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| ADC | 60 Msps, 10-bit | 8 Msps, 12-bit |
| Amplifier | AD8331 TGC, 7.5–55.5 dB, SPI-controlled | Fixed 40.8 dB gain |
| Channels | 1 | 8 time-multiplexed |

**Research tasks:**
- [ ] Quantify axial resolution: 60 Msps vs 8 Msps for typical transducer frequencies
- [ ] Analyze TGC (dynamic gain) benefit for depth compensation vs fixed gain
- [ ] Evaluate WULPUS 8-channel multiplexing: what imaging or sensing modes it enables

### 2.3 Transmit Path

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| Pulser topology | Three-level bipolar (MD1210 + TC6320), separate PMOD | Unipolar +15V programmable |
| HV supply | ±24V, separate board | Integrated HV PCB |

**Research tasks:**
- [ ] Compare bipolar vs unipolar excitation: effect on transducer efficiency and bandwidth
- [ ] HV supply architecture: modular vs integrated — miniaturization and noise tradeoffs

### 2.4 Power and Connectivity

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| Power | USB-tethered | <25 mW, battery, multi-day |
| Data link | USB | BLE 320 kbps + USB dongle |

**Research tasks:**
- [ ] Determine pic0rick USB power draw
- [ ] Benchmark WULPUS BLE data throughput vs raw ADC output rate
- [ ] Understand compression or decimation WULPUS applies to fit 8 Msps into 320 kbps BLE

---

## Phase 3: Software and Firmware Ecosystem

### 3.1 Programming Model

**pic0rick:** MicroPython on RP2040, PIO for timing, pyUn0-lib Python library
**WULPUS:** C firmware for MSP430 + nRF52832 + nRF52840 dongle, Python host tools with GUI

**Research tasks:**
- [ ] Compare customization ease: MicroPython/Python vs C embedded toolchain
- [ ] Evaluate WULPUS GUI for configuration coverage
- [ ] Assess pyUn0-lib documentation and API completeness

### 3.2 Data Pipeline

**Research tasks:**
- [ ] Trace pic0rick: RP2040 ADC → USB → host PC
- [ ] Trace WULPUS: MSP430 → nRF52832 BLE → nRF52840 dongle → host PC
- [ ] Identify buffering, decimation, or compression steps in each pipeline
- [ ] Measure/estimate end-to-end latency for real-time display

### 3.3 Signal Processing

**Research tasks:**
- [ ] Review kelu124/us_rf_processing: B-mode reconstruction pipeline
- [ ] Review WULPUS Python tools: carotid/muscle signal processing
- [ ] Identify beamforming support in each (or absence thereof)

---

## Phase 4: Strengths, Weaknesses, and Specificities

The goal of this phase is not to find which device is "better" but to clearly articulate what each is genuinely good at, where it has real limitations, and what makes it distinctive.

### 4.1 pic0rick — Strengths

- [ ] High sampling rate (60 Msps) for fine axial resolution and high-frequency transducers
- [ ] Fully programmable gain curve via TGC — good for varying imaging depths
- [ ] Modular PMOD architecture — extensible for custom experiments
- [ ] MicroPython / Python ecosystem — low barrier to scripting and automation
- [ ] Open hardware certified (OSHWA FR000023), available pre-built on Tindie
- [ ] Part of established un0rick ecosystem with prior published work

### 4.2 pic0rick — Weaknesses and Limits

- [ ] USB-tethered: no battery operation, no wireless
- [ ] Single channel: no multi-element array imaging without additional hardware
- [ ] Power unknown: not characterized for low-power contexts
- [ ] Not certified for medical use

### 4.3 pic0rick — Specificities

- [ ] PIO-based ultrasound timing: unusual approach — assess flexibility and constraints
- [ ] Three-level bipolar pulser: assess why this topology and what it implies for transducer choice

### 4.4 WULPUS — Strengths

- [ ] Ultra-low power (<25 mW): genuine multi-day wearable operation
- [ ] 8 time-multiplexed channels: multi-site or multi-element sensing
- [ ] BLE wireless: untethered real-time monitoring
- [ ] Compact (13g, 46×25 mm): body-worn deployment
- [ ] Published results on carotid, muscle, gesture, EMG coupling

### 4.5 WULPUS — Weaknesses and Limits

- [ ] 8 Msps ADC: limits usable transducer frequency and axial resolution
- [ ] Fixed gain: no depth compensation without software post-processing
- [ ] BLE bandwidth (320 kbps): constrains raw data throughput
- [ ] C toolchain: steeper embedded development barrier
- [ ] Altium PCB files: replication requires expensive EDA tool (or export workarounds)

### 4.6 WULPUS — Specificities

- [ ] MSP430FR5043 USS peripheral: dedicated ultrasound hardware on-chip — assess what this enables vs locks in
- [ ] Dual-MCU split (ultrasound + BLE): assess power gating strategy and synchronization

### 4.7 Published Benchmarks

- [ ] Collect pic0rick/un0rick published results (Zenodo paper, Hackster)
- [ ] Collect WULPUS published results (IEEE UFFC 2022, IEEEXplore)
- [ ] Extract any quantitative figures: SNR, dynamic range, imaging depth, resolution

---

## Phase 5: Openness, Community, and Cost

### 5.1 Open-Source Posture

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| License | Open hardware (OSHWA FR000023) | Permissive open-source |
| PCB tool | KiCad (free) | Altium (proprietary) |
| Firmware | MicroPython, open | C, open |
| BOM | Yes | Yes |

**Research tasks:**
- [ ] Verify pic0rick PCB tool (KiCad vs other)
- [ ] Identify Altium export formats available in WULPUS repo (Gerbers, PDF, etc.)
- [ ] Assess which is easier to reproduce from scratch

### 5.2 Cost

**Research tasks:**
- [ ] Confirm pic0rick kit cost from Tindie; estimate component BOM separately
- [ ] Estimate WULPUS BOM cost from shared files and PCBWay listing
- [ ] Compare total cost including development tools (EDA software, programmer, etc.)

### 5.3 Community and Maintenance

**Research tasks:**
- [ ] GitHub activity audit: stars, forks, open issues, last commit date for both repos
- [ ] Find community channels (Discord, forums, mailing lists)
- [ ] Assess maintainer responsiveness on issues

---

## Phase 6: Synthesis

**Deliverables:**
- [ ] Master comparison table across all dimensions from Phases 1–5
- [ ] Strengths/weaknesses/specificities summary card for each device
- [ ] Identification of design decisions that most differentiate the two platforms
- [ ] Gap analysis: what neither device covers well
- [ ] Notes on potential cross-pollination (e.g., WULPUS multi-channel approach applied to a pic0rick-style system)

---

## Key References

- pic0rick main repo: https://github.com/kelu124/pic0rick
- pic0rick product page: https://un0rick.cc/pic0rick
- pic0rick Zenodo paper: https://zenodo.org/records/10968504
- pic0rick Tindie: https://www.tindie.com/products/kelu124/pic0rick-a-pico-ultrasound-pulse-echo-system/
- WULPUS main repo: https://github.com/pulp-bio/wulpus
- WULPUS IEEE paper: https://ieeexplore.ieee.org/document/9958156/
- WULPUS IEEE UFFC post: https://ieee-uffc.org/post/news/wulpus-open-source-wearable-ultra-low-power-ultrasound-probe
