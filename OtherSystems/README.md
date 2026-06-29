# OtherSystems — Wearable Ultrasound Platform Survey

*Five recent wearable/portable ultrasound systems, analyzed for relevance to the pic0rick vs WULPUS comparison. Specs sourced primarily from the Anatomy slides (Leitner & Giordano, IEEE CEEUS Warsaw 2026) and individual papers.*

---

## Systems at a Glance

| System | Year | Venue | Platform | Ch | Weight | Power | Depth | Resolution |
|--------|------|-------|----------|----|--------|-------|-------|-----------|
| [WULPUS](WULPUS.md) | 2022 | IEEE IUS | MCU | 1 (8×mux) | ~14.95 g | **22 mW** | 4 cm | 5.5 µm |
| [USoP](USoP.md) | 2023 | Nat. Biotech. | MCU | 1 | 16.2 g | 614 mW | 6 cm | 102.3 µm |
| [EchoLite](EchoLite.md) | 2025 | IEEE IUS | MCU | 1 | **8.7 g** | 33 mW | 3 cm | 11 µm |
| [TinyProbe](TinyProbe.md) | 2025 | IEEE T-UFFC | FPGA | **32** | 39.9 g | 30.3 mW/ch | **15 cm** | **2 µm** |
| [PuLsE](PuLsE.md) | 2025 | IEEE IoT J. | MCU | 1 | ~12.6 g | **5.8 mW** | ~1 cm | — |

*EchoLite: IEEE IUS 2025 paper not yet indexed as of 2026-06-30; specs from Anatomy slides only.*
*PuLsE weight from volume estimate (12.6 cm³); not reported directly.*

---

## Synthesis

### 1. Hardware Trend: MCU at the Low End, FPGA at the High End

Four of the five systems use MCU-based platforms. The MCU approach (MSP430, ARM Cortex-M) enables sub-100 mW operation through integration: the USS_A peripheral on the MSP430FR5043 (WULPUS) eliminates external ADC power; analog envelope detection (PuLsE) reduces ADC rate; optimized duty cycling keeps average power near quiescent levels.

TinyProbe is the outlier: at 32 channels and 30 Msps ADC, the data volume exceeds what any MCU can process in real time. The FPGA enables parallel channel management and neural-network channel reconstruction that would require GHz-class CPU equivalents in software. The power cost is a ~45× increase over WULPUS — an acceptable trade-off for imaging capability, not for continuous monitoring.

**Implication for pic0rick:** The RP2040 sits between these extremes — more flexible than MSP430, less powerful than FPGA. Its PIO-based timing achieves nanosecond resolution for a single channel at low power, but scaling to 32 channels without an FPGA would require significant firmware complexity and would likely hit bandwidth limits. pic0rick's current architecture is optimal for single-channel, high-frequency, high-resolution work; TinyProbe defines what a 32-channel extension requires.

---

### 2. Power vs Channel Count: The Core Trade-Off

The table below compares power efficiency normalized per channel:

| System | Total power | Channels | mW/channel |
|--------|-------------|----------|-----------|
| PuLsE | 5.8 mW | 1 | **5.8** |
| WULPUS | 22 mW | 1 (8 mux) | 22 |
| EchoLite | 33 mW | 1 | 33 |
| TinyProbe | ~970 mW (32×30.3) | 32 | 30.3 |
| USoP | 614 mW | 1 | 614 |

*TinyProbe: "30.3 mW/ch" is the Anatomy slides' normalized metric. Total active power for 32-ch B-mode is <1 W per the TUFFC paper.*

The curve is not linear: going from 1 to 32 channels does not cost 32× power. TinyProbe achieves ~30 mW/channel (comparable to WULPUS's 22 mW for a single channel) by using FPGA parallelism and virtual channel reconstruction — suggesting that the fixed overhead of the HV supply, wireless link, and compute dominates, while the per-channel marginal cost is low once the infrastructure is in place.

**PuLsE at 5.8 mW is the power floor** for any wearable ultrasound system targeting shallow structures at low PRF. Below this, the dominant power consumer is standby current and wireless link, not the ultrasound acquisition itself.

---

### 3. Resolution vs Depth: Physics Defines the Frontier

Axial resolution in ultrasound scales with transducer center frequency (higher frequency = better resolution, shallower penetration):

| System | Resolution | Depth | Implied frequency |
|--------|-----------|-------|------------------|
| TinyProbe | 2 µm | 15 cm | ~5–15 MHz (low freq for depth) |
| WULPUS | 5.5 µm | 4 cm | 2.25 MHz |
| EchoLite | 11 µm | 3 cm | ~5 MHz (estimate) |
| USoP | 102.3 µm | 6 cm | ~1–3 MHz (flexible transducer limitation) |

*Resolution = c/(2 × bandwidth); depth limited by attenuation at 0.5 dB/(cm·MHz) in soft tissue.*

TinyProbe's combination of 2 µm resolution at 15 cm depth is achievable because its 64V transmit voltage compensates for high-frequency attenuation — more acoustic energy pushes the signal further before it falls below the noise floor. This is unavailable to WULPUS (+15V) or PuLsE (wrist, shallow).

USoP's 102.3 µm resolution reflects its flexible transducer: PZT elements embedded in silicone with serpentine interconnects have lower acoustic coupling efficiency and broader bandwidth than rigid clinical probes, degrading resolution.

**For pic0rick:** The ADC10065 at 65 Msps enables transducer frequencies up to ~32 MHz. At 30 MHz, soft tissue attenuation limits depth to ~5 mm — useful for superficial vessel, skin, or tendon imaging, but not cardiac or carotid. At 5 MHz, depth reaches 4–6 cm at pic0rick's ±24V transmit, which is the clinically relevant range for many applications. pic0rick covers the frequency space that WULPUS cannot (>3 MHz) and that TinyProbe partially covers with its 30 Msps ADC (≤15 MHz).

---

### 4. Weight: The Practical Wearability Threshold

Consumer wearables (smartwatch, fitness band) weigh 30–50 g. Research wearable ultrasound systems:

- PuLsE: ~12.6 cm³, low weight (wrist-worn, competitive with smartwatch)
- EchoLite: 8.7 g — lightest; approaching sensor patch territory
- WULPUS: ~14.95 g — acceptable for forearm strapping; heavier than a standard watch
- USoP: 16.2 g — heavier than consumer smartwatch; acceptable for clinical wearable
- TinyProbe: 39.9 g — approaching upper limit for comfortable wrist/forearm wearable; more like a large sport watch

None of the systems approaches the weight of clinical handheld probes (typically 150–400 g), which represents 10× to 100× reduction in form factor that wearable research has achieved over the past decade.

---

### 5. How These Systems Compare to pic0rick and WULPUS

**pic0rick is not in this comparison set** because it is USB-tethered and power-uncharacterized — it does not meet the "wearable" criterion. Its research value is as an open, flexible instrumentation platform, not as a deployable sensor. It covers a frequency range (5–30 MHz) that none of the five wearable systems address, making it complementary rather than competing.

**WULPUS is the baseline** for this comparison — it set the power benchmark (22 mW) that the subsequent systems either approach (EchoLite at 33 mW, PuLsE at 5.8 mW) or exceed (TinyProbe at ~1 W active).

The progression from WULPUS (2022) to TinyProbe (2025) within the same ETH group shows a deliberate escalation: each system traded power budget for imaging capability, guided by the "anatomy" principle (every echo must justify its cost). The Anatomy slides (2026) frame ModulUS as the next step: a fully reconfigurable sandbox where the right combination can be found per application.

---

### 6. Open Questions These Systems Leave

1. **Can FPGA power be reduced to <100 mW for 32 channels?** TinyProbe's <1 W is a large step toward TinyProbe's goal, but multi-day monitoring at 32 channels would require another 45× reduction. Custom ASIC is the likely path.

2. **What is the minimum weight for 32-channel wearable ultrasound?** TinyProbe's 39.9 g reflects a proof-of-concept; a miniaturized ASIC-based design could target <20 g.

3. **Can analog envelope approaches (PuLsE) be combined with multi-channel multiplexing?** WULPUS uses 8 channels at 22 mW; PuLsE uses 1 channel at 5.8 mW. An 8-channel PuLsE-style analog envelope system might achieve 10–15 mW — lower than WULPUS — if the per-channel overhead is truly ~5.8 mW.

4. **What does EchoLite's hardware actually look like?** At 8.7 g and 33 mW with 11 µm resolution, it's the most surprising spec combination; the paper will be informative when available.

---

## Source

All five system files draw from: individual published papers (DOIs in each file), Anatomy of a Wearable Ultrasound System slides (Leitner & Giordano, IEEE CEEUS Warsaw 2026, Zenodo record 21030966), and public GitHub repositories where available. See `references.bib` for structured citations.
