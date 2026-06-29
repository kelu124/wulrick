# WULPUS

**Full name:** Wearable Ultra Low-Power Ultrasound Probe
**Reference:** Frey, Vostrikov, Benini, Cossettini — IEEE International Ultrasonics Symposium 2022
**DOI:** 10.1109/IUS54386.2022.9958156
**Repository:** https://github.com/pulp-bio/wulpus
**Affiliation:** ETH Zurich, Integrated Systems Laboratory (IIS) / PULP group

---

## What It Is

WULPUS is an open-source wearable ultrasound probe for continuous multi-day physiological monitoring. It is the baseline device in this comparison, documented extensively in the main repository. It represents the starting point of the ETH Zurich wearable ultrasound line (followed by TinyProbe, ModulUS).

---

## Hardware Platform

**Primary:** MCU-based (MSP430FR5043 + nRF52832)

| Component | Part | Role |
|-----------|------|------|
| Acquisition MCU | MSP430FR5043 | USS_A peripheral: PPG + 8 Msps ADC + PGA; FRAM |
| BLE radio | nRF52832 | BLE 5.0, 320 kbps effective throughput |
| USB dongle | nRF52840 | Receives BLE on host side; USB CDC to PC |
| HV supply | Boost converter | +15V for MOSFET pulser |
| HV mux | ~HV2707T (provisional) | 8-channel time-multiplexed TX/RX switch |
| External op-amp | Fixed +10 dB | Power-gated between acquisitions |

---

## Key Specifications

| Parameter | Value | Source |
|-----------|-------|--------|
| Active receive channels | 1 (8-ch time-multiplexed) | M — IUS 2022 paper |
| Volume / form factor | 14.95 cm³ / 46×25 mm | M — IUS 2022 paper |
| Weight (with battery) | ~13–15 g | M — IUS 2022 paper |
| Total system power | 22 mW (measured) | M — IUS 2022 paper |
| Transducer frequency | 2.25 MHz (primary) | D — MSP430 USS_A range |
| ADC sampling rate | 8 Msps (12-bit) | D — MSP430FR5043 datasheet |
| Imaging depth | ~4 cm (soft tissue) | M — published demos |
| Axial resolution | 5.5 µm | M — IUS 2022 paper |
| Gain | 40.8 dB fixed (PGA + op-amp) | D — MSP430 + op-amp datasheet |
| PRF | 50 Hz (standard config) | M — firmware config |
| Battery life | 2.5+ days (350 mAh LiPo) | M — IUS 2022 paper |
| Wireless | BLE 5.0, 320 kbps | D — nRF52832 datasheet |
| PCB area | 46×25 mm | M — IUS 2022 paper |

---

## Key Innovations

1. **Ultra-low standby power.** MSP430FR5043's LPM4+RTC mode (450 nA) allows multi-day operation from a small cell. No other published wearable ultrasound system achieves comparable standby power.
2. **Integrated acquisition chain.** The MSP430FR5043 USS_A subsystem integrates PPG + ADC + PGA + DMA on one die, eliminating external ADC chip power (~68 mW for a comparable standalone ADC).
3. **Open-source hardware.** Complete Altium schematics, Gerbers, firmware source, and Python host software published. Independently reproduced by at least 2 external labs.
4. **8-channel time multiplexing.** A single receive chain is shared across 8 transducer elements via HV mux, enabling multi-site monitoring at no additional ADC cost.

---

## Applications Published

| Application | Paper | Key Result |
|-------------|-------|-----------|
| Carotid artery waveform | Frey et al. 2022 IUS | Continuous wall displacement tracking |
| Muscle thickness monitoring | Frey et al. 2022 IUS | Contraction tracking during exercise |
| Cardiorespiratory monitoring | Vostrikov et al. 2023 IUS | <3.9% HR error, 17 mW |
| Hand gesture recognition | Vostrikov et al. 2023 IUS | 97% accuracy, 16 mW |
| Wrist/hand kinematics | Spacone et al. TBioCAS 2025 | RMSE 7.32°, 5 subjects |
| Wrist heart rate (PuLsE-WULPUS) | Vostrikov et al. arXiv 2024 | r=0.99 vs ECG |

---

## Constraints

- ADC limited to 8 Msps → maximum usable transducer frequency ~3 MHz; no compatibility with standard medical imaging probes (5–15 MHz)
- Fixed gain during acquisition → no TGC; depth-variable targets require post-processing normalization
- BLE 320 kbps → data throughput cap; 32-channel SA imaging impractical wirelessly
- Three C toolchains (TI CCS + Segger Embedded Studio + Nordic SDK) → high firmware barrier
- Altium PCB source → schematic modification requires ~$10k/yr software license

---

## Relevance to pic0rick

WULPUS is the primary comparator. See `phase1.md`–`phase6.md` for the full analysis. In the OtherSystems context, WULPUS represents the MCU-integration approach taken to its logical extreme: the lowest power per acquisition of any published wearable ultrasound system, at the cost of frequency range and gain flexibility. pic0rick takes the opposite approach: maximum flexibility and raw data access, at the cost of power and form factor.
