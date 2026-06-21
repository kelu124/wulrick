# Phase 1: Technical Design Analysis — Findings

*All analysis from schematics, BOMs, datasheets, firmware source code, and published papers. No hardware access.*

---

## System Overview

| | pic0rick | WULPUS |
|---|---|---|
| **Boards** | 3 (ADC main + Pulser PMOD + HV supply) | 2 (Acquisition PCB + HV PCB) |
| **Core MCU** | RP2040 (Raspberry Pi Pico) | MSP430FR5043 + nRF52832 |
| **PCB tool** | KiCad (open source) | Altium Designer (proprietary) |
| **Form factor** | Development board (untethered PMOD stack) | 46×25 mm, 9 g wearable |

---

## 1.1 Pulser — Transmit Excitation

### pic0rick

**Components:** MD1213K6-G (dual MOSFET driver) + TC6320TG-G (200V dual N/P MOSFET) + R2D-0524_P (isolated DC-DC, 5V→±24V) + SN74LVC2G04DBVT (logic inverters)

**Topology:** Three-level bipolar pulser. The R2D-0524_P isolated DC-DC generates ±24V from the USB 5V rail (2W rated, 1kV isolation, ~85% efficiency). The MD1213K6-G drives the gates of TC6320TG-G with 2A peak current and 6 ns rise/fall times. The TC6320TG-G switches between +24V, 0V, and −24V to produce a three-level excitation waveform. BAV99 diodes and logic inverters from SN74LVC2G04DBVT handle level shifting and protection.

**Key specs:**
- Peak voltage: ±24V (48 Vpp)
- Driver peak current: 2A (MD1213K6-G)
- Switch breakdown: ±200V (TC6320TG-G)
- Switch Ron: 7Ω (N-ch) / 8Ω (P-ch) (TC6320TG-G)
- Driver edge speed: 6 ns (with 1000 pF load)
- HV supply: isolated, 2W, from USB 5V

**Design rationale:** Three-level bipolar pulsing is the standard approach for broadband ultrasound transducer excitation. The ±24V swing produces a bipolar waveform that drives the transducer both positively and negatively, which maximizes acoustic output and bandwidth (transducers respond to both half-cycles). The isolated DC-DC allows deriving HV from USB without a transformer on the main board.

### WULPUS

**Components:** Integrated MSP430FR5043 USS_A PHY driver + HV MOSFET driver on HV PCB + HV supply generating +15V

**Topology:** Unipolar positive-only pulser. The MSP430FR5043's USS_A module includes an integrated Programmable Pulse Generator (PPG) and a 4Ω PHY output driver. The HV PCB boosts this to +15V via an on-board HV supply. The excitation is unipolar — the transducer is driven from 0V to +15V rather than ±V.

**Key specs:**
- Peak voltage: +15V (unipolar)
- MSP430 PHY driver output impedance: 4Ω (integrated)
- PPG: programmable frequency and number of pulses
- HV supply: integrated on HV PCB (topology not identified from available sources)

**Design rationale:** Unipolar pulsing simplifies the HV supply (only positive rail needed) and reduces component count. At the shallow depths targeted by WULPUS (subcutaneous tissue for carotid/muscle monitoring), +15V is sufficient. The integrated PPG in the MSP430 eliminates the discrete pulser driver components needed in pic0rick, directly enabling the ultra-low-power, miniaturized design.

### Comparison

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Topology | Three-level bipolar | Unipolar |
| Peak voltage | ±24V (48 Vpp) | +15V |
| Excitation driver | Discrete (MD1213 + TC6320) | Integrated (MSP430 USS PHY) |
| HV source | External isolated DC-DC (USB) | Integrated HV PCB |
| Component count (pulser) | ~12 discrete parts | ~2-3 (mostly integrated) |

**Key difference:** Three-level bipolar pulsing (pic0rick) excites the transducer symmetrically and maximizes acoustic output for a given transducer, which is beneficial for NDT and deeper imaging. Unipolar pulsing (WULPUS) is simpler and lower power, adequate for shallow-tissue wearable sensing but produces less acoustic power and asymmetric transducer loading.

---

## 1.2 TX/RX Switch and Multiplexing

### pic0rick

**Component:** MD0100N8-G (passive high-voltage T/R protection switch)

**What it does:** The MD0100N8-G is a single-channel passive T/R switch. It requires no power supply; it self-limits at ±1V when HV pulses are present, protecting the sensitive AD8331 amplifier input. During receive, it presents low impedance (~15Ω Ron) to pass the echo signal through.

**Key specs:**
- ±100V max differential input
- Trip voltage: ±1V (typ)
- Switch Ron: 15Ω
- Switch capacitance: 21pF
- Package: SOT-89

**Channel count:** 1. No multiplexing in the base system (a separate MAX14866 8-channel mux PMOD is available as an optional expansion).

### WULPUS

**Components:** SPI-controlled HV multiplexer on HV PCB with separate TX and RX configuration registers

**What it does:** The HV mux is controlled via SPI from the MSP430. The firmware (us_hv_mux.c) writes 16-bit shift registers to configure which of the 8 channels are connected for TX and which for RX — separately. A latch enable (LE) signal transfers the shift register contents to the actual switch outputs. TX configuration (hvMuxConfTx) and RX configuration (hvMuxConfRx) can be set independently, allowing different channels for transmit and receive.

**Key specs:**
- Channels: 8 (time-multiplexed)
- SPI control: 8 MHz SPI clock, MSB first
- Control: separate 16-bit TX and RX configs
- The specific mux IC part number could not be confirmed from available sources (not in public web documentation; requires Altium BOM)

**Design rationale:** 8-channel time-multiplexed operation allows WULPUS to acquire signals from multiple transducer elements or multiple body sites sequentially within one acquisition cycle. This is essential for the published applications (multi-site carotid + respiration monitoring, array probes for gesture recognition).

### Comparison

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Protection switch | MD0100N8-G (passive, ±100V) | Integrated in HV mux circuit |
| Channel count | 1 (8 with optional MUX PMOD) | 8 (standard) |
| Mux control | Optional, via MAX14866 PMOD | SPI shift register, 8 MHz |
| TX/RX config | N/A (single channel) | Independent 16-bit registers |

---

## 1.3 TGC / Amplifier

### pic0rick

**Components:** AD8331ARQZ (VGA/TGC) + MCP4812-E/MS (12-bit dual SPI DAC)

**The AD8331:**
- Variable gain amplifier designed for ultrasound receive chains
- Two gain modes: LO (−4.5 to +43.5 dB) and HI (+7.5 to +55.5 dB)
- Total controllable range: 48 dB (HI mode: 7.5–55.5 dB)
- Gain control: analog voltage, 40 mV–1V → 50 dB/V slope
- Noise figure: 3.7 dB
- Bandwidth: 120 MHz (full bandwidth across entire gain range)
- Input: ultralow noise LNA preamplifier

**The MCP4812:**
- 12-bit dual-channel SPI DAC
- Used to generate the gain control voltage for the AD8331
- Enables time-gain compensation (TGC): gain increases with time after transmit pulse, compensating for tissue attenuation at depth
- Fully programmable from the RP2040 over SPI

**Design rationale:** TGC is standard in clinical and research ultrasound. Signals from deeper tissue are weaker (more attenuation). By ramping the gain up over the receive window, you equalize signal amplitude across depths, enabling meaningful comparison of reflectors at different depths. The MCP4812 gives 12-bit resolution over the gain curve — 4096 distinct gain steps across 48 dB.

### WULPUS

**Components:** MSP430FR5043 integrated PGA + discrete op-amp (power-gated)

**The MSP430FR5043 integrated AFE:**
- Programmable gain amplifier (PGA): −6.5 dB to +30.8 dB
- SDHS (Sigma-Delta High Speed) ADC: 12-bit, up to 8 Msps
- The PGA and ADC are part of the USS_A module — tightly integrated

**Discrete op-amp:**
- A discrete external op-amp is present on the acquisition PCB (firmware shows separate power enable/disable functions: enableOpAmpSupply / enableOpAmp)
- This op-amp provides additional fixed gain to bring total gain to 40.8 dB
- It is power-gated by the MSP430 to save energy during idle periods
- Specific op-amp part number not confirmed from available sources

**Total gain: 40.8 dB (fixed)**

**Design rationale:** Fixed gain eliminates the DAC, gain-control resistor network, and separate VGA chip needed in pic0rick. This reduces BOM count and power. At the shallow tissue depths targeted by WULPUS (<5 cm), signal amplitude variation with depth is less extreme, making fixed gain workable. The power-gating of the op-amp contributes to the <25 mW system power budget.

### Comparison

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Amplifier type | Discrete VGA (AD8331) | Integrated PGA (MSP430 USS) + op-amp |
| Gain range | 7.5–55.5 dB (48 dB range) | 40.8 dB (fixed) |
| TGC capability | Yes — programmable gain curve via MCP4812 DAC | No |
| Noise figure | 3.7 dB (AD8331) | Not published |
| Bandwidth | 120 MHz (AD8331) | Limited by MSP430 USS AFE |
| Power gating | No | Yes (op-amp power-gated) |

**Key difference:** TGC (pic0rick) vs fixed gain (WULPUS) is one of the most significant design divergences. TGC is essential for depth-resolved imaging (B-mode, NDT) where you need accurate relative amplitude across depths. Fixed gain is adequate when you're monitoring a specific structure at a known depth (carotid at 2 cm, muscle belly at 1 cm) where depth-dependent variation matters less than power consumption.

---

## 1.4 Analog-to-Digital Conversion

### pic0rick

**Component:** ADC10065CIMT/NOPB (Texas Instruments)

**Key specs:**
- Resolution: 10 bits
- Sampling rate: 65 Msps
- ENOB: 9.6 bits
- SNR: 59.6 dB at 11 MHz input
- Input bandwidth: 400 MHz (−3 dB)
- Power: 68.4 mW (at 65 MHz), 14.1 mW standby
- Supply: single +3.0V
- Interface: parallel CMOS/TTL (28-pin TSSOP)
- Input range: selectable 2 Vpp / 1.5 Vpp / 1 Vpp full scale

**Interface to MCU:** Parallel digital bus to RP2040. The RP2040 PIO state machines capture the parallel ADC output at high speed. Firmware captures data into PSRAM (separate PMOD board) before transferring over USB.

**Axial resolution:** At 65 Msps, the time resolution per sample is 1/65 MHz ≈ 15.4 ns. In soft tissue (c ≈ 1540 m/s), the spatial resolution per sample is c/(2×fs) ≈ 1540/(2×65×10⁶) ≈ 11.8 µm. Practical axial resolution is limited by transducer bandwidth, not ADC rate.

### WULPUS

**Component:** Integrated SDHS ADC within MSP430FR5043 USS_A module

**Key specs:**
- Resolution: 12 bits
- Sampling rate: up to 8 Msps
- ENOB: not published in available sources
- Interface: on-chip (no external digital bus)
- Anti-aliasing: integrated within USS_A module
- Supply: operates from MSP430 core supply

**Axial resolution:** At 8 Msps, time resolution per sample is 125 ns. In soft tissue, spatial resolution per sample: 1540/(2×8×10⁶) ≈ 96 µm. This limits compatible transducer center frequency to approximately 1–3 MHz (Nyquist requires fs > 2×fc, so fc < 4 MHz; practical imaging bandwidth typically fc < fs/4 ≈ 2 MHz).

### Comparison

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Resolution | 10-bit | 12-bit |
| Sampling rate | 65 Msps | 8 Msps |
| ENOB | 9.6 bits | Unknown |
| SNR | 59.6 dB (11 MHz input) | Not published |
| Input bandwidth | 400 MHz | Limited (USS_A AFE BW unknown) |
| Spatial res. (soft tissue) | ~12 µm/sample | ~96 µm/sample |
| Max transducer frequency (approx.) | Up to ~30 MHz | ~2–4 MHz |
| Integration | External chip (parallel bus) | On-chip |
| Power | 68.4 mW (ADC alone) | Included in 22 mW system total |

**Key difference:** The 8× sampling rate difference translates directly to 8× coarser time resolution, limiting WULPUS to low-frequency transducers (<4 MHz) and correspondingly lower axial resolution. pic0rick can work with high-frequency transducers (5–30 MHz) used in NDT and high-resolution B-mode imaging. The 12-bit vs 10-bit comparison is less significant: WULPUS gains 2 bits of quantization but the lower bandwidth is the more constraining spec.

---

## 1.5 Power Supply

### pic0rick

**Architecture:** USB-powered. R2D-0524_P isolated DC-DC generates ±24V for the pulser from USB 5V. On-board linear/switching regulators supply the ADC (+3V) and AD8331.

**Key specs:**
- R2D-0524_P: 5V in → ±24V out, 2W, 1kV isolation, ~85% efficiency, 42mA per output rail
- Total power: not characterized; dominated by ADC (68.4 mW) + AD8331 + RP2040
- Rough estimate: ADC (68 mW) + AD8331 (~150 mW) + RP2040 (~50 mW) + pulser losses ≈ ~300–400 mW minimum during acquisition

**Modular HV:** The HV board is a separate PMOD module, not integrated into the main ADC board. Advantage: the main board can operate without HV for signal chain testing. Disadvantage: requires multi-board stack and more connectors.

### WULPUS

**Architecture:** Battery-powered. The HV PCB integrates the HV generation circuit alongside the mux. The acquisition PCB manages power sequencing.

**Key specs (from published paper and datasheet):**
- Total system power: 22 mW during raw data streaming at 50 acquisitions/second
- MSP430FR5043 active mode: ~120 µA/MHz × 16 MHz ≈ 1.9 mA → ~5.5 mW at 2.9V
- MSP430FR5043 standby: 450 nA (with RTC active)
- nRF52832: 4.6–5.0 mA during BLE TX
- Op-amp: power-gated — active only during receive window
- HV supply: integrated on HV PCB, generates +15V

**Design rationale:** The entire WULPUS system consumes less power than the ADC chip alone in pic0rick. This is achieved by: (a) lower ADC sampling rate (8 vs 65 Msps), (b) integrated rather than discrete amplifier, (c) aggressive power gating of the op-amp and other blocks, (d) duty-cycling the entire system between acquisitions.

### Comparison

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Power source | USB (tethered) | Battery (wireless) |
| System power | ~300–400 mW (estimated) | 22 mW (measured) |
| HV supply | Separate board, modular | Integrated on HV PCB |
| Power gating | None identified | Op-amp, other blocks |
| Battery life | N/A | Multi-day at 50 Hz PRF |

---

## 1.6 Digital Control and Timing

### pic0rick

**Component:** RP2040 (Raspberry Pi Pico) — general-purpose dual-core microcontroller

**USS timing mechanism:** RP2040 Programmable I/O (PIO) state machines. PIO is a novel feature of RP2040: small, dedicated co-processors with their own instruction set, running independently of the main ARM cores. They handle precise cycle-accurate timing for ADC clock generation, data capture, and pulse triggering without CPU overhead.

**Firmware:** MicroPython on the ARM Cortex-M0+ cores for high-level control. PIO assembly for timing-critical operations. The adc.pio file in the firmware controls the ADC data capture sequence.

**Flexibility:** PIO is general-purpose and reprogrammable in software. Pulse width, PRF, ADC timing can all be changed in MicroPython without hardware modification. This makes pic0rick highly configurable for different transducers and experimental setups.

### WULPUS

**Component:** MSP430FR5043 with dedicated USS_A module

**USS_A features (from TI datasheet):**
- Programmable Pulse Generator (PPG): generates configurable ultrasound excitation pulses
- SDHS ADC: 12-bit, 8 Msps, integrated with USS_A
- Integrated 4Ω PHY output driver (drives transducer directly)
- Automated measurement sequencer: start trigger → PPG fires → receive window opens → SDHS captures → DMA transfer to FRAM — all without CPU intervention

**Dual-MCU split:**
- MSP430FR5043: ultrasound acquisition and SPI control of HV mux
- nRF52832: BLE communication and data buffering
- Communication between them: SPI (4 transfers × 201 bytes per frame)
- IIS2DH IMU also present on acquisition PCB for motion artifact detection

**Flexibility:** The USS_A peripheral automates the acquisition sequence. This is efficient but less flexible — the PPG operates within the parameters defined by the MSS hardware. Changing acquisition parameters requires reconfiguring USS_A registers and reflashing C firmware.

### Comparison

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| MCU | RP2040 (general-purpose) | MSP430FR5043 (ultrasound-specific) |
| Timing engine | PIO state machines (reprogrammable) | USS_A dedicated sequencer |
| Pulse generator | Software-defined via PIO | Integrated PPG (hardware) |
| CPU intervention for acquisition | Minimal (PIO handles it) | None (DMA + USS sequencer) |
| BLE | None | nRF52832 (dedicated BLE MCU) |
| IMU | None | IIS2DH (on acquisition PCB) |
| Firmware language | MicroPython + PIO assembly | C (MSP430 + nRF52832) |
| Reconfigurability | High (MicroPython in the field) | Medium (requires reflash) |

---

## 1.7 Connectivity and Data Output

### pic0rick

**Interface:** USB 2.0 (Full Speed/High Speed) via RP2040 native USB.

**Data flow:** RP2040 captures ADC output via PIO → writes to PSRAM (optional PMOD) → transfers over USB to host PC.

**Throughput:** USB 2.0 High Speed is capable of 480 Mbps theoretical. At 65 Msps × 10 bits = 650 Mbps raw ADC rate, USB becomes the bottleneck. In practice, the system likely transfers decimated or buffered frames rather than continuous streaming. Actual throughput depends on firmware implementation.

**Host software:** Python (pyUn0-lib, Jupyter notebooks). Data processed on host PC using numpy/scipy.

### WULPUS

**Interface:** nRF52832 BLE → nRF52840 USB dongle → host PC via USB.

**Data flow:** MSP430FR5043 acquires → SPI to nRF52832 (4 × 201 bytes per frame) → nRF52832 buffers (up to 35 frames in RAM) → BLE transmission (320 kbps) → nRF52840 dongle receives → USB serial to host PC.

**Frame math:**
- 4 transfers × 201 bytes = 804 bytes/frame
- 804 bytes × 8 bits × 50 Hz = 321.6 kbps ≈ 320 kbps (matches published spec)
- This means the BLE link is operating near its throughput ceiling at 50 Hz PRF

**Data compression:** At 8 Msps × 12 bits × 400 samples/acquisition × 8 channels = 38.4 Mbps raw rate. The 320 kbps BLE link carries only ~0.8% of raw data — the MSP430 performs heavy windowing/framing: 400 samples × 2 bytes × ~1 channel per frame × 50 Hz fits into 320 kbps. Full 8-channel data at 50 Hz would require the link to be time-multiplexed across channels.

**Host software:** Python with GUI for acquisition configuration and real-time display.

### Comparison

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Link | USB 2.0 (wired) | BLE 320 kbps (wireless) |
| Tethered | Yes | No |
| Host SW | Python (Jupyter) | Python with GUI |
| Peak throughput | ~60+ Mbps (USB, estimated) | 320 kbps |
| Raw data fraction | High | Very low (~0.8% of ADC output) |
| Latency | Low (USB, <10 ms) | Higher (BLE + buffering) |

---

## 1.8 Block Diagram Summary

### pic0rick Signal Chain
```
[USB 5V] → [R2D-0524_P: ±24V HV] → [MD1213 + TC6320: three-level pulser]
                                               ↓
[Transducer] ←→ [MD0100N8-G: T/R switch]
                           ↓
             [AD8331: TGC amp, 7.5–55.5 dB, MCP4812 DAC-controlled]
                           ↓
             [ADC10065: 10-bit, 65 Msps]
                           ↓
             [RP2040 PIO: parallel capture] → [PSRAM] → [USB → Host PC]
```

### WULPUS Signal Chain
```
[Battery] → [HV PCB: +15V supply] → [MOSFET driver: unipolar pulse]
                                              ↓
[Transducer (1 of 8)] ← [HV MUX: 8-ch SPI-controlled]
                                ↓
                   [MSP430FR5043 USS_A]
                      ├ [Integrated PGA: −6.5 to +30.8 dB]
                      ├ [External op-amp (power-gated): +10 dB]
                      └ [SDHS ADC: 12-bit, 8 Msps]
                                ↓
                   [MSP430 FRAM + DMA]
                                ↓
                   [SPI → nRF52832 → BLE → nRF52840 dongle → USB → Host PC]
```

---

## 1.9 What Each Design Optimizes For

### pic0rick
Optimizes for: **signal quality and experimental flexibility**
- High sampling rate (65 Msps) for compatibility with high-frequency transducers and fine time resolution
- Wide TGC range (48 dB) for imaging at varying depths
- Modular PMOD architecture for adding custom signal processing or mux boards
- MicroPython for rapid experiment iteration without recompilation

Explicitly trades away: wireless operation, battery life, miniaturization, multi-channel integration

### WULPUS
Optimizes for: **power consumption, form factor, and autonomous wearable operation**
- <25 mW total (22 mW measured) enabling multi-day battery life
- 9g, 46×25 mm body-worn form factor
- 8-channel multiplexed acquisition for multi-site monitoring
- BLE wireless link for untethered operation
- Automated acquisition sequencing with minimal CPU overhead

Explicitly trades away: sampling rate (8 vs 65 Msps), TGC, ADC bit depth in practice (ENOB unknown), raw throughput, flexibility for experimental reconfiguration
