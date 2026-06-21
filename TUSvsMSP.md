# TUSS4470 vs MSP430FR5043 — Comparative Analysis

*Comparing two Texas Instruments ultrasound ICs: the TUSS4470 standalone AFE and the MSP430FR5043 integrated MCU+AFE. Both target ultrasonic sensing but at different integration levels and for different applications.*

---

## 1. What Each Device Is

**TUSS4470:** A standalone analog front-end IC. It contains a transmit driver and a receive signal chain (preamplifier → bandpass filter → logarithmic amplifier → envelope output). It has no ADC and no microcontroller. It requires external components to form a complete system. Price: $1.63.

**MSP430FR5043:** A complete 16-bit microcontroller with an integrated ultrasonic sensing peripheral (USS_A) containing a pulse generator (PPG), sigma-delta ADC (SDHS, 8 Msps, 12-bit), and programmable gain amplifier (PGA). The MCU, FRAM (64 KB), DMA, timers, SPI, UART, and GPIO are all on the same die. Price: varies by supplier, typically $4–8.

The comparison is not symmetric: the TUSS4470 is a component; the MSP430FR5043 is a system.

---

## 2. Side-by-Side Specification Table

| Parameter | TUSS4470 | MSP430FR5043 |
|-----------|----------|--------------|
| **Device type** | Standalone AFE IC | Integrated MCU + AFE |
| **ADC included** | No (external required) | Yes (12-bit SDHS, 8 Msps) |
| **MCU included** | No | Yes (16-bit RISC, 16 MHz) |
| **FRAM/Flash** | No | 64 KB FRAM |
| **DMA** | No | Yes (≥6 channels) |
| **Transmit topology** | H-bridge (bipolar) | PPG (unipolar, drives external FET) |
| **Max transducer voltage** | 24V (VPWR up to 36V) | 3.3V PPG output → external boost to +15V |
| **Transducer frequency range** | 40 kHz – 1 MHz | 0 – 5 MHz (PPG); ADC limits to ≤ 4 MHz |
| **Receive gain type** | Logarithmic amplifier | Linear PGA (46 discrete steps) |
| **Gain range** | 19–113 dB (94 dB log) | −6.5 to +30.8 dB (46 steps, linear) |
| **TGC / variable gain** | Fixed log compression curve | Fixed during acquisition (no TGC) |
| **Receive output** | Analog envelope (VOUT) | Digital samples via DMA to FRAM |
| **Bandpass filter** | Yes (configurable, ≤500 kHz) | Integrated anti-aliasing (specs not published) |
| **Zero-crossing output** | Yes | No explicit output (computed in firmware) |
| **Configuration interface** | SPI (config only) | SPI (config + internal DMA) |
| **Active power** | ~57.5 mW @ 5V | ~5.5 mW @ 16 MHz, 2.9V (MCU only) |
| **Standby power** | ~7.5 mW | **0.45 µW** (450 nA LPM4+RTC) |
| **Sleep power** | ~1.75 mW | ~1.5 µW (LPM4.5 ~35 nA) |
| **Supply voltage** | 5–36V (VPWR) | 1.8–3.6V |
| **Single-cell LiPo compatible** | No (needs 5V minimum) | Yes (direct) |
| **Package** | 20-pin WQFN, 4×4 mm | 64-pin LQFP (WULPUS variant) |
| **Unit price (qty 1)** | $1.63 | ~$4–8 |
| **Datasheet** | SLAS982 | SLASEC4 (or SLAS-series) |
| **Evaluation kit** | BOOSTXL-TUSS4470 | MSP-EXP430FR6989 (general); WULPUS uses directly |

---

## 3. Integration Level

The most fundamental difference is integration.

**TUSS4470 minimum system:**
```
Transducer ↔ [TUSS4470] → VOUT → [External ADC] → [External MCU] → Data
                  ↑ SPI config from MCU
```
Minimum BOM to complete a system: TUSS4470 + external ADC + external MCU + power supply components. Typical board area: ~20–30 cm² for a complete design.

**MSP430FR5043 minimum system:**
```
Transducer ↔ [MSP430FR5043 USS_A] → [FRAM via DMA] → [SPI to external device or USB]
                  (All on single die)
```
Minimum BOM for complete acquisition: MSP430FR5043 + HV boost circuit for pulser (MOSFET + inductor) + decoupling capacitors. Typical board area: ~8–12 cm².

WULPUS adds nRF52832 (BLE) and external op-amp stage, but the core acquisition chain is MSP430-only.

---

## 4. Frequency Range and Transducer Compatibility

| Frequency range | TUSS4470 | MSP430FR5043 (WULPUS) |
|----------------|----------|----------------------|
| 40 kHz – 200 kHz | ✓ Optimal | Possible (PPG capable, ADC oversampled) |
| 200 kHz – 1 MHz | ✓ Full range | ✓ PPG capable; ADC Nyquist allows |
| 1 MHz – 2 MHz | ✗ Exceeds ceiling | ✓ WULPUS primary range |
| 2 MHz – 4 MHz | ✗ | Edge of ADC Nyquist limit |
| 4 MHz – 30 MHz | ✗ | ✗ ADC cannot digitize |

**TUSS4470's range (40 kHz–1 MHz) and MSP430FR5043's range (practical 1–4 MHz) are adjacent, not overlapping.** The TUSS4470 targets industrial sonar and proximity sensing; the MSP430FR5043 targets flow metering and (via WULPUS) wearable biomedical monitoring. Neither covers the medical imaging or NDT range (2–15 MHz). pic0rick (65 Msps) covers this range; neither TI chip does.

---

## 5. Receive Signal Output — Raw RF vs Envelope

This is the most architecturally significant difference.

**TUSS4470 VOUT:** Logarithmically compressed amplitude envelope. The carrier frequency is demodulated out. The output signal's bandwidth is 2–5× lower than the transducer frequency. An ADC sampling at 1–2 Msps is sufficient for a 500 kHz transducer.

**MSP430FR5043 SDHS:** 12-bit samples at 8 Msps of the raw received RF signal. The ADC captures the echo waveform with full phase and frequency information. All signal processing (envelope detection, filtering, Doppler extraction, matched filtering) can be applied in firmware or on the host.

**Consequence for post-processing:**

| Processing task | TUSS4470 (envelope out) | MSP430FR5043 (raw RF) |
|----------------|------------------------|----------------------|
| Time-of-flight measurement | ✓ (zero-crossing output) | ✓ (computed from RF) |
| Echo amplitude measurement | ✓ (envelope amplitude) | ✓ (computed from RF) |
| Matched filtering | ✗ (phase lost) | ✓ |
| Coherent beamforming | ✗ | ✓ |
| Doppler velocity extraction | ✗ | ✓ (complex demodulation) |
| B-mode image reconstruction | ✗ (no RF data) | ✓ (with delay-and-sum) |
| Tissue characterization (elastography) | ✗ | ✓ |

The TUSS4470's envelope output makes it suitable for ToF measurement and echo detection, but not for any processing that requires the underlying RF waveform. The MSP430FR5043 preserves the RF, enabling WULPUS's A-mode traces used for carotid wall tracking, muscle deformation, and gesture recognition — all of which rely on precise echo position rather than just presence/absence detection.

---

## 6. Gain Architecture

**TUSS4470 log-amp (94 dB range):**
The logarithmic amplifier compresses a wide input dynamic range into a narrower output range. Strong echoes and weak echoes are both visible in the same acquisition without saturation or noise floor limiting. The compression curve is fixed (logarithmic transfer function). This is ideal for applications where echo amplitude varies unpredictably (factory floor, range-finding in variable environments).

The 94 dB range is impressive: it means the TUSS4470 can handle echo amplitudes spanning 5 orders of magnitude in a single acquisition window. For industrial proximity sensing, this matters — the same sensor may face a paper-thin label or a concrete wall.

**MSP430FR5043 PGA (37.3 dB range, 46 steps, fixed during acquisition):**
The PGA amplifies the received signal linearly. The 46-step gain range is set before the acquisition sequence starts and held constant. There is no depth compensation (TGC). For WULPUS's use case — fixed-depth monitoring of a specific anatomical structure — this is workable. For any depth-varying application, it is limiting.

**Neither supports true TGC** (monotonically increasing gain over the receive window): the TUSS4470 because its log-amp is a static compression function; the MSP430FR5043 because the PGA is register-set before acquisition and cannot change mid-acquisition.

---

## 7. Power Profile

**TUSS4470 — not viable for wearable use:**

At 57.5 mW active and 7.5 mW standby, a duty-cycled TUSS4470 at 50 Hz PRF (1 ms burst, 19 ms idle) would draw:
- Active: 57.5 mW × 0.05 = 2.875 mW
- Standby: 7.5 mW × 0.95 = 7.125 mW
- **Total: ~10 mW** — before any MCU or ADC power

Adding an MCU (at minimum ~5 mW for active operation) and ADC (~2–10 mW depending on choice), the floor for a complete TUSS4470-based system is **17–30 mW**. On a 350 mAh cell, this gives 12–20 hours — not the 2.5+ days achieved by WULPUS.

**MSP430FR5043 — viable for multi-day wearable:**

The 450 nA LPM4 standby and ~5.5 mW active combined with WULPUS's full system (HV supply, op-amp, nRF52832) achieves 22 mW measured. This is the result of optimizing every subsystem simultaneously.

**At 350 mAh × 3.7V = 1.295 Wh:**
- TUSS4470 system at ~20 mW: **64 hours (2.7 days)** — borderline
- WULPUS MSP430FR5043 system at 22 mW: **58 hours (2.4 days)** — comparable
- If TUSS4470 system draws 30 mW: **43 hours (1.8 days)** — below WULPUS

The gap closes when duty cycling is applied aggressively. But the TUSS4470's 7.5 mW standby floor is difficult to overcome without power-gating the chip entirely between acquisitions, which requires additional switching circuitry and wake-up latency.

---

## 8. Development Ecosystem

| Dimension | TUSS4470 | MSP430FR5043 |
|-----------|----------|--------------|
| IDE | Any MCU IDE (TI CCS, IAR, Arduino) | TI Code Composer Studio (free) |
| SDK | Energia library for eval; direct SPI otherwise | TI MSP430 DriverLib + WULPUS firmware |
| Debug interface | Via paired MCU's debugger | JTAG/SWD (MSP-FET) |
| Example code | EVM GUI examples, TI E2E forum | WULPUS open-source firmware |
| Community | TI E2E forum, industrial integrators | WULPUS GitHub, ETH Zurich group |
| Documentation quality | SLAS982 + SLDA058 well-documented | MSP430FR5043 datasheet (register-level) |
| Learning curve | Medium (SPI config + external ADC integration) | High (3-toolchain WULPUS; direct: medium) |

The TUSS4470 is arguably simpler to start with: configure via SPI, read VOUT with any ADC. The MSP430FR5043 requires understanding the USS_A state machine and DMA for full use.

---

## 9. Cost Comparison (Complete System)

| Component | TUSS4470 system | MSP430FR5043 (WULPUS-style) |
|-----------|----------------|------------------------------|
| Main IC | $1.63 | ~$5 |
| External ADC | $1–5 | — (integrated) |
| External MCU | $2–10 (MSP430, STM32, etc.) | — (integrated) |
| HV supply | $2–5 | ~$3 |
| BLE module | $3–8 (if needed) | ~$5 (nRF52832) |
| **Total BOM (estimate)** | **$10–30** | **~$15–20** |

At system BOM level, both are inexpensive. The TUSS4470's $1.63 unit price is misleading — the complete system requires more external components, reducing the cost advantage. The MSP430FR5043's higher unit price buys integration that eliminates several external chips.

---

## 10. What Each Is Optimised For

**TUSS4470 is optimised for:**
- Industrial ultrasonic presence/absence detection and distance measurement
- Applications needing a simple, low-cost, single-chip AFE with wide dynamic range
- Frequency range: 40 kHz – 1 MHz (sonar, proximity, some flow metering)
- Designs where the MCU and ADC are already present (adding ultrasound to an existing controller platform)
- High-voltage transducer drive (up to 24V) from a 5–36V supply
- Envelope detection only — no raw RF needed

**MSP430FR5043 is optimised for:**
- Ultrasonic flow metering: water, gas, heat meters with µs-precision ToF measurement
- Battery-powered devices requiring multi-day operation without charging
- Integrated designs minimising chip count and board area
- Applications in the 0.5–4 MHz transducer range
- Embedded firmware control of the complete acquisition sequence without external ADC
- Wearable sensing platforms (WULPUS demonstrates this)
- Raw RF capture at 8 Msps for post-processing

---

## 11. Which to Choose

**Choose TUSS4470 if:**
- Your transducer operates below 1 MHz (industrial sonar, proximity sensing, some flow)
- You need wide dynamic range (94 dB log compression) without post-processing
- You already have an MCU and ADC and want to add an ultrasound front-end
- You don't need raw RF data — envelope detection is sufficient
- You're designing for a 5V+ supply (not single-cell LiPo)
- Cost per unit is the primary constraint ($1.63)

**Choose MSP430FR5043 if:**
- You need an integrated acquisition solution with no external ADC or MCU
- Your transducer operates in the 0.5–4 MHz range
- Battery life is critical (multi-day from small cell)
- You need raw RF waveforms for post-processing
- You want to replicate or extend the WULPUS design
- Board space and BOM count are constrained

**Neither is suitable if:**
- You need transducers above 4 MHz (requires external high-speed ADC — pic0rick architecture)
- You need TGC / depth-variable gain during acquisition
- You need real-time coherent beamforming (requires higher compute than either chip)
- You need NDT-class pulse energy and bandwidth at 5–15 MHz
