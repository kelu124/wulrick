# Phase 3: Software and Firmware Ecosystem

---

## 3.1 Firmware Architecture

### pic0rick

**Language and build system:** C firmware using Pico SDK 2.2.0, CMake build. PIO programs in RP2040 assembly. MicroPython is documented as an option but the production firmware is C with a serial REPL interface.

**Core firmware structure:**
- `main.c` — serial REPL command dispatcher (115200 baud over USB)
- `adc/adc.c` + `adc/adc.pio` — PIO-based ADC capture engine
- `max/max14866.c` — MAX14866 mux and AD8331 TGC control

**How PIO handles acquisition:**
The RP2040 PIO runs three state machines in parallel:
- SM0: captures 10-bit parallel ADC data on rising clock edge, feeds DMA ring buffer
- SM2: controls GPIO 11–12 for pulse timing (transmit pulse on/off/damp)
- SM3: controls GPIO 16–17 (synchronized second pulse output)

Configurable pulse parameters passed via serial: `start acq <pon_ns> <poff_ns> <damp_ns>`. All timing is cycle-accurate via PIO without CPU intervention during the capture window. DMA fills an 8000-sample buffer.

**Serial interface commands:**
```
start acq <pon_ns> <poff_ns> <damp_ns>   # trigger acquisition
read                                       # dump buffer as hex CSV
write dac <value>                          # set TGC gain (10-bit)
write mux <value>                          # configure MAX14866
set mux / clear mux                        # GPIO control
```

### WULPUS

**Languages:** C for both MCUs (MSP430FR5043 + nRF52832 + nRF52840 dongle), three separate firmware projects requiring three different toolchains.

**MSP430 firmware** (us acquisition):
- Controls USS_A module: PLL (68–80 MHz), PPG (pulse generator), SDHS ADC (8 Msps, 12-bit), PGA (46 gain steps, -6.5 to +30.8 dB)
- Controls HV mux via SPI (separate TX/RX configs for each acquisition slot)
- Power management: turns on/off DC-DC converter, op-amp supply, all synchronized by timer
- Supports up to 16 TX/RX channel configuration slots per acquisition cycle
- Stores data in FRAM, DMA transfer to nRF52 via SPI
- Configuration received from nRF52 as 68-byte binary package (start byte 0xFA)

**nRF52832 firmware** (BLE bridge):
- Receives ultrasound frames from MSP430 via SPI (4 × 201 bytes = 804 bytes/frame)
- Buffers up to 35 frames in RAM
- Transmits over BLE at 320 kbps to nRF52840 USB dongle
- Reads IIS2DH accelerometer (IMU on acquisition PCB) for motion detection
- Handles configuration relay: host → BLE → nRF52 → SPI → MSP430

**nRF52840 dongle firmware** (USB gateway):
- Receives BLE frames from nRF52832
- Re-serializes to USB at 4 Mbps to host PC

### Comparison: Firmware

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| MCU firmware language | C + PIO assembly | C (3 separate projects) |
| Build system | Pico SDK / CMake | TI CCS (MSP430) + Nordic nRF SDK |
| Firmware projects | 1 | 3 (MSP430 + nRF52832 + nRF52840) |
| Toolchains needed | 1 (Pico SDK) | 3 (TI CCS, nRF SDK, nRF SDK v2) |
| Pulse configuration | Serial CLI params | Binary config package (68 bytes) |
| Acquisition trigger | Serial command | Timer-driven, autonomous |
| Parameter granularity | ns-level (PIO ticks) | µs-level (register-based) |
| Gain steps | Continuous (10-bit DAC) | 46 discrete (PGA register) |

---

## 3.2 Host-Side Software

### pic0rick

**Library: `pic0lib`** (Python, MIT/GPLv3 mixed)

**Device communication (`device.py`):**
- PySerial over USB-ACM (115200 baud)
- Auto-detects serial port across Linux/macOS/Windows
- Key API:
  ```python
  pic = Pic0rick()
  pic.pulse_adc_trigger(pon=200, poff=200, damp=2000)  # ns
  pic.dac(N)        # 10-bit gain value
  pic.read()        # returns numpy array of samples
  ```
- `Fech = 60 MHz` constant in library
- Verbose/logging support

**NDT acquisition framework (`ndt_acquisition.py`):**
- `UltrasonicAcquisition` dataclass: stores signal + metadata (gain, pulse params, transducer ID, timestamp)
- **Signal processing:**
  - 4th-order Butterworth bandpass (zero-phase via sosfiltfilt)
  - Automated back-wall echo detection with periodic echo prediction
  - Grid search over pon/poff space for optimal pulse parameters
- **Storage:** HDF5 (h5py) with deduplication by acquisition key; gzip compression
- **Plotting:** matplotlib with raw + filtered dual-channel display

**Signal processing repo (`us_rf_processing`):**
- B-mode image reconstruction from raw RF data
- Scan conversion (sector/linear → Cartesian)
- Beamforming pipeline (synthetic aperture)
- Data from multiple probe types: HP2121, Kretz AW145BA, ATL3, 5 MHz NDT
- Sampling frequencies tested: 10–64 MHz

**Python dependencies:** numpy, scipy, h5py, matplotlib, pyserial, pandas, altair, jupyter

### WULPUS

**Application: `sw/wulpus/`** (Python, Apache 2.0)

**Serial communication (`dongle.py`):**
- PySerial at 4 Mbps (USB-to-serial via nRF52840 dongle)
- Frame format: `b'START\n'` + 2-byte acq number + 1-byte TX/RX config ID + 800 bytes RF data (400 × int16)
- `send_config()`: transmits 68-byte config package to probe

**Configuration system (`uss_conf.py` + `config_package.py`):**
- `WulpusUssConfig` dataclass — all acquisition parameters in one object:
  - Sampling freq: 8 Msps / {10, 20, 40, 80, 160} = {800k, 400k, 200k, 100k, 50 kHz}
  - Number of samples (default 400)
  - Measurement period (655–65535 µs)
  - PPG: pulse frequency (0–5 MHz), pulse count (0–30), duty cycle
  - PGA gain: 46 steps from -6.5 to +30.8 dB
  - Per-acquisition timing: PPG start, ADC on, PGA bias, sample start, capture restart, timeout, HV mux RX switch (all in µs, converted to hardware ticks internally)
  - Up to 16 TX/RX configuration slots

**Channel mux control (`rx_tx_conf.py`):**
- `WulpusRxTxConfigGen`: maps logical channels to physical switches
- RX channels on even switches [0,2,4,6,8,10,12,14], TX on odd [1,3,5,7,9,11,13,15]
- `add_config(tx_channels, rx_channels, optimized_switching)`: generates 16-bit bitwise switch patterns
- Optimized switching option pre-activates channels before TX to reduce artifacts

**GUI (`gui.py`):**
- Jupyter widget-based (`ipywidgets.VBox`)
- Real-time A-mode: raw RF + bandpass-filtered + Hilbert envelope overlay
- Real-time B-mode: 8-channel synthetic aperture depth map (depth = samples/fs × 1540/2)
- Bandpass filter: 31-tap Remez FIR, slider-adjustable (0.1–0.9 × Nyquist), SciPy filtfilt
- Envelope: Hilbert transform
- Data saved as NumPy `.npz` files (auto-numbered: data_0.npz, data_1.npz…)

**Entry point (`wulpus_gui.ipynb`):** Jupyter notebook, step-by-step workflow

**Python dependencies:** Python 3.9, numpy 1.23.5, scipy 1.9.3, matplotlib 3.5.2, ipywidgets, pyserial, jupyter

### Comparison: Host Software

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Interface paradigm | Python library + Jupyter notebooks | Python application + Jupyter GUI |
| Serial interface | 115200 baud USB-ACM | 4 Mbps via nRF52840 dongle |
| Real-time visualization | No (notebook-based, poll model) | Yes (ipywidgets live update) |
| Real-time filtering | No | Yes (31-tap FIR, Hilbert envelope) |
| Data format | HDF5 (keyed, compressed) | NumPy .npz (sequential) |
| B-mode support | Separate repo (us_rf_processing) | Built into GUI (8-ch synthetic aperture) |
| Gain control API | `pic.dac(N)` (10-bit, continuous) | `rx_gain=X` in config (46 discrete dB steps) |
| Channel control | Serial command | 16-slot TX/RX config table |
| Configuration method | CLI parameters at acquisition time | Typed Python dataclass, compiled to binary packet |
| NDT-specific tools | Yes (echo detection, thickness gauge) | No |
| Beamforming | Separate repo (us_rf_processing) | Basic synthetic aperture in GUI |
| Storage | HDF5 with deduplication | NumPy .npz sequential |

---

## 3.3 Key Software Findings

### pic0rick is optimized for exploratory NDT experiments

The pic0lib API is intentionally low-level: you set pulse timings in nanoseconds, read raw samples, and do your own processing. The NDT module adds higher-level tools (echo detection, gain sweep, HDF5 logging) built on top of this. The `us_rf_processing` repo is a separate research repository with B-mode pipeline examples, not a packaged library. The overall ecosystem is notebook-driven and optimized for one-off experiment scripting, not continuous real-time monitoring.

### WULPUS is built for continuous real-time monitoring

The WULPUS software is a complete acquisition-to-display pipeline: configure → stream → visualize → save. The Jupyter GUI provides live A-mode and B-mode updates during acquisition. The configuration system is fully parameterized, and the 16-slot TX/RX system enables multi-channel synthetic aperture in the GUI without custom code. The data model is stateless (sequential .npz files), optimized for continuous long-recording sessions.

### Customization barrier differs significantly

| Customization task | pic0rick | WULPUS |
|---|---|---|
| Change pulse timing | Change serial command args (no recompile) | Edit usconfig Python dict (no recompile) |
| Change gain | `pic.dac(N)` call (no recompile) | `rx_gain=X` in config (no recompile) |
| Add custom signal processing | Edit Python notebook | Edit Python notebook |
| Change acquisition sequence | Edit PIO assembly + recompile C | Edit C firmware for all 3 MCUs + recompile |
| Add a new hardware peripheral | Add PMOD board + firmware module | Modify hardware + MSP430 C firmware |
| Change pulse waveform shape | Edit PIO program + recompile | Constrained by MSP430 USS PPG hardware |

pic0rick's PIO-based approach makes pulse timing highly configurable from the Python host without reflashing. WULPUS's acquisition sequence is automated by the MSP430 USS_A hardware, which is efficient but less modifiable without touching C firmware.
