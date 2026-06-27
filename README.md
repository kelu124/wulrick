# wulrick — pic0rick vs WULPUS: Open-Source Ultrasound Platform Comparison

A structured technical comparison of two open-source ultrasound platforms:

- **[pic0rick](https://github.com/kelu124/pic0rick)** — RP2040-based pulse-echo acquisition board by kelu124. OSHWA certified, available pre-built on Tindie, aimed at experimenters and researchers needing raw RF capture.
- **[WULPUS](https://github.com/pulp-bio/wulpus)** — Wearable Ultra Low-Power Ultrasound probe by ETH Zurich IIS/PULP group. 22 mW total system power, BLE-connected, designed for multi-day wearable physiological monitoring.

The goal is to map the **strengths, weaknesses, and specificities** of each design — not to rank one above the other. The two devices solve different problems under different constraints.

All analysis is derived from schematics, PCB files, component datasheets, and published papers. No physical hardware was available; all performance figures come from datasheets and published measurements.

---

## Key Findings

### The Fundamental Tradeoff

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| ADC | ADC10065, 65 Msps, 10-bit (ENOB 9.6) | MSP430 SDHS, 8 Msps, 12-bit |
| Transducer range | 40 kHz – ~32 MHz | 1 – 3 MHz practical |
| Amplifier | AD8331 TGC, 7.5–55.5 dB, 12-bit DAC | Fixed 40.8 dB (PGA + op-amp stage) |
| Pulser | ±24V bipolar (MD1213 + TC6320) | +15V unipolar (MOSFET + boost) |
| Channels | 1 base + 8 via MAX14866 PMOD | 8 time-multiplexed HV mux |
| Connectivity | USB (tethered) | BLE 320 kbps (untethered) |
| Power | ~300–400 mW (USB, uncharacterized) | 22 mW measured |
| Form factor | Standalone board + PMOD expansion | 46×25 mm, 9 g, worn on body |
| PCB tool | KiCad (free) | Altium (~$10k/yr) |
| License | TAPR OHL + GPLv3 (copyleft) | SHL-0.51 + Apache 2.0 (permissive) |
| Pre-built | Tindie (~$200–400) | DIY only |
| Price (DIY BOM) | ~$100–200 | ~$250–500 |

**pic0rick** is the better instrument: higher frequency range, variable TGC, flexible transducer compatibility, easier PCB reproduction, lower barrier to scripting. **WULPUS** is the better wearable: 40× lower power, wireless, body-worn, validated in peer-reviewed publications across seven application papers.

---

### Six Differentiating Design Decisions

1. **ADC approach** — pic0rick uses an external 65 Msps ADC (ADC10065); WULPUS uses the MSP430FR5043's integrated 8 Msps SDHS ADC. The external ADC costs 68.4 mW alone, more than WULPUS's entire 22 mW system. Integration trades frequency range for power.

2. **Gain architecture** — pic0rick's AD8331 TGC with MCP4812 DAC enables depth-variable gain compensation (crucial for imaging at varying depths). WULPUS's PGA is fixed during acquisition — adequate for fixed-depth physiological monitoring, insufficient for general imaging.

3. **Wireless vs wired** — BLE enables WULPUS's wearable use case but caps throughput at 320 kbps. At 8 Msps with 8 channels, WULPUS transmits only a subset of data in real time; the rest is buffered in FRAM and sent between acquisitions. USB (pic0rick) has no practical throughput constraint for research frame rates.

4. **Pulser polarity** — Bipolar ±24V (pic0rick) produces symmetric excitation; the transducer rings symmetrically, potentially improving bandwidth efficiency. Unipolar +15V (WULPUS) is simpler to implement but drives the transducer asymmetrically. The frequency ceilings (WULPUS: 3 MHz practical, pic0rick: ~32 MHz) mean this rarely affects the same transducer.

5. **Power strategy** — WULPUS achieves 22 mW through co-design: MSP430 USS_A eliminates the external ADC, LPM4+RTC (450 nA) between acquisitions, FRAM captures samples without CPU involvement via DMA, and all subsystems are power-gated when idle. No single optimization dominates; it's the system sum.

6. **PCB tool choice** — KiCad (pic0rick) is free and JLCPCB-compatible; anyone can order boards without a software license. Altium (WULPUS) produces professional results but requires ~$10k/year. WULPUS exports Gerbers, making fabrication accessible, but schematic editing requires Altium.

---

### What Neither Device Covers Well

- **Transducers above 4 MHz with low power** — pic0rick handles high frequencies but is USB-tethered and power-hungry; WULPUS is low-power but ADC-limited to ≤3 MHz.
- **Time-gain compensation in a wearable** — neither device supports true TGC (monotonically increasing gain over the receive window). WULPUS-Pro is planned to address this.
- **Real-time 2D B-mode** — both can produce B-mode offline, but neither supports real-time multi-element beamforming. pic0rick lacks on-board compute; WULPUS's BLE bottleneck prevents data rates needed for real-time reconstruction.
- **Coherent multi-element arrays out of the box** — both support up to 8 channels, but full 32-channel synthetic aperture requires additional hardware (cascaded mux PMODs for pic0rick; custom HV PCB for WULPUS).

---

## Repository Structure

| File | Contents |
|------|----------|
| [`plan.md`](plan.md) | Six-phase research plan + Phase 7 (Array Multiplexers). The roadmap document. |
| [`phase1.md`](phase1.md) | Functional block analysis: pulser, TX/RX, amp, ADC, power, timing, connectivity |
| [`phase2.md`](phase2.md) | Hardware performance master table with typed specs (M/D/C/E) |
| [`phase3.md`](phase3.md) | Software and firmware ecosystem; data pipeline traces |
| [`phase4.md`](phase4.md) | Per-device strengths, weaknesses, specificities; published benchmarks |
| [`phase5.md`](phase5.md) | Licenses, PCB tools, cost, community metrics, reproducibility |
| [`phase6.md`](phase6.md) | Synthesis: master table, device cards, design decisions, gap analysis, cross-pollination, decision guide |
| [`picorecommendation.md`](picorecommendation.md) | What pic0rick does well; hardware and software improvements inspired by WULPUS |
| [`MSP430FR5043.md`](MSP430FR5043.md) | Deep dive on the MSP430FR5043 MCU: USS_A module, ADC, PGA, FRAM, DMA, power modes |
| [`TUSS4470.md`](TUSS4470.md) | Deep dive on the TI TUSS4470 standalone ultrasound AFE IC |
| [`TUSvsMSP.md`](TUSvsMSP.md) | TUSS4470 vs MSP430FR5043 comparative analysis |
| [`array_sourcing.md`](array_sourcing.md) | Sourcing guide: compatible transducers and arrays for each platform |
| [`w_discussions.md`](w_discussions.md) | WULPUS GitHub Discussions/Issues summary + Vermon 32-ch array deep analysis |
| [`w_uses.md`](w_uses.md) | All known research uses of WULPUS: 7 papers with full citations |
| [`link.md`](link.md) | Research source tracker |
| [`todo.md`](todo.md) | Completed items + 8 documented open gaps |

---

## MSP430FR5043 vs TUSS4470

Two Texas Instruments ICs that appear superficially similar (both for ultrasound) but are architecturally different:

**TUSS4470** (`TUSS4470.md`, `TUSvsMSP.md`): A standalone analog front-end IC. It contains a transmit H-bridge driver (40 kHz–1 MHz, up to 24V) and a logarithmic receive amplifier (94 dB dynamic range). It has no ADC and no microcontroller — it outputs an analog envelope signal. Price: $1.63. Suited for industrial proximity sensing and flow metering where wide dynamic range matters but raw RF data is not needed.

**MSP430FR5043** (`MSP430FR5043.md`, `TUSvsMSP.md`): A complete 16-bit MCU with an integrated ultrasonic sensing peripheral (USS_A) containing a programmable pulse generator (0–5 MHz), sigma-delta ADC (8 Msps, 12-bit), and programmable gain amplifier. FRAM (64 KB), DMA, BLE-compatible. Standby at 450 nA. This is the chip at the heart of WULPUS.

The comparison is not symmetric: the TUSS4470 is a component; the MSP430FR5043 is a system. Their frequency ranges are adjacent (TUSS4470: 40 kHz–1 MHz; MSP430FR5043: 1–4 MHz practical) and do not significantly overlap.

---

## Array Sourcing Summary

Full details in [`array_sourcing.md`](array_sourcing.md).

**pic0rick-compatible transducers (any frequency up to ~32 MHz):**
- Olympus V110 (5 MHz), V111 (10 MHz) — the standard research starting points
- Generic NDT contact probes (5 MHz, BNC) — affordable for development
- 32-element arrays via cascaded MAX14866 PMODs (4× PMOD = 32 channels)

**WULPUS-compatible transducers (1–3 MHz only):**
- Olympus V106 / V125 (2.25 MHz) — characterized, LEMO 00 connector
- Generic 2.25 MHz contact probe (BNC) — lowest cost starting point
- Vermon research arrays (custom FPC, requires adapter PCB, see below)

**Vermon 32-channel array for WULPUS** (from `w_discussions.md` analysis):
- Feasible as a research tool but requires: (1) Vermon array at ≤3 MHz, (2) BLE replaced by USB for data transfer, (3) new HV PCB with 4 cascaded mux stages. Without these changes, BLE limits frame rate to ~0.2 fps for 32-channel synthetic aperture — not useful for imaging.

---

## pic0rick Recommendations

Full details in [`picorecommendation.md`](picorecommendation.md).

**pic0rick's core strengths**: 65 Msps ADC enabling compatibility with 2–30 MHz transducers; AD8331 TGC with 12-bit DAC for depth-variable gain; RP2040 PIO for flexible nanosecond-resolution timing; PMOD modularity for 8-channel expansion; Python-first API; OSHWA-certified open hardware with Tindie availability.

**Hardware improvements from WULPUS**: low-power standby mode between acquisitions; multi-channel HV mux PMOD (MAX14866); power gating for ADC and amplifier during idle periods; FRAM or external SDRAM for autonomous frame buffering.

**Software improvements (SW-1 through SW-7)**: typed AcquisitionConfig dataclass; Hilbert envelope detection; configurable bandpass filter matching transducer frequency; M-mode scrolling display; automated pulse tuning via echo amplitude optimization; versioned HDF5 schema with metadata; CMake/UF2 build documentation.

---

## How to Continue This Research

See [`claude.md`](claude.md) for a detailed agent skill file that documents the methodology, file structure, and prompts needed to continue or replicate this comparison with a fresh agent.

---

## Open Gaps

From the review pass documented in `todo.md`:

1. WULPUS HV mux chip identity unconfirmed (provisionally HV2707T-C/R8X)
2. MSP430FR5043 SDHS ENOB unpublished by TI
3. pic0rick system SNR uncharacterized (no phantom measurements published)
4. WULPUS-Pro: referenced in papers but no public repository as of 2026-06
5. Vermon 32-ch adapter PCB: analysis done, physical design not yet built
6. Array pricing estimates need direct quotation from manufacturers
7. M-mode GUI support for WULPUS: open question, no documented answer
8. Board-to-board calibration procedure: multiple builders note per-unit variability, no procedure exists

---

*Research compiled by loup (NanoClaw agent) for Luc, 2026-06. All analysis from public datasheets and published papers; no physical hardware was available for testing.*
