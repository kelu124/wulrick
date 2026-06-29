# EchoLite

**Full name:** EchoLite — lightweight wearable ultrasound system (working name from Anatomy slides)
**Reference:** Lu et al. — IEEE International Ultrasonics Symposium 2025
**Affiliation:** Not confirmed; likely academic group (author surname Lu is common in Chinese and US academic ultrasound research)
**Status:** ⚠ Not yet indexed in public databases as of 2026-06-30. IEEE IUS 2025 proceedings (Utrecht, September 15–18, 2025) are not fully indexed. Specs sourced from the Anatomy slides (Leitner & Giordano, IEEE CEEUS Warsaw 2026), which cite this work directly.

---

## What It Is

EchoLite is a lightweight single-channel wearable ultrasound system. Based on the Anatomy slides, it achieves 33 mW system power and 11 µm axial resolution at 3 cm imaging depth — positioning it between WULPUS (22 mW, 5.5 µm, 4 cm) and USoP (614 mW, 102.3 µm, 6 cm) in the power-resolution trade-off space.

Its weight (8.7 g) is lower than all other systems in this comparison, suggesting aggressive miniaturization of the probe assembly. The name "EchoLite" is consistent with a design emphasis on minimizing mass and volume.

---

## Hardware Platform

**Primary:** MCU-based (specific parts not confirmed — paper not yet publicly accessible)

The MCU platform is inferred from the Anatomy slides classification. No schematics, firmware, or block diagrams are publicly available as of this writing.

---

## Key Specifications

*All figures from the Anatomy slides (Leitner & Giordano, IEEE CEEUS 2026). No independent source found to cross-check.*

| Parameter | Value | Source |
|-----------|-------|--------|
| Active receive channels | 1 | M — Anatomy slides |
| Weight | 8.7 g | M — Anatomy slides |
| System power | 33 mW | M — Anatomy slides |
| Imaging depth | 3 cm | M — Anatomy slides |
| Axial resolution | 11 µm | M — Anatomy slides |
| Hardware platform | MCU | M — Anatomy slides |

---

## Key Innovations

*Inferred from the spec positioning; not confirmed from the paper itself.*

- **Lowest reported weight (8.7 g)** among the five systems in this comparison — lighter than WULPUS (13–15 g) despite comparable power. Suggests aggressive PCB miniaturization or reduced battery capacity.
- **Resolution improvement over WULPUS** (11 µm vs 5.5 µm) at comparable power (33 mW vs 22 mW), suggesting a higher-frequency transducer or improved signal processing.
- **MCU-based at 33 mW** — consistent with the MSP430 / ARM Cortex-M class approach rather than FPGA or ASIC.

---

## Constraints

- **Imaging depth of 3 cm** — shallower than WULPUS (4 cm) and USoP (6 cm). Suitable for superficial structures (radial artery, superficial muscles, tendons) but may not reach deep structures (carotid at depth, diaphragm, deep abdominal muscles).
- **Single channel** — A-mode only; no multi-element imaging.
- **No public data** — paper not yet indexed; cannot verify specs, methodology, or hardware details.

---

## Relevance to pic0rick and WULPUS

EchoLite's most notable characteristic relative to WULPUS is its weight (8.7 g vs ~13–15 g) at roughly comparable power. If this is achieved by reducing the PCB area and battery rather than through circuit innovations, it represents a form-factor refinement rather than a new design approach. If it reflects a different signal processing architecture or transducer choice, that would be a meaningful advance.

The 3 cm depth limitation may indicate a higher-frequency transducer (higher frequency = better resolution, shallower penetration). If EchoLite uses a 5 MHz transducer where WULPUS uses 2.25 MHz, this would explain both the improved resolution and reduced depth — and would also imply a higher-speed ADC than the MSP430FR5043's 8 Msps ceiling, which would be a significant design distinction.

**This file should be updated when the IEEE IUS 2025 paper becomes publicly available.**
