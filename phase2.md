# Phase 2: Hardware Performance Characterization

*All figures from datasheets, published papers, and verified manufacturer specs. Type column: M = measured/experimental, D = design/datasheet spec, C = calculated from specs, E = estimated.*

---

## Master Performance Comparison Table

| Parameter | pic0rick | WULPUS | Source type |
|-----------|----------|--------|-------------|
| **ADC sampling rate** | 65 Msps | 8 Msps | D |
| **ADC resolution** | 10 bits | 12 bits | D |
| **ADC ENOB** | 9.6 bits | Not published | D / — |
| **ADC SNR (component)** | 59.6 dB (at 11 MHz input) | Not published | D |
| **ADC input bandwidth** | 400 MHz | Not published (USS_A AFE limited) | D / — |
| **ADC power (component)** | 68.4 mW | Included in system total | D |
| **Amplifier type** | VGA/TGC (AD8331) | Integrated PGA + op-amp | D |
| **Amplifier gain range** | 7.5–55.5 dB (48 dB) | 40.8 dB fixed | D |
| **Amplifier noise figure** | 3.7 dB | Not published | D / — |
| **Amplifier bandwidth** | 120 MHz | Not published | D / — |
| **Pulser topology** | Three-level bipolar | Unipolar | D |
| **Pulser peak voltage** | ±24V (48 Vpp) | +15V | D |
| **TX channel count** | 1 | 8 (time-multiplexed) | D |
| **System power (measured)** | Not characterized | 22 mW at 50 Hz PRF | M |
| **System power (estimated)** | ~300–400 mW | — | E |
| **Battery operation** | No (USB only) | Yes | D |
| **Battery life (350 mAh)** | N/A | >2.5 days continuous | M |
| **Wireless link** | None (USB) | BLE 320 kbps | D / M |
| **Form factor** | Dev board (multi-board stack) | 46×25 mm, 9 g | D / M |
| **Compatible transducer freq.** | Up to ~30 MHz | ~2–4 MHz | C |
| **Axial res./sample (soft tissue)** | ~11.8 µm | ~96 µm | C |
| **Acquisition rate** | Not specified | Up to 50 Hz PRF | D |
| **Samples per acquisition** | Not specified | 400 | D |
| **Max PRF** | Not specified | 50 Hz | D |

---

## Notes on Uncharacterized / Missing Specs

The following performance figures were not found in any publicly accessible source for either device. They would require empirical measurement with physical hardware (phantom imaging, calibrated transducers, spectrum analyzer):

| Missing figure | pic0rick | WULPUS |
|----------------|----------|--------|
| System SNR (in situ) | Not published | Not published |
| Dynamic range (dB) | Not published | Not published |
| Imaging depth (mm, measured) | Not published | Not published |
| Axial resolution (measured on phantom) | Not published | Not published |
| Lateral resolution (measured on phantom) | Not published | Not published |
| Noise floor (dBV) | Not published | Not published |
| ADC ENOB (system-level) | Not published | Not published |
| Transducer frequency response | Transducer-dependent | Transducer-dependent |

This is expected: both are open-source research platforms, not commercial devices. Neither has undergone formal IEC 61157 or AIUM characterization. The absence of system-level measured SNR and imaging metrics is a gap in both projects, not a fault.

---

## Key Performance Insights

### 1. The ADC chip alone exceeds WULPUS total system power

The ADC10065 consumes 68.4 mW — more than three times the entire WULPUS system (22 mW). The RP2040 adds another ~40–50 mW at full speed, and the AD8331 datasheet indicates ~100–150 mW typical. The pic0rick system during acquisition likely draws 300–400 mW from USB. This is not a criticism of pic0rick — it is an intentional design tradeoff for high sampling rate and measurement flexibility.

### 2. Sampling rate gap dominates transducer compatibility

The 8× sampling rate difference (65 vs 8 Msps) is the most consequential hardware difference. Nyquist requires fs > 2 × fc. At 8 Msps, the practical usable transducer frequency range is approximately DC–4 MHz (with margin), corresponding to millimeter-scale spatial resolution in tissue. pic0rick's 65 Msps supports transducers from DC to approximately 30 MHz, covering high-resolution NDT and medical imaging probes (5–15 MHz common for clinical B-mode).

### 3. WULPUS achieves "multi-day" operation via dual-domain optimization

WULPUS reaches <25 mW by simultaneously optimizing:
- **Sampling rate** (8 vs 65 Msps — lower rate directly reduces ADC power)
- **Integration** (MSP430 USS_A — no external ADC chip with its own power budget)
- **Power gating** (op-amp disabled between acquisitions)
- **Duty cycling** (MCU enters standby between frames; MSP430 standby: 450 nA)
- **BLE** (nRF52832 active only during TX bursts)

No single optimization alone reaches the target; all must be applied together.

### 4. BLE is the WULPUS throughput ceiling

At 50 Hz and 400 samples/acquisition, WULPUS transmits 804 bytes/frame = 32 KB/s = 256 kbps of payload. With BLE protocol overhead, this approaches the 320 kbps link capacity. Increasing PRF above ~50 Hz or adding more samples per acquisition would saturate the BLE link. pic0rick has no equivalent ceiling on data throughput (USB).

### 5. WULPUS-Pro (successor) addresses several WULPUS limitations

Research from the PULP group references a "WULPUS-Pro" with upgraded capabilities:
- 16 channels (vs 8)
- +30V unipolar pulser (vs +15V)
- Programmable TGC (vs fixed gain)
- CMUT bias (±30V)
- Frequency range 100 kHz–10 MHz

This suggests the ETH Zurich team is aware of the fixed-gain and low-voltage limitations and has addressed them in a subsequent design. WULPUS-Pro was not the subject of this analysis (no public repo found at time of writing).
