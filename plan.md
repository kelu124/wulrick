# Wulrick: pic0rick vs WULPUS — Comparison Research Plan

## Overview

This repo tracks a structured comparison between two open-source ultrasound platforms:

- **pic0rick** — RP2040-based pulse-echo acquisition board by kelu124 (kelu124/pic0rick)
- **WULPUS** — Wearable Ultra Low-Power Ultrasound probe by ETH Zurich IIS/PULP group (pulp-bio/wulpus)

Despite both being open-source ultrasound platforms, they target very different use cases and design philosophies. This research plan lays out a systematic approach to understanding the tradeoffs.

---

## Phase 1: Hardware Architecture Deep-Dive

### 1.1 Core Processing Units

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| MCU | RP2040 (Raspberry Pi Pico W) | MSP430FR5043 (ultrasound MCU) + nRF52832 (BLE MCU) |
| FPGA | None | None |
| Form factor | Development board | 46×25 mm wearable, 13g |

**Research tasks:**
- [ ] Review RP2040 PIO capabilities used for ultrasound timing in pic0rick
- [ ] Understand MSP430FR5043's built-in ultrasound features vs general-purpose RP2040 PIO
- [ ] Compare dual-MCU vs single-MCU architecture tradeoffs (WULPUS: MSP430 + nRF52832)

### 1.2 Analog Front-End

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| ADC | 60 Msps, 10-bit | 8 Msps, 12-bit |
| Amplifier | AD8331 TGC (7.5–55.5 dB), SPI-controlled via MCP4812 DAC | 40.8 dB fixed gain |
| Channels | Single channel | 8 time-multiplexed channels |

**Research tasks:**
- [ ] Quantify impact of 60 Msps vs 8 Msps on axial resolution
- [ ] Analyze AD8331 TGC gain curve vs WULPUS fixed gain for different tissue depths
- [ ] Evaluate 8-channel multiplexing in WULPUS: what imaging modes does it enable?

### 1.3 Transmit Path

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| Pulser | MD1210 + TC6320, three-level, separate PMOD board | +15V unipolar programmable excitation |
| HV supply | ±24V (separate HV board) | Integrated on High-Voltage PCB |

**Research tasks:**
- [ ] Compare three-level bipolar pulser (pic0rick) vs unipolar excitation (WULPUS) — effect on transducer bandwidth
- [ ] HV integration: modular (pic0rick) vs integrated (WULPUS) — implications for miniaturization

### 1.4 Power and Form Factor

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| Power consumption | Not specified (USB-powered) | <25 mW |
| Battery operation | No (USB) | Yes, multi-day |
| Connectivity | USB | BLE (320 kbps), USB dongle |

**Research tasks:**
- [ ] Benchmark WULPUS battery life at different acquisition rates
- [ ] Determine pic0rick power draw from USB to understand the gap

---

## Phase 2: Software and Firmware Ecosystem

### 2.1 Programming Model

**pic0rick:**
- MicroPython on RP2040
- PIO state machines for precise pulse-echo timing
- pyUn0-lib Python library

**WULPUS:**
- C firmware for MSP430FR5043
- C firmware for nRF52832 (BLE)
- C firmware for nRF52840 USB dongle
- Python host-side tools with GUI

**Research tasks:**
- [ ] Compare ease of customization: MicroPython/Python vs C toolchain
- [ ] Evaluate WULPUS GUI capabilities vs pic0rick scripted workflows
- [ ] Assess community library ecosystem for each platform

### 2.2 Data Pipeline

**Research tasks:**
- [ ] Trace pic0rick data path: RP2040 ADC → USB → host PC processing
- [ ] Trace WULPUS data path: MSP430 ADC → nRF52832 → BLE → nRF52840 dongle → host PC
- [ ] Compare achievable data throughput: 60 Msps × 10-bit USB vs 8 Msps × 12-bit at 320 kbps BLE
- [ ] Evaluate latency for real-time display in both systems

### 2.3 Signal Processing

**Research tasks:**
- [ ] Review kelu124's us_rf_processing for B-mode reconstruction from pic0rick data
- [ ] Review WULPUS Python tools for carotid/muscle monitoring signal processing
- [ ] Identify gaps: does either platform have beamforming support?

---

## Phase 3: Use Case and Application Analysis

### 3.1 Intended Use Cases

| Use case | pic0rick | WULPUS |
|----------|----------|--------|
| NDT / material testing | Primary target | Not designed for |
| Biomedical monitoring | Possible but not primary | Primary target |
| B-mode imaging | Supported | Not primary |
| Real-time wearable | No | Yes |
| Educational/pedagogical | Primary | Secondary |

**Research tasks:**
- [ ] Identify overlap scenarios where both could be used
- [ ] Map frequency range compatibility with common transducers (medical vs NDT)
- [ ] Assess pic0rick for biomedical use: sensitivity, noise floor, regulatory status
- [ ] Assess WULPUS for NDT use: sampling rate limits for high-frequency transducers

### 3.2 Published Results and Benchmarks

**Research tasks:**
- [ ] Collect pic0rick/un0rick published experiments (Zenodo paper, Hackster demos)
- [ ] Collect WULPUS published results (IEEE UFFC 2022, IEEEXplore, ResearchGate papers)
- [ ] Compare SNR, imaging depth, resolution figures where reported

---

## Phase 4: Openness, Community, and Cost

### 4.1 Open-Source Posture

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| License | Open hardware (OSHWA FR000023) | Permissive open-source |
| PCB files | Yes (KiCad/similar) | Yes (Altium) |
| Firmware | MicroPython open | C, open |
| BOM | Yes | Yes |

**Research tasks:**
- [ ] Compare PCB tool accessibility: KiCad (free) vs Altium (expensive) for replication
- [ ] Identify which platform is easier to reproduce from scratch

### 4.2 Cost and Accessibility

**Research tasks:**
- [ ] Confirm pic0rick cost: listed as <$500; verify actual BOM cost
- [ ] Estimate WULPUS BOM cost from shared files
- [ ] Compare availability: pic0rick sold on Tindie vs WULPUS DIY-only

### 4.3 Community and Support

**Research tasks:**
- [ ] Compare GitHub stars, forks, issue activity for both repos
- [ ] Find forums, Discord, or mailing lists for each project
- [ ] Identify whether either project has active maintainers responding to issues

---

## Phase 5: Synthesis and Comparative Summary

**Deliverables:**
- [ ] Side-by-side comparison table (all dimensions)
- [ ] Decision guide: "choose pic0rick if… / choose WULPUS if…"
- [ ] Identification of hybrid approach possibilities (e.g., WULPUS form factor + pic0rick analog front-end)
- [ ] Gap analysis: what neither platform currently covers well

---

## Key References

- pic0rick main repo: https://github.com/kelu124/pic0rick
- pic0rick product page: https://un0rick.cc/pic0rick
- pic0rick Zenodo paper: https://zenodo.org/records/10968504
- pic0rick Tindie: https://www.tindie.com/products/kelu124/pic0rick-a-pico-ultrasound-pulse-echo-system/
- WULPUS main repo: https://github.com/pulp-bio/wulpus
- WULPUS IEEE paper: https://ieeexplore.ieee.org/document/9958156/
- WULPUS IEEE UFFC post: https://ieee-uffc.org/post/news/wulpus-open-source-wearable-ultra-low-power-ultrasound-probe
