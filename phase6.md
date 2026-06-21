# Phase 6: Synthesis

*A consolidated view across all five preceding phases. This section is the deliverable — everything before it is the evidence.*

---

## 6.1 Master Comparison Table

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| **HARDWARE** | | |
| ADC chip | ADC10065 (external) | MSP430FR5043 SDHS (integrated) |
| Sampling rate | 65 Msps | 8 Msps |
| ADC resolution | 10 bits (ENOB 9.6) | 12 bits (ENOB unpublished) |
| Amplifier | AD8331 VGA (TGC) | MSP430 PGA + external op-amp |
| Gain range | 7.5–55.5 dB (48 dB, variable) | 40.8 dB (fixed) |
| Gain steps | Continuous (12-bit DAC) | 46 discrete steps |
| Pulser topology | Three-level bipolar ±24V | Unipolar +15V |
| TX channel count | 1 (base) / 8 with PMOD mux | 8 time-multiplexed |
| Pulser location | Separate PMOD board | Integrated HV PCB |
| T/R switch | MD0100N8-G (passive) | SPI-controlled HV mux |
| Digital controller | RP2040 (PIO state machines) | MSP430FR5043 USS_A peripheral |
| Secondary MCU | — | nRF52832 (BLE bridge) |
| Tertiary MCU | — | nRF52840 USB dongle |
| IMU | — | IIS2DH (on-board) |
| Connectivity | USB (full-speed/high-speed) | BLE 320 kbps + USB dongle |
| Throughput ceiling | USB bandwidth (>>1 Mbps) | 320 kbps (BLE) |
| **POWER** | | |
| System power (measured) | Not published | 22 mW at 50 Hz PRF |
| System power (estimated) | ~300–400 mW | — |
| Battery operation | No | Yes |
| Battery life (350 mAh) | N/A | >2.5 days |
| **FORM FACTOR** | | |
| Weight | Not specified | 9 g |
| PCB dimensions | Multi-board stack | 46 × 25 mm (2 boards) |
| Enclosure | Open dev board | Silicone mold (waterproof option) |
| Wearable | No | Yes |
| **PERFORMANCE** | | |
| Max transducer freq. | ~30 MHz | ~2–4 MHz |
| Axial res./sample (tissue) | ~11.8 µm | ~96 µm |
| PRF (max) | Unspecified | 50 Hz |
| Samples/acquisition | 8000 (DMA buffer) | 400 |
| SNR (system, measured) | Not published | Not published |
| **SOFTWARE** | | |
| Firmware language | C + PIO assembly | C (3 separate projects) |
| Toolchains | 1 (Pico SDK) | 3 (TI CCS, nRF SDK, nRF SDK v2) |
| Host language | Python | Python |
| Real-time visualization | No | Yes (ipywidgets) |
| Real-time filtering | No | Yes (31-tap FIR, Hilbert) |
| Data format | HDF5 (compressed, keyed) | NumPy .npz (sequential) |
| B-mode support | Separate repo | Built-in GUI |
| TGC control API | `pic.dac(N)` (continuous) | N/A (fixed gain) |
| Channel config | Serial command | 16-slot TX/RX config table |
| **OPENNESS** | | |
| Hardware license | TAPR OHL (copyleft) | Solderpad v0.51 (permissive) |
| Software license | GPLv3 | Apache 2.0 |
| PCB tool | KiCad (free) | Altium Designer (~$10k/yr) |
| OSHWA certified | Yes (FR000023) | No |
| Pre-built available | Yes (Tindie) | No |
| **COST** | | |
| DIY system cost | ~$100–$200 | ~$250–$500 |
| Pre-built cost | ~$200–$400 (Tindie) | Not available |
| EDA tool cost | Free | ~$10k/yr (or PDF/Gerber only) |
| **COMMUNITY** | | |
| GitHub stars | 71 | 110 |
| GitHub forks | 26 | 26 |
| Contributors | 1 (kelu124) | 3 (ETH Zurich IIS) |
| Institutional backing | Individual | ETH Zurich IIS / PULP group |
| Community channel | Matrix chat | GitHub Discussions |
| IEEE publications | 1 (Zenodo) | 5+ |
| Last commit | June 8, 2026 | June 21, 2026 |

---

## 6.2 Per-Device Summary Cards

### pic0rick

**What it is:** A single-board USB pulse-echo ultrasound platform for lab-based NDT and academic experimentation. Designed to be inexpensive, hackable, and compatible with a wide range of transducers.

**Built for:** Benchtop use. Requires a laptop and USB cable. The target user is someone who wants to quickly prototype an ultrasound measurement with a Python notebook and a transducer they already own.

**Core technical identity:** 65 Msps ADC + full TGC + RP2040 PIO. The combination of high sampling rate and programmable gain allows it to work with virtually any common ultrasound transducer (1–30 MHz) and properly compensate for depth attenuation.

**Where it excels:**
- Transducer frequency range: nothing else at this price point reaches 30 MHz
- TGC flexibility: 12-bit DAC, 48 dB range, configurable from Python
- Experiment scripting: Python API with numpy arrays, HDF5 logging, Jupyter notebooks
- Reproducibility: KiCad files, JLCPCB-ready BOMs, Tindie availability, OSHWA certified

**Where it falls short:**
- USB-tethered: no mobile, wearable, or ambulatory applications
- Single channel at base (8-ch mux board adds complexity)
- Power draw makes battery operation impractical (~300–400 mW estimated)
- Single maintainer: bus factor 1

---

### WULPUS

**What it is:** A wearable 8-channel BLE-connected ultrasound probe designed for continuous physiological monitoring. Built around the MSP430FR5043's integrated ultrasound sequencer and optimized relentlessly for power.

**Built for:** On-body, cable-free deployment. The target user is a biomedical researcher who needs to collect continuous ultrasound data (carotid, muscle, heart) from a subject who is moving.

**Core technical identity:** MSP430 USS_A + nRF52832 BLE + dual-MCU power gating = 22 mW system power. Every design decision is downstream of the wearable power constraint.

**Where it excels:**
- Power: 22 mW measured, >2.5 days on 350 mAh battery — genuine wearable operation
- Wireless: BLE at 320 kbps, no cable to subject
- Form factor: 9g, 46×25 mm, silicone mold available
- Multi-channel: 8 TX/RX channels, 16-slot config table, usable for synthetic aperture
- Published validation: carotid artery, muscle tracking, gesture recognition (92%), EMG-gated imaging

**Where it falls short:**
- 8 Msps ceiling: limited to ~2–4 MHz transducers; excludes clinical probes, NDT, vascular imaging
- Fixed gain: no TGC, unsuitable for depth-varying signals
- Firmware complexity: 3 MCUs, 3 toolchains, C throughout — high barrier to modification
- Altium lock-in: PCB source files inaccessible without expensive license
- BLE ceiling: 50 Hz PRF at 400 samples/frame is near-saturation; no headroom

---

## 6.3 Key Differentiating Design Decisions

These are the decisions that account for most of the difference between the two devices. Each one is a genuine tradeoff — not a mistake by one side.

### 1. ADC rate: 65 Msps vs 8 Msps

**pic0rick chose:** External 65 Msps ADC (ADC10065), 10-bit, ENOB 9.6.
**WULPUS chose:** Integrated SDHS ADC on MSP430FR5043, 8 Msps, 12-bit.

**Consequence:** pic0rick's ADC chip alone consumes 68.4 mW — more than three times WULPUS's entire system. But it supports transducers up to 30 MHz and achieves ~11.8 µm axial resolution per sample in soft tissue. WULPUS's ADC limits compatible transducers to ~2–4 MHz with ~96 µm axial resolution. This is the single most consequential difference.

**Why WULPUS chose this:** The 8 Msps limit is set by the MSP430FR5043 USS_A peripheral — the same component that enables integrated power management, sequencing, and PGA. Gaining the power efficiency required accepting the sampling rate constraint.

### 2. TGC vs fixed gain

**pic0rick chose:** AD8331 VGA (time-gain compensation) controlled by MCP4812 12-bit DAC.
**WULPUS chose:** MSP430 PGA with 46 fixed gain steps + external power-gated op-amp = fixed 40.8 dB.

**Consequence:** TGC is essential for imaging (depth compensation). Fixed gain is acceptable for monitoring a single anatomical target at a known, stable depth. WULPUS's IEEE papers exclusively show monitoring use cases (carotid position tracking, muscle thickness); pic0rick's demos show B-mode imaging and NDT thickness gauging.

**Why WULPUS chose this:** Adding a DAC-controlled VGA would require additional power budget, an extra SPI peripheral, and careful noise management. The 22 mW target made this infeasible. WULPUS-Pro (successor) adds programmable TGC — confirming the ETH Zurich team considered it necessary for future work.

### 3. Wireless BLE vs USB tether

**pic0rick chose:** USB connection to host PC. The RP2040 USB controller is used for both power and data.
**WULPUS chose:** BLE via nRF52832 + nRF52840 dongle + USB gateway chain.

**Consequence:** USB gives pic0rick unconstrained data throughput and simplicity. BLE gives WULPUS cable-free operation but caps throughput at 320 kbps and constrains PRF. The BLE choice is inseparable from the wearable goal: you cannot wear a USB tether.

### 4. Single MCU vs dual-MCU power gating

**pic0rick chose:** RP2040 handles all tasks: PIO acquisition, DMA, USB transfer, host communication.
**WULPUS chose:** MSP430 handles acquisition; nRF52832 handles BLE. Each powers down when idle.

**Consequence:** The dual-MCU split lets each processor enter its optimal low-power standby mode independently (MSP430: 450 nA; nRF52832: radio off between bursts). A single MCU doing both would keep the radio active during acquisition or require complex duty-cycling. The price: two different C toolchains, two different SDKs, and SPI inter-MCU communication.

### 5. Modular (PMOD) vs integrated design

**pic0rick chose:** Main ADC board + separate pulser PMOD + separate HV board. Expandable via PMOD connectors.
**WULPUS chose:** Acquisition PCB + HV PCB — a fixed, optimized two-board stack.

**Consequence:** pic0rick's modularity makes experimentation easy (swap in different mux, PSRAM, display boards) but increases assembly complexity and connection count. WULPUS's fixed design is optimized for the wearable use case but harder to modify.

### 6. Open toolchain vs integrated IDE

**pic0rick chose:** RP2040 + Pico SDK (open source, CMake, free, cross-platform) + KiCad (free).
**WULPUS chose:** MSP430 + TI CCS + nRF SDK + Altium Designer.

**Consequence:** Anyone can build, modify, and understand pic0rick's firmware and hardware without purchasing proprietary tools. WULPUS requires TI Code Composer Studio (free download but closed), Nordic nRF SDK (open source but complex), and Altium (~$10k/yr) for PCB changes. PDF/Gerber exports make WULPUS replicable without Altium but not modifiable.

---

## 6.4 Gap Analysis

These are real gaps in either the hardware, documentation, or ecosystem of each device — not design choices but missing pieces.

| Gap | Device | Impact | Notes |
|-----|--------|--------|-------|
| System SNR not measured | Both | Makes receiver dynamic range comparison impossible | Expected for research platforms |
| Axial/lateral resolution not measured on phantom | Both | Can't compare imaging quality directly | Would require a calibrated US phantom |
| pic0rick power draw not measured | pic0rick | Rough estimate only (~300–400 mW) | Component datasheets give per-chip estimates |
| WULPUS ADC ENOB not published | WULPUS | MSP430FR5043 USS_A ADC dynamic range unknown | Not in accessible datasheet |
| WULPUS max PRF untested above 50 Hz | WULPUS | BLE ceiling confirmed at 50 Hz, not beyond | Would saturate link |
| pic0rick PRF not specified | pic0rick | Can't compare acquisition rate capability | Depends on pulse parameters + USB transfer time |
| pic0rick multi-element array imaging undocumented | pic0rick | MAX14866 PMOD exists but no B-mode array demo published | us_rf_processing handles single-element data |
| WULPUS imaging depth not published | WULPUS | Theoretical ~60 mm at 2 MHz, not validated | Would require experimental measurement |
| WULPUS HV mux part number unconfirmed | WULPUS | Firmware reveals SPI 16-bit shift register; chip ID unknown from available sources | Altium BOM is binary-only |
| pic0rick Tindie price not confirmed precisely | pic0rick | Listed as <$500; specific unit price not confirmed | Page accessible but price varies |
| WULPUS-Pro not yet public | WULPUS | Successor with TGC, 16 channels, ±30V referenced in papers but no public repo | ETH Zurich, no release date |

---

## 6.5 Cross-Pollination Notes

Observations about what each project could adopt from the other — if a follow-on version were to extend beyond current scope.

### What WULPUS could adopt from pic0rick

**TGC.** WULPUS-Pro already plans this. Fixed gain is the most functionally limiting constraint for any application beyond fixed-depth monitoring. Even coarse TGC (4-step) would expand usable depth range significantly.

**KiCad as secondary export target.** The Altium source is not going away, but providing a KiCad-compatible export (or a FreeCAD/KiCad layout of the critical traces) would enable community modifications without a $10k/yr license. PDF/Gerber exports allow replication but not editing.

**OSHWA certification.** The WULPUS open-source posture (SHL-0.51 hardware, Apache 2.0 software) would qualify for OSHWA certification. Certification costs are low and would increase discoverability and credibility within the open hardware community.

**HDF5 data storage.** WULPUS's sequential .npz files are simple but lack metadata. pic0rick's HDF5 approach with acquisition keys, timestamps, gain values, and pulse parameters would make long monitoring sessions more navigable.

### What pic0rick could adopt from WULPUS

**Multi-channel acquisition with independent TX/RX config table.** WULPUS's 16-slot TX/RX config table is architecturally clean. pic0rick's MAX14866 PMOD achieves channel switching but with simpler control logic. A config-table approach would make multi-element array imaging more systematic.

**IMU-gated acquisition.** The IIS2DH accelerometer in WULPUS enables motion rejection — acquisitions are automatically flagged or gated based on subject motion. For any NDT or clinical application where motion artifacts are a concern, this is a meaningful addition.

**BLE as an optional wireless mode.** Adding a BLE PMOD (nRF52832 or similar) would allow pic0rick to operate cable-free for lower-throughput monitoring use cases. At 50 Hz PRF and 400 samples/acquisition, a BLE link is sufficient — and the RP2040 has SPI available for the radio connection.

**Silicone enclosure.** The WULPUS moldable silicone enclosure makes the probe genuinely wearable and water-resistant. pic0rick's open dev board form factor is fine for bench use but would benefit from an enclosure for field NDT deployment.

---

## 6.6 Choosing Between Them

| If you need… | Choose |
|--------------|--------|
| Transducer frequency > 4 MHz | **pic0rick** |
| B-mode imaging with any standard probe | **pic0rick** |
| TGC (depth-variable gain) | **pic0rick** |
| NDT thickness gauging | **pic0rick** |
| Maximum raw data throughput | **pic0rick** |
| Cheap entry point (< $200 DIY) | **pic0rick** |
| Full hardware modifiability (KiCad) | **pic0rick** |
| Python experimentation in a notebook | **pic0rick** |
| Battery-powered / cable-free operation | **WULPUS** |
| Wearable / body-worn deployment | **WULPUS** |
| Multi-day continuous monitoring | **WULPUS** |
| 8-channel simultaneous monitoring | **WULPUS** |
| Ambulatory physiological monitoring | **WULPUS** |
| Low-frequency tissue monitoring (≤ 2 MHz) | **WULPUS** |
| Real-time streaming GUI out of the box | **WULPUS** |
| Academically validated biomedical use | **WULPUS** |

The devices are not in competition: there is no realistic use case where both would work equally well. pic0rick's 65 Msps and TGC are irrelevant for wearable monitoring; WULPUS's 22 mW and BLE are irrelevant for benchtop NDT.
