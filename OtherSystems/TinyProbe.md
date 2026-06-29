# TinyProbe

**Full name:** TinyProbe: A Wearable 32-Channel Multimodal Wireless Ultrasound Probe
**Reference:** Vostrikov, Tille, et al. — IEEE Transactions on Ultrasonics, Ferroelectrics, and Frequency Control, Vol. 72, No. 1, pp. 64–76, January 2025
**DOI:** 10.1109/TUFFC.2024.3496474
**IEEE Xplore:** https://ieeexplore.ieee.org/document/10750870/
**ETH Research Collection:** https://www.research-collection.ethz.ch/handle/20.500.11850/714261
**GitHub:** https://github.com/pulp-bio/TinyProbe
**Affiliation:** ETH Zurich, Integrated Systems Laboratory (IIS) / PULP group — same team as WULPUS

---

## What It Is

TinyProbe is the direct successor to WULPUS, scaling from 1 active channel to 32 and adding FPGA-accelerated deep learning for virtual channel reconstruction. It is the most capable wearable ultrasound probe published by the ETH Zurich group as of 2025, enabling B-mode imaging and high-PRF Doppler alongside the A-mode monitoring that WULPUS demonstrated. The wireless link switches from BLE (WULPUS's 320 kbps) to Wi-Fi (21.6 Mb/s UDP), resolving the bandwidth bottleneck identified in the WULPUS analysis.

---

## Hardware Platform

**Primary:** FPGA-based with custom multiplexing ASICs

| Component | Role |
|-----------|------|
| FPGA | Central compute and control; specific part not disclosed in available sources |
| Custom ASICs | Local preamplifiers and channel multiplexing |
| Transmit | 64V peak-to-peak; 16 programmable delay profiles for transmit beamforming |
| ADC | Up to 30 Msps, 10-bit (external high-speed ADC, not MSP430 integrated) |
| Wireless | Wi-Fi, 21.6 Mb/s UDP |
| Battery | 500 mAh Li-Po |

The move from MSP430 USS_A (WULPUS) to FPGA is driven by the demands of 32-channel coordination: the FPGA can simultaneously manage transmit beamforming delay profiles, ADC data capture from 32 channels, and deep learning inference for channel reconstruction — tasks that exceed the MSP430's 16 MHz single-core capability by orders of magnitude.

---

## Key Specifications

| Parameter | Value | Source |
|-----------|-------|--------|
| Active receive channels | 32 | M — IEEE TUFFC 2025 |
| Weight | 39.9 g | M — Anatomy slides / TUFFC paper |
| System power (B-mode, 33 Hz) | <1 W (full 32 ch at imaging PRF) | M — TUFFC paper |
| Power per channel | 30.3 mW/ch (Anatomy slides metric) | M — Anatomy slides 2026 |
| Power efficiency | 2 mW/MHz (best in class per Anatomy slides) | M — Anatomy slides 2026 |
| Imaging depth | 15 cm | M — TUFFC paper |
| Axial resolution | 2 µm | M — Anatomy slides 2026 |
| ADC | 30 Msps, 10-bit | M — TUFFC paper |
| Transducer aperture | 83.2 mm × 16 mm | M — TUFFC paper |
| High-PRF Doppler power | <1.3 W (2 channels at 1400 Hz PRF) | M — TUFFC paper |
| Wireless throughput | 21.6 Mb/s (UDP) | M — TUFFC paper |
| Wireless efficiency | 44.9 mW·s/Mb | M — TUFFC paper (best reported for wearable US) |

*Note on "30.3 mW per channel": the Anatomy slides present this as a normalized figure for cross-system comparison. TinyProbe's total active power for 32-channel B-mode is <1 W, not 32 × 30.3 mW = 970 mW. The per-channel metric isolates channel-count scaling.*

---

## Key Innovations

1. **32-channel wearable ultrasound** — 32× channel count of WULPUS in a wearable form factor. The critical challenge is managing 32 × 30 Msps data streams in real time at wearable power levels.

2. **FPGA-accelerated deep learning for virtual channel reconstruction.** Rather than physically implementing all 32 channels at full signal quality, TinyProbe uses a neural network to reconstruct missing or lower-quality channels from a subset of measurements. This effectively doubles imaging aperture while halving physical front-end component count and power.

3. **Transmit beamforming with 16 delay profiles.** Unlike WULPUS's unipolar single-phase pulser, TinyProbe supports programmable transmit delays across elements — enabling focused beam steering and improved spatial resolution over synthetic aperture alone.

4. **64V transmit voltage.** More than 4× WULPUS's +15V, enabling deeper tissue penetration (15 cm vs 4 cm) and higher acoustic energy for Doppler measurements.

5. **Wi-Fi data link.** Replacing BLE (320 kbps) with Wi-Fi (21.6 Mb/s) removes the bandwidth bottleneck that prevents real-time B-mode over wireless. At 21.6 Mb/s, 32-channel frames can be transmitted in near-real time.

6. **Dual imaging mode:** B-mode (tissue structure) and high-PRF Doppler (blood flow velocity) from the same hardware with mode switching.

---

## Applications

- Wearable B-mode imaging for structural tissue monitoring
- Blood flow velocity (Doppler) at depth
- Multi-channel muscle imaging (32-element array over forearm/limb)
- Deep tissue monitoring (bladder, cardiac, hepatic) at 15 cm
- Research platform for FPGA-accelerated wearable imaging algorithms

---

## Constraints

- **39.9 g** — heaviest system in this comparison; approaching the upper bound of comfortable wearable weight for wrist/forearm positioning
- **<1 W active power** — acceptable for intermittent imaging sessions, but approximately 45× higher than WULPUS's 22 mW. Not suitable for multi-day continuous monitoring.
- **Wi-Fi dependency** — requires Wi-Fi infrastructure; not suitable for remote or outdoor use cases where BLE/cellular would be preferable
- **FPGA complexity** — higher development and fabrication cost than MCU-based designs; less accessible for independent replication
- **64V transmit** — requires careful HV supply design and safety considerations for skin-contact use

---

## Relevance to pic0rick and WULPUS

TinyProbe is the most direct answer to the "what would WULPUS look like with 32 channels?" question raised in `w_discussions.md` Issue #16 (Vermon array adapter). The ETH group's own answer: replace the MSP430 with an FPGA, replace BLE with Wi-Fi, add transmit beamforming, and accept a 45× power increase to gain 32× channels and 3.75× depth.

This establishes the design boundary: below ~30 mW, only 1-channel A-mode monitoring is achievable with current technology. 32-channel B-mode at wearable power (<1 W) requires FPGA-class compute and Wi-Fi bandwidth.

For pic0rick, TinyProbe's 30 Msps ADC is closer to pic0rick's 65 Msps capability than WULPUS's 8 Msps. The pic0rick → TinyProbe architectural path (high-speed ADC + flexible MCU/FPGA + PMOD channel expansion) is more direct than the WULPUS → TinyProbe path.

**See also:** `w_discussions.md` Vermon 32-ch analysis for the pre-TinyProbe assessment of what 32-channel WULPUS would require. TinyProbe's published specs confirm the BLE bottleneck prediction (resolved by Wi-Fi) and the power increase estimate.
