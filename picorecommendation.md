# pic0rick — Strengths and Recommendations

*This document highlights what pic0rick does well and identifies concrete improvements that the WULPUS design validates as achievable.*

---

## Part 1: pic0rick's Core Strengths

### 1. Transducer compatibility range

The ADC10065 at 65 Msps is the single most consequential design choice in pic0rick. It supports transducers from 1 MHz to approximately 30 MHz — covering all common NDT probes (2.25, 5, 10 MHz), clinical B-mode probes (5–15 MHz), and high-resolution near-surface probes. No other open-source ultrasound platform at this price point comes close to this frequency range.

Axial resolution per sample in soft tissue: ~11.8 µm. The temporal resolution does not limit imaging for any transducer below 100 MHz.

### 2. Full time-gain compensation

The AD8331 VGA controlled by an MCP4812 12-bit DAC gives 48 dB of gain range (7.5–55.5 dB) in 4096 continuous steps, adjustable at runtime from Python. This is essential for any application where signal amplitude varies with depth — which is every imaging application and most NDT applications (near-surface vs back-wall echoes). TGC is what separates a measuring instrument from a monitoring device.

WULPUS (fixed 40.8 dB gain) documents exactly why fixed gain is limiting: it constrains the device to monitoring a single target at a known, stable depth.

### 3. Modular PMOD architecture

The PMOD connectors expose analog, digital, and power rails in a way that has already enabled: MAX14866 8-channel mux board, PSRAM sample buffer board, VGA real-time display board. Any PMOD-compatible peripheral can be added without redesigning the base board. This is the right architectural choice for a research platform.

### 4. Python-first, notebook-driven workflow

`pic0lib` provides a clean Python API: `pulse_adc_trigger(pon, poff, damp)` in nanoseconds, `dac(N)` for gain, `read()` returning numpy arrays. Pulse timing is configurable from the host without reflashing firmware — the RP2040 PIO programs accept parameters at runtime. Prototyping a new measurement protocol requires only Python scripting.

The HDF5 storage format (keyed, gzip-compressed, with acquisition metadata: gain, pulse params, transducer ID, timestamp) is production-quality data management for a research instrument.

### 5. Open hardware, maximum reproducibility

- KiCad source files: anyone can open, inspect, and modify without paying for EDA software
- JLCPCB-ready BOMs with pick-and-place files: outsourced assembly available at low volume
- OSHWA FR000023: externally certified open hardware
- Tindie availability: accessible without DIY assembly
- All components (ADC10065, AD8331, MCP4812, MD0100, MD1213, TC6320, Pico) are standard commercial ICs on Mouser/Digikey

WULPUS requires Altium Designer (~$10,000/year) to modify the PCB source. Anyone can build pic0rick; very few can modify WULPUS.

### 6. Three-level bipolar pulser on a separate board

The MD1213 + TC6320 combination produces ±24V (48 Vpp) bipolar pulses — the standard for broadband ultrasound. The pulser is on a separate PMOD board, physically isolating the HV switching circuit from the receive chain. This reduces HV-to-analog coupling noise and allows the main ADC board to be used for signal characterization without the pulser attached. WULPUS uses a unipolar +15V pulser: lower voltage and single polarity, which reduces acoustic energy per pulse and may contribute to bandwidth limitation.

---

## Part 2: What WULPUS Validates That pic0rick Should Adopt

WULPUS was designed for a completely different use case (wearable, ambulatory monitoring vs benchtop NDT). But several WULPUS design decisions address real gaps in pic0rick that are independent of the power/wireless constraints.

### Recommendation 1: Structured multi-channel TX/RX configuration table

**What WULPUS does:** A 16-slot TX/RX configuration table allows up to 16 independent channel-pair configurations per acquisition cycle. Each slot specifies which TX channels fire and which RX channels receive, with a separate switch pattern per slot. The GUI renders a real-time 8-channel synthetic aperture B-mode from sequential slots without custom code.

**pic0rick current state:** The MAX14866 PMOD board supports 8-channel switching, but channel control is a raw serial command (`write mux <value>`). There is no structured multi-slot config system. Writing a multi-element acquisition sequence requires manual loop scripting in the host.

**Recommendation:** Add a `MultiChannelConfig` dataclass to pic0lib — analogous to `WulpusUssConfig` — that encodes a sequence of (TX channel mask, RX channel mask, gain, pulse params) slots and executes them in sequence. This would make 8-element synthetic aperture imaging a one-line configuration rather than a custom acquisition loop. The MAX14866 PMOD already provides the hardware; the software abstraction is missing.

---

### Recommendation 2: IMU-gated acquisition for motion rejection

**What WULPUS does:** An IIS2DH 3-axis accelerometer is mounted on the acquisition PCB. The nRF52832 firmware reads the IMU between acquisition frames. Acquisitions corrupted by subject movement can be flagged or suppressed before transmission.

**pic0rick current state:** No motion sensing. In NDT applications where the probe is manually positioned against a surface, probe movement during acquisition is a common source of corrupted frames. There is no built-in mechanism to detect or reject these frames.

**Recommendation:** Add IMU support as a PMOD peripheral (the IIS2DH or equivalent SPI/I2C accelerometer is a small, cheap IC). Add a `motion_gate` parameter to `pic0lib`: if the IMU detects acceleration above a threshold during the acquisition window, the frame is flagged in the HDF5 metadata. For NDT thickness gauging specifically, this would allow automated rejection of frames acquired while the probe was shifting, improving measurement repeatability without requiring manual frame selection.

---

### Recommendation 3: Real-time streaming visualization

**What WULPUS does:** The `wulpus_gui.ipynb` Jupyter notebook uses `ipywidgets` to display a live-updating A-mode plot (raw RF + bandpass-filtered + Hilbert envelope) and a live B-mode synthetic aperture depth map during acquisition. Visualization updates in real time without stopping the acquisition loop.

**pic0rick current state:** The notebook workflow is poll-based: acquire a frame, plot it, acquire the next frame. There is no live-updating display. For multi-frame averaging, sweep measurements, or finding the optimal probe position, the lack of live feedback is a friction point.

**Recommendation:** Add a `live_view()` mode to pic0lib using `ipywidgets` output widgets and `threading`. The main acquisition loop runs in a background thread; the foreground thread updates the plot widget at ~10 Hz. The implementation is straightforward: WULPUS's GUI provides a working template, and pic0rick's data is already in numpy arrays. A real-time A-mode display with bandpass filter and Hilbert envelope would significantly improve the probe-positioning workflow.

---

### Recommendation 4: Enclosure for field NDT deployment

**What WULPUS does:** Provides mold files for a silicone enclosure that holds the two PCBs as a single waterproof unit. The enclosure can be strapped to a limb. The form factor enables deployment in environments where an open dev board is not appropriate.

**pic0rick current state:** Open dev board form factor. Fine for benchtop use. Not appropriate for field NDT, outdoor environments, or any application where the device might be exposed to dust, water, or mechanical stress.

**Recommendation:** Design a minimal 3D-printable or CNC-machinable enclosure that houses the main ADC board and pulser PMOD as a single handheld unit, with a transducer connector on one end and a USB port on the other. This does not require hardware changes — only a mechanical design. The STEP files already present in the repo provide the mounting geometry. A waterproof variant (IP54 or better) would make pic0rick suitable for outdoor NDT (pipe inspection, structural steel, concrete). This is the lowest-effort improvement relative to its impact on usability.

---

### Recommendation 5: Enable GitHub Discussions

**What WULPUS does:** GitHub Discussions is enabled on the WULPUS repo and serves as the primary community support channel. Users can post questions, share results, and discuss use cases without the repo owner being the sole point of contact.

**pic0rick current state:** GitHub Discussions is disabled. Community support is routed through a Matrix chat room (matrix.to link in the README). Matrix is appropriate for real-time chat, but it is not indexed by search engines, doesn't accumulate a searchable FAQ, and requires Matrix account registration.

**Recommendation:** Enable GitHub Discussions. Questions, calibration results, and transducer compatibility reports accumulate as a searchable resource that benefits future users without requiring the maintainer to respond every time. This is a zero-effort infrastructure change that compounds in value over time.

---

## Summary

| Recommendation | Effort | Impact | Prerequisite hardware? |
|----------------|--------|--------|------------------------|
| Structured TX/RX config table in pic0lib | Medium (Python) | High — unlocks systematic array imaging | No (MAX14866 PMOD already exists) |
| IMU-gated acquisition PMOD | Medium (Python + PMOD board) | Medium — improves NDT repeatability | Yes (SPI/I2C IMU PMOD needed) |
| Real-time ipywidgets live view | Low (Python) | High — improves probe positioning UX | No |
| Field enclosure (3D-printable) | Medium (mechanical design) | Medium — enables outdoor/field NDT | No (mechanical only) |
| Enable GitHub Discussions | Trivial | Medium — reduces single-author bus factor | No |

All five recommendations are feasible without changing the core hardware (ADC, pulser, amplifier, RP2040). They address usability gaps that are real, well-documented, and validated by WULPUS's design choices. None require the power/wireless tradeoffs that define WULPUS's architecture.

---

## Part 3: Software and Firmware Recommendations

*Drawn from the WULPUS codebase, WULPUS published workflows (IEEE IUS 2022–2023, TBioCAS 2024), and WULPUS GitHub Discussions (#20, #34) and Issues (#13, #23).*

### SW-1: Acquisition configuration as a typed dataclass, not CLI strings

**Current pic0rick approach:** Acquisition parameters are passed as positional CLI arguments over serial: `start acq <pon_ns> <poff_ns> <damp_ns>`. Gain is set separately with `write dac <value>`. There is no structured representation of an "acquisition configuration" as a Python object.

**What WULPUS does:** `WulpusUssConfig` is a Python dataclass holding all acquisition parameters — PRF, sample count, pulse frequency, pulse count, duty cycle, gain, timing margins, and up to 16 TX/RX channel slot definitions. It serializes to a 68-byte binary package sent to the probe. A configuration is a first-class object: it can be saved, loaded, versioned, compared, and logged alongside the data it produced.

**Recommendation:** Add an `AcquisitionConfig` dataclass to pic0lib:
```python
@dataclass
class AcquisitionConfig:
    pon_ns: int = 200
    poff_ns: int = 200
    damp_ns: int = 2000
    gain_dac: int = 512        # 10-bit, 0–1023
    transducer_id: str = ""
    note: str = ""
```
`Pic0rick.acquire(config)` sends the parameters, reads the buffer, and returns an `AcquisitionResult` containing both the numpy array and the config that produced it. This config object is then stored in the HDF5 file alongside the data. Benefit: you can always reconstruct exactly how a stored trace was acquired. WULPUS's published papers all rely on this property — reproducible acquisition parameters are required for peer review.

---

### SW-2: Hilbert envelope as a first-class output in pic0lib

**Current pic0rick approach:** The `ndt_acquisition.py` module applies a Butterworth bandpass filter and plots raw + filtered signals. Envelope detection (Hilbert transform) is not part of the standard output — users must implement it separately using `scipy.signal.hilbert`.

**What WULPUS does:** The GUI (`gui.py`) computes and displays three simultaneous traces for every acquired frame: raw RF, bandpass-filtered RF (31-tap Remez FIR via `filtfilt`), and Hilbert envelope. The envelope trace is the primary clinical display — it shows echo amplitude as a function of depth without the RF carrier oscillation, making reflector positions visually unambiguous.

**Recommendation:** Add `pic0lib.processing.envelope(samples, fs, f_low, f_high)` that returns `(filtered, envelope)` — bandpass-filtered signal and its Hilbert magnitude. Wire this into the `AcquisitionResult` object so that `result.envelope` is available immediately after acquisition without writing signal processing code. The implementation is 5 lines of scipy; the value is that it becomes the default output, not something users discover after reading forum posts.

This is validated by the WULPUS discussions: users in bring-up (#20) would have immediately seen from the envelope whether they were receiving echoes at all, rather than trying to interpret raw RF data with saturated early-time artifacts.

---

### SW-3: Configurable bandpass filter with transducer-aware defaults

**Current pic0rick approach:** `ndt_acquisition.py` applies a 4th-order Butterworth bandpass via `sosfiltfilt`. The filter cutoffs are not exposed as API parameters — users must edit the source to change them.

**What WULPUS does:** The GUI provides slider-adjustable bandpass filter cutoffs (0.1–0.9 × Nyquist), implemented as a 31-tap Remez FIR filter recomputed on parameter change. The filter parameters are also stored in the `WulpusUssConfig` serialization.

**Recommendation:** Make bandpass cutoffs first-class parameters in `AcquisitionConfig` (or a companion `FilterConfig`). Provide sensible transducer-frequency-aware defaults — e.g., for a 5 MHz transducer at 65 Msps, default passband 2–8 MHz. Expose a `set_transducer(center_freq_mhz, bandwidth_fraction=0.7)` helper that automatically sets `pon_ns`, `poff_ns`, and filter cutoffs for the specified transducer. WULPUS's papers all specify the filter parameters used; being able to reproduce a result requires that the filter is part of the stored configuration.

---

### SW-4: M-mode display

**What WULPUS does:** Not implemented. A user in Discussion #20 explicitly asked whether WULPUS supports M-mode; the question went unanswered. WULPUS's GUI shows A-mode (single frame, amplitude vs depth) and B-mode (8-channel synthetic aperture spatial image). Neither of these is M-mode.

**What M-mode is:** M-mode (Motion mode) is a time series of a single depth gate across many sequential acquisitions. It displays depth on the Y-axis and time on the X-axis, showing how the position of a reflector evolves over time. It is the standard display for cardiac wall motion, carotid pulsation, and muscle thickness change over time.

**Recommendation for pic0rick:** Add `pic0lib.display.mmode(frames, gate_start_ns, gate_end_ns, fs, prf)` that slices a sequence of A-mode frames at a specified depth gate and renders the time-series as a 2D image. This is directly useful for pic0rick's NDT applications: monitoring back-wall position over time reveals thermal drift, material deformation, and multi-layer delamination. It is also directly applicable to the cardiorespiratory use case demonstrated by WULPUS papers (Papers #3 and #7 in `w_uses.md` both rely on M-mode-style time series of arterial wall position). The implementation is a 2D numpy stack — the complexity is in the display, not the data.

---

### SW-5: Pulse parameter sweep with automated quality scoring

**Current pic0rick approach:** `ndt_acquisition.py` includes a grid search over `(pon, poff)` space (`grid_search_pulse_params`). This is already a strong feature. However, the quality metric used (signal-to-noise ratio of the back-wall echo) requires prior knowledge of where the back-wall echo should appear.

**WULPUS bring-up insight (Discussion #20, #34):** The most common failure mode for new WULPUS users is incorrect HV MUX timing — functionally equivalent to pic0rick's `damp_ns` being too short or too long. In both cases, there is no feedback to the user about which timing parameter is wrong. WULPUS has no automated pulse parameter sweep; users tune by hand via oscilloscope.

**Recommendation:** Extend pic0rick's existing grid search with two additional automated quality metrics:
1. **Echo presence score:** Does any window beyond the initial saturation zone contain a signal peak above N × noise floor? (Binary pass/fail, does not require knowing echo depth.)
2. **Ringing duration:** How many samples after the pulse is the signal above a threshold? Shorter ringing = better damping = better near-surface resolution. This directly measures whether `damp_ns` is tuned correctly.

These two metrics together allow automated bring-up: `pic0lib.sweep.auto_tune(transducer_freq_mhz)` runs a grid search and returns the (pon, poff, damp) triple that minimizes ringing while maximizing echo presence. This would have resolved both Discussion #20 and Discussion #34 in minutes rather than days of back-and-forth.

---

### SW-6: Per-frame metadata and provenance logging

**Current pic0rick approach:** HDF5 storage includes acquisition metadata. However, the metadata schema is not strictly enforced — fields are optional and the format is not version-controlled.

**What WULPUS does:** Each frame carries a 3-byte header (2-byte acquisition number + 1-byte TX/RX config ID) before the 800-byte RF payload. At the host side, every saved `.npz` file is auto-numbered (`data_0.npz`, `data_1.npz`…) but does not embed the configuration that produced it. This is actually a weakness of WULPUS that WULPUS papers work around by saving the config separately.

**Recommendation:** Define a versioned metadata schema for pic0rick HDF5 files:
```
/acquisitions/{key}/
    data         — numpy array (samples × 1)
    config       — JSON-serialized AcquisitionConfig
    timestamp    — ISO 8601
    firmware_ver — string (queried from device at connect time)
    lib_ver      — pic0lib version string
    notes        — free text
```
Add `Pic0rick.firmware_version()` — a serial command that returns the firmware version string. Store it in every HDF5 file. This is the minimum needed to trace any future result back to the exact software and hardware configuration that produced it, which is a requirement for any published experimental result.

WULPUS's peer-reviewed papers all include explicit firmware revision numbers in their methods sections. pic0rick's published demos do not — this is a reproducibility gap that a versioned schema would close.

---

### SW-7: Makefile / non-IDE build for firmware

**Current pic0rick approach:** The firmware uses Pico SDK + CMake, which is already significantly better than WULPUS's three-IDE requirement. However, the build instructions are not prominently documented, and the development environment setup assumes familiarity with CMake and the Pico SDK toolchain.

**WULPUS issue insight (#23):** A core WULPUS contributor (CedricHirschi) opened an issue requesting Makefile-based compilation as an alternative to the IDEs. The issue notes that TI's online CCS (`dev.ti.com/ide`) now works for MSP430 builds without a local install — but this was not in the documentation.

**Recommendation for pic0rick:** Add a `firmware/BUILD.md` documenting the exact CMake commands to build without an IDE:
```bash
mkdir build && cd build
cmake .. -DPICO_SDK_PATH=/path/to/pico-sdk
make -j4
```
This takes 10 minutes to write and removes the IDE requirement entirely. The CMake build already works — it just isn't documented as the primary path. Additionally, document the UF2 drag-and-drop flash method (Pico bootloader) as the default flashing approach, which requires no debugger. WULPUS users spend significant time on JTAG programmer setup; pic0rick avoids this entirely but doesn't highlight it.

---

### Software Recommendations Summary

| Recommendation | Effort | Source of insight | Impact |
|----------------|--------|-------------------|--------|
| SW-1: `AcquisitionConfig` dataclass | Low (Python) | WULPUS `WulpusUssConfig` | High — reproducible configurations |
| SW-2: Hilbert envelope as default output | Very low (Python) | WULPUS GUI, Discussion #20 | High — standard US display |
| SW-3: Configurable bandpass with transducer defaults | Low (Python) | WULPUS GUI, published methods | Medium — filter reproducibility |
| SW-4: M-mode display | Low (Python) | WULPUS gap + Discussion #20 unanswered | Medium — cardiac/NDT time series |
| SW-5: Automated pulse tuning (ringing + echo metrics) | Medium (Python) | Discussions #20, #34 | High — eliminates bring-up failures |
| SW-6: Versioned HDF5 metadata schema | Low (Python) | WULPUS paper methodology | Medium — reproducibility for publication |
| SW-7: Document CMake/UF2 build path | Trivial (docs) | WULPUS Issue #23 | Medium — lowers firmware barrier |

SW-1 through SW-3 are prerequisites for any published result from pic0rick. SW-5 directly addresses the most common failure mode documented in WULPUS's community threads and would have reduced bring-up time from days to minutes for multiple users.
