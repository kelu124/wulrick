# WULPUS — Known Research Uses

*Compiled 2026-06-21 from IEEE Xplore, arXiv, ETH Zurich IIS group publications, and the WULPUS GitHub README.*

---

## Primary WULPUS Papers (by the development team)

### 1. WULPUS: a Wearable Ultra Low-Power Ultrasound probe for multi-day monitoring of carotid artery and muscle activity
**Authors:** Sebastian Frey, Sergei Vostrikov, Luca Benini, Andrea Cossettini
**Venue:** 2022 IEEE International Ultrasonics Symposium (IUS), Venice, Italy
**Year:** 2022
**DOI:** 10.1109/IUS54386.2022.9958156

**How WULPUS was used:** The foundational paper. Presents the complete WULPUS system design (46×25 mm, 9 g, <25 mW, 8 time-multiplexed channels, 50 Hz frame rate, BLE 320 kbps). Demonstrates two use cases: (1) carotid artery waveform acquisition — the probe was positioned over the carotid and continuous A-mode traces captured the arterial wall displacement waveform, enabling heart rate and blood pressure waveform monitoring; (2) muscle activity monitoring — probe placed over the forearm tracked muscle thickness changes during contraction. Establishes the power budget breakdown showing how 22 mW is achieved.

---

### 2. A Wearable Ultra-Low-Power sEMG-Triggered Ultrasound System for Long-Term Muscle Activity Monitoring
**Authors:** Sebastian Frey, Victor Kartsch, Christoph Leitner, Andrea Cossettini, Sergei Vostrikov, Simone Benatti, Luca Benini
**Venue:** 2023 IEEE International Ultrasonics Symposium (IUS), Montreal, Canada
**Year:** 2023
**arXiv:** 2309.06851
**DOI:** 10.1109/IUS51837.2023.10307041

**How WULPUS was used:** WULPUS was integrated with the BioGAP sEMG wearable platform. Surface EMG signals detected muscle contractions and triggered WULPUS acquisitions only when muscle activity was present, suppressing idle acquisitions. Result: 59% power reduction — from 29.8 mW (continuous 50 Hz) to 12.2 mW (sEMG-triggered). Application: monitoring the transverse abdominis (deep abdominal muscle) during breathing exercises for physiotherapy biofeedback. Demonstrates that WULPUS's nRF52832 firmware can receive external trigger signals to gate acquisition.

---

### 3. Complete Cardiorespiratory Monitoring via Wearable Ultra-Low Power Ultrasound
**Authors:** Sergei Vostrikov, Luca Benini, Andrea Cossettini
**Venue:** 2023 IEEE International Ultrasonics Symposium (IUS), Montreal, Canada
**Year:** 2023
**DOI:** 10.1109/IUS51837.2023.10307166

**How WULPUS was used:** A single WULPUS probe placed on the chest simultaneously captured both cardiac and respiratory signals from the same A-mode trace. Signal processing separated the two physiological signatures by frequency band. Achieved <3.9% error in heart rate and <4.6% error in respiration rate compared to ECG and manual reference counting. System power: 17 mW — lower than the standard 22 mW figure because PRF was reduced from 50 Hz to ~17 Hz (respiratory motion is slower than cardiac). Projected 7+ days of operation on a smartwatch-class battery.

---

### 4. Hand Gesture Recognition via Wearable Ultra-Low Power Ultrasound
**Authors:** Sergei Vostrikov, Andrea Cossettini, Luca Benini
**Venue:** 2023 IEEE International Ultrasonics Symposium (IUS), Montreal, Canada
**Year:** 2023
**DOI:** 10.1109/IUS54521.2023.10307059

**How WULPUS was used:** A WULPUS armband with four ultrasound transducers was placed around the forearm. Each transducer captured an A-mode trace capturing muscle deformation patterns during hand gestures. XGBoost gradient-boosted tree classifiers were trained on the raw time-domain features. Result: 97% average cross-validated accuracy on four gestures (fist, open hand, pinch, point), with 3% inter-session variability. Total system power: 16 mW. Establishes WULPUS as viable for gesture-based human-machine interfaces at wearable power levels.

*Note: Phase 4 of this research cites 92% accuracy — the 97% figure is from the 2023 IUS paper with a refined feature extraction pipeline. Both figures refer to WULPUS gesture recognition.*

---

### 5. Tracking of Wrist and Hand Kinematics with Ultra Low Power Wearable A-Mode Ultrasound
**Authors:** Giusy Spacone, Sergei Vostrikov, Victor Kartsch, Simone Benatti, Luca Benini, Andrea Cossettini
**Venue:** IEEE Transactions on Biomedical Circuits and Systems, Vol. 19, No. 3, pp. 536–548
**Year:** Published June 2025 (accepted September 2024)
**DOI:** 10.1109/TBCAS.2024.3454563

**How WULPUS was used:** A four-transducer WULPUS armband tracked continuous hand-wrist kinematics using regression models (not classification). Output: 3 degrees of freedom (wrist flexion/extension, radial/ulnar deviation, forearm pronation/supination). Performance: RMSE 7.32° ± 1.97°, MAE 5.31° ± 1.42°. Validated against optical motion capture. Notably addresses transducer repositioning robustness — the model was tested across sessions where the armband was removed and reapplied, a critical requirement for practical wearable use. Five subjects.

---

### 6. Wearable and Ultra-Low-Power Fusion of EMG and A-Mode Ultrasound for Hand-Wrist Kinematic Tracking
**Authors:** Giusy Spacone, Sebastian Frey, Oreste Orlandi, Silvia Rapa, Victor Kartsch, Simone Benatti, Luca Benini, Andrea Cossettini
**Venue:** Preprint (arXiv)
**Year:** 2025
**arXiv:** 2510.02000

**How WULPUS was used:** Multimodal fusion of WULPUS ultrasound and surface EMG for hand-wrist tracking. Compared ultrasound-only, EMG-only, and fused models. Fusion improved accuracy in ambiguous motion states (where one modality alone is unreliable) and reduced power by selectively using EMG (lower power) when the motion signal is unambiguous. Extends the approach of Paper #2 (sEMG triggering) into full kinematic regression.

---

### 7. PuLsE: Accurate and Robust Ultrasound-based Continuous Heart-Rate Monitoring on a Wrist-Worn IoT Device
**Authors:** Sergei Vostrikov, Andrea Cossettini, Luca Benini
**Venue:** arXiv preprint; also submitted to IEEE journal (document 11050898 on IEEE Xplore)
**Year:** 2024
**arXiv:** 2410.16219

**How WULPUS was used:** WULPUS probe placed on the wrist (over the radial artery) for continuous heart rate monitoring. A signal processing pipeline (PuLsE) extracted cardiac waveforms from the ultrasound A-mode trace. Achieved Pearson correlation r = 0.99 with ECG and mean error of 0.69 ± 1.99 bpm. Open-source dataset and processing code released on GitHub. Establishes wrist-worn WULPUS as competitive with PPG-based heart rate monitors (while providing richer information — raw pressure waveform, not just pulse timing).

---

## WULPUS-Pro (Next Generation — Referenced but not yet public)

**Status:** Referenced in papers and in the WULPUS GitHub README as a follow-on platform. No public repository found as of 2026-06-21.

**Cited improvements over WULPUS:**
- 16 channels (vs 8)
- +30V unipolar pulser (vs +15V)
- Programmable TGC (vs fixed 40.8 dB)
- CMUT transducer bias support (±30V)
- Extended frequency range: 100 kHz – 10 MHz

**ModulUS (related platform):** A sandbox wearable ultrasound development platform from the same group, presented at IEEE IUS 2024. Features 32-channel pulser, analog frontend with bandwidth reduction, ARM Cortex-M4 MCU. Lowest published power normalized per MHz. Not the same device as WULPUS but shares the low-power wearable philosophy.

---

## External Research Uses

### ETH Zurich — Vermon 32-Channel Array Adapter
GitHub issue #16 records "multiple requests" for a Vermon 32-channel array adapter PCB to be included in the WULPUS repository. This indicates that at least some external groups are attempting to connect WULPUS to Vermon linear array probes — moving from single-element A-mode toward 2D B-mode imaging with the same front-end. This use case is not yet officially documented or supported but represents an active experimental direction.

### DIY Community Builders
Discussion threads document at least two independent labs building custom WULPUS boards from scratch (Daalvarenga's lab, Ethan010722). Both confirmed successful assembly and acquisition after resolving bring-up issues. No specific affiliation disclosed in the GitHub threads.

---

## IEEE UFFC Newsletter Feature

**Title:** "WULPUS — An Open-Source Wearable Ultra-Low Power UltraSound Probe"
**Source:** IEEE Ultrasonics, Ferroelectrics, and Frequency Control Society Newsletter
**URL:** https://ieee-uffc.org/post/news/wulpus-open-source-wearable-ultra-low-power-ultrasound-probe

The IEEE UFFC Society featured WULPUS as a notable open-source hardware contribution to the ultrasound community. This is significant: IEEE UFFC is the primary professional society for ultrasound engineering, and a newsletter feature signals that the design is considered a meaningful contribution to the field, not just a student project.

---

## Summary Table

| Paper | Year | Venue | Application | Key result |
|-------|------|-------|-------------|------------|
| WULPUS system paper (Frey et al.) | 2022 | IEEE IUS | Carotid + muscle monitoring | <25 mW, 2.5+ days, 8 ch |
| sEMG-triggered acquisition (Frey et al.) | 2023 | IEEE IUS | Muscle monitoring (biofeedback) | 59% power reduction (12.2 mW) |
| Cardiorespiratory monitoring (Vostrikov et al.) | 2023 | IEEE IUS | Heart rate + respiration, single probe | <3.9% HR error, 17 mW |
| Gesture recognition (Vostrikov et al.) | 2023 | IEEE IUS | Hand gesture HMI | 97% accuracy, 16 mW |
| Wrist/hand kinematics (Spacone et al.) | 2024/2025 | IEEE TBioCAS | 3-DOF kinematic regression | RMSE 7.32°, repositioning-robust |
| EMG+US fusion (Spacone et al.) | 2025 | arXiv | Kinematic tracking, multimodal | Improved accuracy + power |
| Heart rate, wrist-worn (Vostrikov et al.) | 2024 | arXiv/IEEE | Continuous heart rate | r=0.99 vs ECG, open dataset |

---

## Research Themes

Across all documented uses, three application themes dominate:

1. **Physiological monitoring** (carotid, heart rate, respiration): WULPUS used as a wearable sensor that replaces or supplements ECG/PPG for cardiovascular parameters. Key advantage over ECG: ultrasound can measure arterial wall displacement and flow, not just electrical activity.

2. **Musculoskeletal imaging** (muscle thickness, deep muscle tracking): WULPUS used to measure changes in muscle cross-section and depth during contraction. Key advantage over surface EMG: EMG only detects electrical activation; ultrasound captures the mechanical deformation directly.

3. **Human-machine interface** (gesture recognition, kinematic tracking): WULPUS armband used as a limb-position and motion decoder. Key advantage over IMU/accelerometer: ultrasound sees through skin to muscle and tendon deformation, enabling higher-fidelity motion capture at low power.

All published use cases operate in the 1–2 MHz transducer frequency range and at imaging depths of 20–60 mm. None use the device for NDT, structural inspection, or materials characterization — these are exclusively biomedical applications.
