# WULPUS PRO

**Full name:** WULPUS PRO: Multi-mode Ultra-Low-Power Wearable Ultrasound and Array Imaging with CMUT Support
**Reference:** Vostrikov, Villani, Hirschi, Lu, Welsch, Angerer, Cretu, Rohling, Cossettini, Benini — arXiv 2026
**arXiv:** 2607.12137v1
**Repository:** https://github.com/pulp-bio/wulpus-pro
**Affiliation:** ETH Zurich PULP group / University of British Columbia

---

## What It Is

WULPUS PRO is the second-generation wearable ultrasound platform from ETH Zurich's PULP group, building directly on WULPUS (2022). It extends the original design with three operating modes (A-mode RF, A-mode envelope, B-mode synthetic aperture imaging), 16-channel time-multiplexed acquisition, analog time-gain compensation (TGC), and CMUT transducer support — while retaining the same MSP430FR5043 acquisition core.

**Applications:** Muscle dynamics monitoring, bladder volume assessment, cardiovascular activity tracking, hand/finger movement tracking, long-term continuous body monitoring.

---

## Hardware Platform

**Primary:** MCU-based (MSP430FR5043), same chip as WULPUS original.

| Component | Part | Role |
|-----------|------|------|
| Acquisition MCU | MSP430FR5043 (TI) | USS_A: PPG + 8 Msps 12-bit sigma-delta ADC + PGA; FRAM data buffer |
| BLE radio | nRF52832 (Nordic) | BLE 5.0, 320 kbps effective; same as WULPUS |
| WiFi (optional) | ESP32-C6-DevKitC-1 (Espressif) | 2 Mbps measured throughput; alternative wireless path |
| HV supply | LT3463 (Analog Devices) | Dual DC-DC buck-boost: +30 V (TX) and −30 V (CMUT bias) from single LiPo |
| HV mux | HV2707 (Microchip) | 16-channel time-multiplexed TX/RX switch; ±100 V rated; SPI-controlled |
| T/R switch | MD0100 (Microchip) | Passive TX/RX limiter; ±2.5 V clamp; ~5 µs RX recovery |
| Pulser driver | IXDD604 (Littelfuse) | MOSFET gate driver; tri-state; steps 3.3 V PPG signal to 30 V |
| Pre-amplifier | OPA836 (TI) | 6 dB gain buffer; AC-coupled input; same as WULPUS |
| VGA / TGC | AD8338 (Analog Devices) | 0–74 dB programmable VGA; analog RC network drives TGC (40 dB slope) |
| Envelope detector | LT5507 (Analog Devices) | RF power detector, 1.5 MHz BW, <2 mW; optional bypass for direct RF |

**Transducers tested:**
- LA-2.25-32 (Vermon, Tours): 2.25 MHz, 32-element linear array, 0.6 mm pitch, 19.2 mm aperture
- LA-5-32 (Vermon, Tours): 5 MHz, 32-element linear array, 0.6 mm pitch
- 16-channel polyCMUT array: custom flexible PCB, 0.64 mm pitch, 3 MHz nominal center frequency, 119% fractional bandwidth

---

## Key Specifications

| Parameter | Value | Notes |
|-----------|-------|-------|
| Form factor | 39 × 21 × 6 mm | vs 46 × 25 mm for WULPUS |
| Weight | **5 g** | vs ~13 g for WULPUS |
| RX channels | 16 (time-multiplexed) | vs 8 in WULPUS |
| TX voltage | 0–30 V unipolar | vs +15 V in WULPUS; enables CMUT DC biasing |
| ADC rate | 8 Msps, 12-bit | Unchanged — same MSP430FR5043 |
| End-to-end BW (−3 dB) | 1.4 MHz | Limited by CIC decimation filter in MSP430 sigma-delta ADC |
| Amplification chain BW | 305 kHz – 10.2 MHz | Without ADC as bottleneck |
| Excitation frequency | 0.2–10 MHz | PPG register-configurable |
| TGC | 40 dB slope via RC network | Not available in WULPUS original |
| Total gain | Up to 70 dB (OPA836 + AD8338) | VGA replaces fixed op-amp stage of WULPUS |
| SNR | 32 dB at passband centre, 40 dB gain | SINAD peak: 41 dB @ 1 MHz, 36 dB @ 2.25 MHz |
| Core power — A-mode | **35 mW** @ 50 Hz PRF | +12 mW BLE; +314 mW WiFi |
| Core power — B-mode | **58 mW** @ 300 Hz PRF per channel | 16-ch synthetic aperture |
| Total power — A-mode BLE | ~47 mW | vs 22 mW for WULPUS |
| Axial resolution (B-mode) | ~0.7 mm @ 3 cm depth | 2.25 MHz, synthetic aperture (vs 5.5 µm *displacement precision* in WULPUS A-mode) |
| Lateral resolution | 2.0–2.3 mm | DAS beamforming with coherence factor weighting |
| B-mode frame rate | 18.75 Hz | 16 elements × 300 Hz PRF / 16 |
| PolyCMUT measured fc | 1.46 MHz | Nominal 3 MHz; limited by 1.4 MHz system BW |
| PRF range | 1–300 Hz | Mode-dependent |
| Battery life (BLE) | 1–2 days @ 50 Hz | 300 mAh LiPo |
| Battery life (WiFi) | >3 hours @ 300 Hz | 300 mAh LiPo |
| Host interface | 4-wire SPI, 8 MHz | Host-agnostic front-end; data-ready + host-ready handshake |

---

## Operating Modes

| Mode | RX channels | PRF | Frequency | ADC | Output | Core power |
|------|-------------|-----|-----------|-----|--------|-----------|
| I — A-mode RF | 1 | 50 Hz | 2.25 MHz | 8 Msps | Raw RF | 35 mW |
| II — A-mode envelope | 1 | 50 Hz | 2.25 or 5.0 MHz | 8 Msps | Envelope | 38 mW |
| III — B-mode | 16 mux | 300 Hz / ch | 2.25 MHz | 8 Msps | Raw RF | 58 mW |

*B-mode effective image rate: 300 Hz × 1/16 = 18.75 Hz. Each frame uses plane-wave transmission and synthetic aperture delay-and-sum beamforming with coherence factor weighting.*

---

## Key Advances over WULPUS Original

1. **Analog TGC (AD8338 VGA).** The gain is varied during the receive window by an RC network that produces a rising ramp voltage on the AD8338 gain control pin, compensating for depth-dependent attenuation with a 40 dB slope. WULPUS's MSP430 PGA is fixed during acquisition — TGC was the single most requested missing capability.

2. **CMUT support.** The AC-coupled input path and the LT3463 dual ±30 V supply enable both indirect (−30 V on CMUT top electrode ground) and direct (simultaneous ±30 V) CMUT biasing. WULPUS's +15 V unipolar supply cannot provide the DC offset required for CMUT collapse-mode operation.

3. **B-mode array imaging.** 16-channel time-multiplexed acquisition enables synthetic aperture plane-wave imaging. WULPUS's 8-channel mux is a site selector for independent A-mode channels; WULPUS PRO's 16 channels support lateral reconstruction across a 10.2 mm aperture.

4. **Smaller and lighter.** 5 g vs ~13 g; 39×21 mm vs 46×25 mm. The LT3463 dual-output converter (single IC for both rails) is the primary BOM reduction vs WULPUS's separate +15 V boost plus any auxiliary supply.

5. **WiFi link option.** The ESP32-C6 provides ~2 Mbps vs nRF52832 BLE's 320 kbps — a 6× throughput increase. This makes B-mode RF streaming at 300 Hz PRF feasible wirelessly.

---

## Constraints

- **8 Msps ADC ceiling unchanged.** Same MSP430FR5043; end-to-end −3 dB bandwidth limited to 1.4 MHz. PolyCMUT arrays with 3 MHz nominal center frequency are imaged at only 1.46 MHz effective — less than half their design point. *Proposed future fix: STM32L412 at 12.3 Msps (10-bit interleaved) → ~6 MHz BW.*
- **Single receive chain.** 16 channels are time-multiplexed through one ADC. Full-parallel 16-channel reception requires 16 simultaneous ADC instances — not achievable at MSP430 power levels.
- **B-mode frame rate below real-time for fast structures.** 18.75 Hz is adequate for muscle, bladder, carotid; borderline for fast cardiac motion.
- **MD0100 (passive T/R) vs MD0101 (active T/R).** The passive limiter's ~5 µs recovery creates a ~3.75 mm near-field dead zone at 1540 m/s. WULPUS uses MD0101 (100 ns recovery); pic0rick offers the MD0101 upgrade as an option. WULPUS PRO's choice of MD0100 limits near-surface imaging.

---

## Comparison to WULPUS and Other Systems

| | WULPUS | WULPUS PRO | PuLsE | TinyProbe |
|-|--------|-----------|-------|----------|
| Year | 2022 | 2026 | 2025 | 2025 |
| MCU | MSP430FR5043 | MSP430FR5043 | Custom MCU | FPGA |
| Channels | 8 mux | 16 mux | 1 | 32 parallel |
| TX voltage | +15 V | +30 V | ±15 V est. | ±32 V |
| TGC | None | AD8338 (40 dB) | None | Programmable |
| CMUT support | No | Yes | No | No |
| B-mode | No | Yes (SA, 16 ch) | No | Yes (32 ch) |
| Core power | 22 mW | 35–58 mW | 5.8 mW | 430–830 mW |
| Weight | ~13 g | **5 g** | ~12.6 g | 39.9 g |
| Open source | Yes | Yes | Yes | Partial |

WULPUS PRO slots between WULPUS and TinyProbe: more capable than WULPUS (TGC, B-mode, CMUT, 2× channels), substantially less power-hungry than TinyProbe (58 mW vs 830 mW), but retains the same 8 Msps ADC bandwidth constraint.

---

## Relevance to ultr4rick / pic0rick

WULPUS PRO validates the **AD8338** as the correct low-power TGC VGA for wearable ultrasound front-ends — the same part is a candidate for ultr4rick's analog chain. The **LT3463** dual-output ±30 V supply is a compact alternative to the separate +/− supplies if ultr4rick targets bipolar ±24 V or CMUT operation.

The core bandwidth bottleneck (1.4 MHz end-to-end with MSP430) is the primary argument for ultr4rick's CH569 + high-speed external ADC architecture: the WULPUS PRO roadmap's proposed fix (STM32L412 at 12.3 Msps) achieves ~6 MHz BW, while a pic0rick-style path (ADC10065 at 65 Msps via CH569 USB3) delivers ~32 MHz — more than 5× better — on a single PCB.
