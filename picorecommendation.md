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
