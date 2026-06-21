# Phase 4: Strengths, Weaknesses, and Specificities

*A per-device characterization — not comparative ranking. The goal is to state clearly what each device is genuinely good at, where it has real limits, and what makes it distinct.*

---

## 4.1 pic0rick — Strengths

### High sampling rate (65 Msps) with wide transducer compatibility

The ADC10065 at 65 Msps is the defining characteristic of pic0rick. It supports transducer center frequencies up to approximately 30 MHz (practical, with margin for aliasing avoidance), covering the full range of common clinical and NDT probes: 1 MHz (large-area NDT), 2.25 MHz (general NDT), 5 MHz (near-surface NDT, general B-mode), 10–15 MHz (high-resolution medical), and higher. This is not achievable with any 8 Msps system regardless of other optimizations.

Axial resolution per sample in soft tissue: ~11.8 µm — well below the acoustic wavelength for any transducer below 100 MHz. In practice, pic0rick's temporal resolution does not limit the achievable axial resolution for any transducer likely to be connected.

### Full TGC — variable gain over the receive window

The AD8331 + MCP4812 DAC combination enables time-gain compensation: the amplifier gain can be increased progressively through the receive window to equalize signals from different depths. This is essential for imaging applications where reflector amplitude varies by many dB across the image depth. The 48 dB gain range (7.5–55.5 dB) and 12-bit DAC (4096 gain steps) give fine-grained control over the compensation curve.

### Modular PMOD architecture — extensible by design

The double and single PMOD connectors expose analog, digital, and power signals. This has already been used for: MAX14866 mux board (8-channel array switching), PSRAM board (large sample buffer), VGA board (real-time display). Any PMOD-compatible peripheral can be added. This makes pic0rick a genuine experiment platform: the hardware is a scaffold, not a fixed product.

### Python-first, notebook-driven workflow

pic0lib provides a clean Python API with numpy arrays, scipy-compatible data, HDF5 storage, and Jupyter notebooks as the development environment. Prototyping a new measurement protocol requires only Python scripting, no firmware recompilation. The separation of firmware (acquisition) from software (analysis) is clean and well-designed.

### Open hardware certified, commercially available

OSHWA FR000023 is a documented external certification. The device is available pre-built on Tindie with KiCad source files, so reproduction and modification follow an accessible path. The PCB tool (KiCad) is free and cross-platform.

### Part of established ecosystem with prior published work

pic0rick is the current board in the un0rick lineage, which has published use cases, a Zenodo paper, and integration with the broader open-source ultrasound community (opensourceimaging.org). The `us_rf_processing` repo provides a beamforming and B-mode pipeline that works with the same data format.

---

## 4.2 pic0rick — Weaknesses and Limits

### USB-tethered: no battery or wireless operation

The device is strictly powered and connected via USB. Power consumption is estimated at 300–400 mW — the ADC chip alone draws 68 mW, and the AD8331 adds ~100–150 mW. There is no path to battery operation without significant hardware redesign. This makes pic0rick unsuitable for any wearable, ambulatory, or field-deployed application where a cable to a laptop is not acceptable.

### Single channel in the base configuration

Without the optional MAX14866 PMOD board, pic0rick is single-channel: one transmit, one receive. Multi-element array imaging, synthetic aperture, or multi-site monitoring require adding the mux board and writing additional control logic. Even with the mux board, the maximum channel count is 8, and channel switching is controlled from the host PC (not deterministically timed from hardware).

### Power consumption not characterized as a system

No published figure exists for the total power draw of the pic0rick system during acquisition. The estimate of 300–400 mW is derived from individual component datasheets and is not verified. This makes power-related comparisons uncertain.

### Not a medical-grade device

pic0rick carries no IEC 60601, CE, FDA, or equivalent regulatory marking. It explicitly targets NDT, educational, and academic use. This is a stated design constraint, not an oversight, but it limits the device from any clinical or patient-contact application.

### Single-author project

pic0rick has one contributor (kelu124). This concentrates the maintenance burden and creates a bus-factor risk. The GitHub discussions are disabled; community support is routed through a Matrix chat room.

---

## 4.3 pic0rick — Specificities

### PIO-based ultrasound timing: flexible but unconventional

The use of RP2040 PIO state machines for pulse-echo timing is architecturally novel. PIO provides cycle-accurate GPIO control independent of the CPU, with deterministic latency. This means: (a) pulse timing can be set to nanosecond resolution via configuration parameters without recompiling firmware, (b) multiple synchronized state machines handle transmit and receive simultaneously, (c) DMA feeds the CPU-side buffer without interrupts. However, PIO programs have limited instruction sets (32 instructions per state machine) and no floating-point, so complex timing sequences require careful PIO assembly programming.

The practical consequence for users: pulse width, damping, and PRF are freely configurable from the Python host. Changing pulse shape (e.g., adding a coded excitation sequence) would require editing PIO assembly — a higher barrier.

### Three-level bipolar pulser on a PMOD

The transmit path (MD1213 + TC6320 on a separate PMOD) produces three voltage levels: +24V, 0V, −24V. This is the standard approach for broadband ultrasound: the bipolar swing maximizes acoustic energy per pulse and symmetric transducer loading. Placing the pulser on a separate PMOD board is an unusual modularity choice — it means the HV circuit is physically isolated from the receive chain on the main board, which reduces direct HV-to-analog coupling noise. It also means the main board can be used for signal characterization without the pulser connected.

---

## 4.4 WULPUS — Strengths

### Ultra-low power: 22 mW measured, multi-day battery life

22 mW is a measured figure from the IEEE paper, not a datasheet estimate. On a 350 mAh battery at 3.7V (1.295 Wh), this yields approximately 59 hours of continuous operation — more than 2.5 days. This is not "low power for a lab device" but genuinely wearable power consumption, comparable to consumer fitness trackers. No other open-source ultrasound platform approaches this.

### 8 time-multiplexed channels with independent TX/RX configuration

Each acquisition cycle can be configured with up to 16 different TX/RX channel pairs via the firmware's config table. This enables: simultaneous multi-site monitoring (carotid + respiration), multi-element array probes with synthetic aperture, single-transmit/multi-receive (STMR) configurations, and sequential element-by-element firing. The RX and TX channel configurations are independent (different switches), enabling cross-element coupling measurements.

### Wireless BLE, untethered operation

The nRF52832 BLE link at 320 kbps enables acquisition while the subject is mobile. This is essential for the documented use cases: ambulatory carotid monitoring, gesture recognition during hand movement, EMG-coupled muscle monitoring. No cable runs from the body to a laptop.

### Compact, wearable form factor

9g, 46×25 mm (approximately the size of a stack of two thumbnails) is genuinely body-wearable. The silicone package (mold files in the repo) holds the two PCBs as a waterproof unit that can be strapped to the forearm, neck, or limb. No comparable open-source ultrasound platform fits this form factor.

### Published biomedical validation results

Multiple IEEE papers document WULPUS in actual experimental conditions: carotid artery waveform capture, muscle thickness tracking, concurrent heart rate and respiration from a single probe, hand gesture classification (92% accuracy), EMG-gated muscle imaging, sEMG-triggered ultrasound (59% power reduction via triggering). These are real experimental results, not simulations.

### Active multi-contributor research group

3 contributors, GitHub Discussions enabled, active today (last commit 2026-06-21). The ETH Zurich IIS/PULP group publishes regularly and maintains the repo as a research platform for ongoing work.

---

## 4.5 WULPUS — Weaknesses and Limits

### 8 Msps ADC limits usable transducer frequency range

The 8 Msps SDHS ADC constrains the maximum transducer center frequency to approximately 2–4 MHz (below the Nyquist limit of 4 MHz, with practical bandwidth margin). This excludes: standard clinical B-mode probes (5–15 MHz), NDT probes (2.25–10 MHz), vascular imaging probes (5–10 MHz), and any high-resolution application. The device is effectively limited to low-frequency A-mode monitoring applications (deep tissue at low resolution).

### Fixed gain — no TGC

40.8 dB fixed gain cannot be changed at runtime. There is no depth compensation: reflectors at 1 cm and reflectors at 5 cm are amplified identically. For monitoring a specific anatomical structure at a known, consistent depth (e.g., carotid artery position is stable across a monitoring session), this is workable. For imaging applications where depth varies or comparison across depths is needed, fixed gain is a significant limitation. (Note: WULPUS-Pro addresses this.)

### BLE 320 kbps is the throughput ceiling

At 50 Hz PRF with 400 samples/acquisition, the BLE link operates near its throughput limit. Increasing PRF or sample count would saturate the link. The WULPUS is fundamentally a streaming device limited by BLE throughput — it cannot deliver high-duty-cycle raw data. It operates at ~0.8% of the raw ADC data rate after framing.

### C toolchain across three separate MCUs

Building WULPUS firmware requires three separate toolchains: TI Code Composer Studio (MSP430), Nordic nRF SDK (nRF52832), and Nordic nRF SDK v2 (nRF52840 dongle). Each has its own IDE, HAL, and build system. Modifying the acquisition sequence (e.g., adding a new timing parameter) requires changing C code in the MSP430 firmware, rebuilding, and flashing. This is a substantially higher barrier than pic0rick's Python-configurable interface.

### Altium PCB source files

The primary PCB design files are in Altium Designer format (.PcbDoc, .SchDoc). Altium licenses cost thousands of dollars per year. While PDF schematics and Gerber files are provided (enabling replication without Altium), modifying the hardware design requires either Altium access or converting to another EDA tool. pic0rick uses KiCad (free and open source), which is directly importable and modifiable by anyone.

### ENOB and system-level SNR not published

No measured system-level SNR or ENOB figure is available for WULPUS. The MSP430FR5043 SDHS ADC ENOB is not specified in accessible documentation. This makes it impossible to assess the receiver's actual dynamic range.

---

## 4.6 WULPUS — Specificities

### MSP430FR5043 USS_A peripheral: integrated ultrasound sequencer

The MSP430FR5043 is one of a small family of MCUs with dedicated ultrasound hardware. The USS_A module integrates the entire signal path: PPG (pulse generator with configurable frequency, duty cycle, count), PGA (46-step programmable gain), SDHS ADC (12-bit, 8 Msps), and a DMA engine — all sequenced automatically from a single trigger. The MSP430 FRAM (64 KB) stores the ADC data without requiring external PSRAM. This level of integration is what makes the <25 mW system power possible. The tradeoff: the USS_A is designed for flow metering applications (its primary target market), so its operating envelope (8 Msps max, limited bandwidth) reflects those requirements rather than general-purpose ultrasound imaging.

### Dual-MCU split with explicit power gating

The decision to use two MCUs (MSP430 for acquisition, nRF52832 for BLE) rather than a single MCU with integrated BLE is a deliberate power optimization. The nRF52832's BLE radio is power-gated between transmission bursts. The MSP430 enters low-power standby (450 nA) between acquisition cycles. A single MCU doing both roles would keep the radio active during acquisition (increasing power) or require complex duty-cycling logic. The dual-MCU split trades firmware complexity for cleaner power domain separation.

The IIS2DH IMU on the acquisition PCB adds motion artifact detection capability — not just an accessory, but a functional requirement for wearable ultrasound where body movement corrupts acquisition.

---

## 4.7 Published Performance Benchmarks

| Benchmark | pic0rick | WULPUS |
|-----------|----------|--------|
| System power (measured) | Not published | 22 mW at 50 Hz PRF |
| Battery life (measured) | N/A | >2.5 days (350 mAh) |
| BLE throughput (measured) | N/A | 320 kbps |
| Carotid artery waveform | Not demonstrated | Yes (IEEE EMBC 2022) |
| Muscle thickness tracking | Not demonstrated | Yes (published) |
| Gesture recognition | Not demonstrated | 92% accuracy (published) |
| NDT imaging | Yes (steel blocks, notebook demos) | Not demonstrated |
| B-mode imaging | Yes (us_rf_processing repo) | Basic (8-ch synthetic aperture in GUI) |
| SNR (measured, in situ) | Not published | Not published |
| Axial resolution (measured) | Not published | Not published |
