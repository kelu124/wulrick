# PuLsE

**Full name:** PuLsE: Accurate and Robust Ultrasound-based Continuous Heart-Rate Monitoring on a Wrist-Worn IoT Device
**Reference:** Giordano, Leitner, Vogt, Benini, Magno — IEEE Internet of Things Journal, Vol. 12, 2025
**DOI:** 10.1109/JIOT.2025.3557089
**arXiv:** https://arxiv.org/abs/2410.16219 (preprint; PDF in `pdf/PuLsE_Giordano2025_arXiv.pdf`)
**IEEE Xplore:** https://ieeexplore.ieee.org/document/11050898/
**Affiliation:** ETH Zurich, Integrated Systems Laboratory; ETH Zurich, Department of Information Technology and Electrical Engineering

**Disambiguation:** This PuLsE (Giordano et al. 2025, IEEE IoT Journal) is distinct from a WULPUS-based wrist heart rate paper by Vostrikov et al. (arXiv 2410.16219 was originally the Vostrikov paper; the Giordano paper has a separate IEEE IoT Journal DOI). Both are from ETH Zurich and address wrist-worn ultrasound heart rate monitoring, but with different hardware platforms. The Anatomy slides (Leitner & Giordano 2026) list PuLsE as a separate system entry alongside WULPUS — indicating the two papers use different hardware.

---

## What It Is

PuLsE is a wrist-worn ultrasound system for continuous heart-rate monitoring via pulsatile blood flow detection at the radial artery. It demonstrates ultrasound as a viable alternative to photoplethysmography (PPG) for wearable heart rate tracking — with the advantage of deep-tissue penetration (not blocked by skin pigmentation, motion artifacts, or poor skin contact that degrade PPG).

The system adds an **analog envelope detection circuit** between the ultrasound AFE and the ADC, reducing the required ADC sampling rate by more than 5× compared to raw RF capture. This is architecturally distinct from WULPUS (which captures raw 8 Msps RF and envelopes in firmware) — PuLsE envelopes in the analog domain, allowing use of a lower-speed ADC and lower-power MCU.

---

## Hardware Platform

**Primary:** MCU-based (ARM Cortex-M4 class; specific part not fully disclosed)

| Component | Description |
|-----------|-------------|
| MCU | ARM Cortex-M4 class microcontroller with integrated ADC |
| Analog envelope circuit | Custom circuit; reduces ADC bandwidth requirement >5× |
| Ultrasound pulser | Custom low-power single-cycle pulser |
| Transducer | Single-element, focused (specific frequency not disclosed; compatible with radial artery depth ~5–10 mm) |
| Form factor | Wrist-worn IoT device |
| Wireless | Not specified in available sources |

---

## Key Specifications

| Parameter | Value | Source |
|-----------|-------|--------|
| Active receive channels | 1 | M — Giordano et al. 2025 |
| Volume | 12.6 cm³ | M — Anatomy slides 2026 |
| System power | 5.8 mW | M — Anatomy slides 2026 |
| Application | Wrist heart rate (radial artery) | M — Giordano et al. 2025 |
| Accuracy vs ECG | r = 0.99, mean error 0.69 ± 1.99 bpm | M — Giordano et al. 2025 |
| Envelope vs RF correlation | r = 0.996 | M — Giordano et al. 2025 |
| ADC reduction factor | >5× vs raw RF capture | M — Giordano et al. 2025 |

*Note: Weight in grams not reported in available sources. Volume is 12.6 cm³ from Anatomy slides.*

*Note: 5.8 mW is significantly lower than WULPUS's 22 mW. This reflects: (1) single-element focused wrist probe, no HV mux, no multi-channel switching; (2) analog envelope circuit reducing ADC power; (3) shallow target (~5–10 mm depth for radial artery) allowing lower transmit voltage; (4) lower PRF possible for heart rate monitoring vs tissue tracking.*

---

## Key Innovations

1. **Analog envelope detection reduces ADC bandwidth >5×.** Raw RF from a radial artery at 5–10 mm depth (at e.g. 5 MHz) requires ADC sampling at ≥10 Msps. By enveloping in analog before ADC, a 1–2 Msps ADC suffices. This is architecturally similar to the TUSS4470's approach (see `TUSvsMSP.md`) but optimized for wearable IoT rather than industrial sensing.

2. **5.8 mW total system power** — lowest in this five-system comparison. Achieved by combining shallow target depth (low TX power needed), analog envelope (low ADC rate), and ARM Cortex-M4 class MCU (≈10 mW peak, well below 22 mW WULPUS system which includes HV supply and multi-channel mux).

3. **Wrist-worn form factor** — radial artery is accessible on the ventral wrist, making a smartwatch-style deployment possible. This is a different anatomical site from WULPUS's typical deployments (forearm muscles, carotid, chest).

4. **Comparable accuracy to PPG** — r=0.99 vs ECG at rest and during light motion. Ultrasound's mechanical sensing is orthogonal to PPG's optical sensing, making them complementary rather than competing modalities.

---

## Applications

- Continuous heart rate monitoring on wrist IoT wearable
- Radial artery pulse waveform capture (enables blood pressure waveform beyond heart rate)
- PPG alternative for users with poor PPG signal quality (dark skin, poor perfusion, thick calluses)
- Low-power continuous cardiac monitoring for clinical and consumer use

---

## Constraints

- **Single element, wrist-only** — no multi-site capability; radial artery geometry is specific (requires precise alignment over vessel)
- **No multi-channel or imaging capability** — not designed for B-mode or tissue tracking; optimized purely for pulsatile arterial signal extraction
- **Analog envelope irreversible** — the analog enveloping discards phase information, making coherent processing (matched filtering, Doppler) impossible on this data; only amplitude envelope available (same limitation as TUSS4470)
- **Hardware not fully published** — specific MCU and circuit topology not fully disclosed in arXiv preprint; not open-source

---

## Relevance to pic0rick and WULPUS

PuLsE demonstrates that the analog envelope approach — which the TUSS4470 uses for industrial sensing — is viable and power-optimal for wearable cardiac monitoring. At 5.8 mW, it achieves lower power than WULPUS (22 mW) by accepting the constraint that raw RF is not needed for heart rate extraction.

This has a direct implication for pic0rick: pic0rick captures raw 65 Msps RF and envelopes on the host. For applications that only need the envelope (distance measurement, presence detection, heart rate), the raw RF pipeline wastes power and bandwidth. A PuLsE-style analog envelope front-end (TUSS4470 or custom circuit) combined with pic0rick's host processing would be lower-power for envelope-sufficient applications — while retaining the option to use the full ADC for raw RF when needed.

The PuLsE result also contextualizes WULPUS's 22 mW: WULPUS's overhead vs PuLsE comes from the HV mux (8 channels, adds power), the +15V HV supply (needed for deeper tissue like carotid at 4 cm, overkill for wrist), and the MSP430 USS_A's minimum power configuration. For single-element wrist use, 5.8 mW shows what's achievable.
