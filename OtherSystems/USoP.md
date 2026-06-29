# USoP

**Full name:** Ultrasonic System on Patch — fully integrated wearable ultrasound system
**Reference:** Lin, Zhang, et al. — Nature Biotechnology, 2023 (published online May 2023)
**DOI:** 10.1038/s41587-023-01800-0
**Affiliation:** University of California San Diego (UCSD), Department of Nanoengineering; group of Sheng Xu
**Note on date:** The task metadata lists 2024; the Nature Biotechnology publication date is May 2023. A 2024 follow-on study focused on clinical validation of blood pressure monitoring.

---

## What It Is

USoP is the first reported fully integrated, cable-free wearable ultrasound system capable of monitoring deep tissues (up to 6 cm) in moving subjects. Unlike WULPUS, which separates the transducer from a rigid PCB, USoP integrates control electronics directly into a soft, conformable patch that adheres to the skin. The system requires no cables, no external controller, and operates autonomously on battery.

---

## Hardware Platform

**Primary:** MCU-based (custom integrated flexible electronics)

The control circuitry is embedded within the patch. Published details on specific MCU part number are limited. The architecture includes:

| Component | Description |
|-----------|-------------|
| Transducer array | Flexible PZT-5H (lead zirconate titanate) elements with serpentine Cu/PI electrodes for stretchability |
| Control IC | Onboard microcontroller with ADCs; specific part not published |
| Wireless | Bluetooth (specific spec not disclosed) |
| Power | Onboard Li-ion battery within patch assembly |
| Processing | On-patch signal acquisition; host-side ML for deep tissue tracking |

The key hardware innovation is the **flexible/stretchable interconnect**: the transducer elements and control electronics are embedded in a silicone matrix with serpentine copper traces, allowing the patch to conform and stretch with skin without signal loss or delamination.

---

## Key Specifications

| Parameter | Value | Source |
|-----------|-------|--------|
| Active receive channels | 1 (single-element focused) | M — Nat. Biotech. 2023 |
| Weight | 16.2 g | M — Anatomy slides (Leitner & Giordano 2026) |
| System power | 614 mW (continuous) | M — Anatomy slides / Nat. Biotech. 2023 |
| Imaging depth | ~6 cm | M — Nat. Biotech. 2023 |
| Axial resolution | 102.3 µm | M — Anatomy slides 2026 |
| Battery life | ~12 hours (continuous) | M — Nat. Biotech. 2023 |
| Wireless | Bluetooth (spec not disclosed) | M — Nat. Biotech. 2023 |
| Transducer frequency | ~3–5 MHz (estimate; not disclosed) | E |

*Power of 614 mW is the dominant differentiator from all other systems in this comparison.*

---

## Key Innovations

1. **First cable-free, autonomous wearable ultrasound patch.** All prior wearable ultrasound systems (including WULPUS at time of USoP publication) required a tethered connection or external controller. USoP places everything — transducer, electronics, battery, wireless — in one skin-conforming patch.

2. **Flexible/stretchable electronics integration.** PZT elements embedded in silicone with serpentine interconnects survive repeated stretching and bending without performance degradation. This is a materials and packaging innovation, not primarily an electronics one.

3. **Deep tissue tracking in moving subjects.** A machine learning algorithm compensates for motion artifacts, enabling monitoring during physical activity. Prior wearable ultrasound systems required static or minimally-moving subjects.

4. **Multi-modal physiological capture.** Central blood pressure waveform, heart rate, and cardiac output can be simultaneously derived from a single patch position over the carotid or cardiac window.

---

## Applications

- Continuous cardiovascular monitoring during daily life and exercise
- Central blood pressure measurement (clinical validation in 2024 follow-on work)
- Cardiac output estimation
- Deep-tissue tracking during physical activity

---

## Constraints

- **614 mW** — 28× higher power than WULPUS, 18× higher than EchoLite. Approximately 12 hours of operation on the integrated battery. Not suitable for multi-day monitoring without charging.
- **Axial resolution 102.3 µm** — significantly lower than WULPUS (5.5 µm), EchoLite (11 µm), TinyProbe (2 µm). The flexible patch transducer trades resolution for conformability and stretchability.
- **Single element** — no multi-element imaging; A-mode only.
- **Closed hardware** — no open-source repository. Published schematic-level detail is limited to the Nature Biotechnology supplementary material.
- **Complex fabrication** — flexible PZT array with silicone encapsulation requires specialized facilities not available in standard PCB assembly lines.

---

## Relevance to pic0rick and WULPUS

USoP represents a different axis of wearable ultrasound design: form factor and integration over power efficiency. Where WULPUS prioritizes battery life (22 mW → days), USoP prioritizes mechanical compliance with skin (hours). The 28× power difference is primarily attributable to the control electronics architecture — a power-optimized MCU approach (WULPUS style) has not been applied to the USoP patch format, which represents an open research opportunity.

For pic0rick, USoP is largely orthogonal: pic0rick's open, rigid PCB architecture is incompatible with flexible patch formats. However, USoP validates that A-mode continuous monitoring of deep tissue (carotid, cardiac) is clinically meaningful, which supports the application case for WULPUS-style monitoring.

The power gap (614 mW vs 22–33 mW for WULPUS/EchoLite) is the key unanswered question in USoP's design: if its electronics were redesigned with WULPUS-class power efficiency, could the patch approach achieve multi-day operation? The Anatomy (2026) slides frame this as an active research direction.
