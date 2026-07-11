# pic0rick — OtherSystems Feature Port Assessment

**Purpose:** Concrete transplant analysis for each OtherSystems design feature that could realistically be added to pic0rick. Not aspirational — each entry states what changes, what stays, rough BOM delta, and the actual blockers.

**pic0rick baseline:**
- MCU: RP2040 (Raspberry Pi Pico), 264 KB SRAM, 8× PIO state machines
- ADC: ADC10065, 65 Msps, 10-bit (ENOB 9.6)
- TGC amp: AD8331 VGA (48 dB, 7.5–55.5 dB), MCP4812 12-bit DAC
- T/R switch: MD0100N8-G (passive limiter)
- Pulser: MD1213K6-G + TC6320TG-G, ±24V three-level bipolar, on separate PMOD
- HV supply: R2D-0524_P isolated DC-DC (USB 5V → ±24V), 2W rated
- Mux: MAX14866 8-ch PMOD (optional, already exists)
- Connectivity: USB Full Speed (Pico built-in)
- Form: multi-board dev stack, USB-tethered, no battery

---

## 1. Multi-Element Mux for Array Scanning

**Source systems:** WULPUS (HV2707, 8-ch), TinyProbe (32-ch, custom ASICs)

### What WULPUS does

Uses the HV2707 8:1 analog mux (Microchip, SPI-controlled, ±80V rated, SOIC-16) to time-multiplex transmit and receive across 8 transducer elements through a single T/R switch (MD0101) and single receive chain. The mux sits directly at the transducer node, before the T/R switch. This enables sequential A-mode scanning or synthetic aperture (SA) imaging across 8 sites with one ADC.

### What pic0rick already has

The MAX14866 PMOD board is already the direct functional equivalent of the HV2707 — it is an 8-channel analog mux rated for ±100V ultrasound signals, SPI-controlled, and already integrated into the PMOD ecosystem. This means the mux transplant from WULPUS is **already done** for 8 channels.

### Transplant 1A — Integrate mux onboard rather than as PMOD

**What changes:** Move the MAX14866 (or HV2707 — both are pin-compatible architecturally) from the external PMOD board to the main ADC PCB. The 8 transducer SMA/MMCX connectors move to the main board.

**What stays:** Signal chain unchanged. SPI control from RP2040 unchanged. MAX14866 BOM unchanged.

**Benefit:** Eliminates the PMOD connector insertion loss (~0.2 dB) and stray capacitance at the switch output. The MD0100 T/R limiter sits between the mux output and the AD8331 input, same as now.

**BOM delta:** Zero component cost (moves existing parts to main PCB). PCB area increase: ~15 × 12 mm for HV2707/MAX14866 + 8 connector footprints.

**Blocker:** None, if the PMOD interface PCB is redesigned. However, having the mux as PMOD preserves flexibility — an argument for keeping it modular.

---

### Transplant 1B — Scale to 32 channels via cascaded mux boards (TinyProbe approach)

**What TinyProbe does:** 32 elements via FPGA-coordinated switching with custom preamplifier ASICs per element. The FPGA manages simultaneous transmit beamforming delays across 16 sub-apertures.

**What the RP2040/2350 can do instead:** 4× cascaded MAX14866 boards (4 PMOD connectors × 8 channels each), software-sequenced, single-channel at a time. No transmit beamforming (all elements fire simultaneously through a 4:1 switch tree, or each fires sequentially for SA imaging). This gives 32-element sequential A-mode, not simultaneous beamforming.

**SA imaging with 32 channels at 65 Msps:**
- Each element fires once → 8000 samples × 2 bytes = 16 KB
- 32 elements × 16 KB = 512 KB per full SA frame
- At 30 Hz frame rate: 15.4 MB/s — within USB 2.0 HS (60 MB/s) if the Pico runs USB HS

**What changes:** Add 3 more MAX14866 PMOD boards + a 4:1 switch or a secondary mux tree. RP2040 SPI drives all boards via 4 CS lines.

**BOM delta:** 3× MAX14866 PMOD board (~$30–$50 each) + 3× 8-element transducer headers. No core board changes.

**Blockers:**
- USB: standard Pico uses USB Full Speed (12 Mbit/s). 32-element SA frame at 30 Hz = 123 Mbit/s — **exceeds Full Speed.** Requires RP2350 with USB HS or an FTDI FT2232H bridge (same constraint as pic32arick).
- RP2040 SRAM: 264 KB holds only 16 frames of a single 8 KB acquisition — not a full 512 KB SA frame. RP2350's 520 KB improves this marginally; PSRAM PMOD (already in use) resolves it.
- PRF: at 32-element sequential firing, effective frame rate at 50 Hz PRF = 50/32 = **1.56 fps** — exactly the bottleneck identified in `w_discussions.md` Vermon analysis. This is fundamental physics, not a hardware limitation.

---

## 2. RP2350B vs RP2350A

**Source systems:** None use RP2350 — this is a pic0rick-internal upgrade path informed by OtherSystems requirements.

### Why the question matters

The OtherSystems analysis surfaces two demands that strain the RP2040:
- **Multi-element coordination (TinyProbe):** 32 elements need more GPIO, more SPI CS lines, more PIO bandwidth
- **On-chip computation (PuLsE/Anatomy):** envelope detection, FIR filtering, and correlation are currently done on the host PC; moving them on-chip requires M33/FPU and more SRAM

### RP2350A vs RP2350B — the concrete differences

| Feature | RP2040 | RP2350A | RP2350B |
|---------|--------|---------|---------|
| CPU | 2× Cortex-M0+ | 2× M33 + 2× RISC-V | same |
| GPIO | 30 | 30 | **48** |
| Package | QFN-56 | QFN-60 | **QFN-80** |
| SRAM | 264 KB | 520 KB | 520 KB |
| PIO SM | 8 | 12 | 12 |
| HSTX | No | Yes | Yes |
| HW float | No (soft) | Yes (M33 FPU) | Yes |
| USB HS | No (FS only) | No (FS only) | No (FS only) |

### What RP2350B gains for pic0rick specifically

**48 GPIO vs 30:** At 8-channel mux, pic0rick already uses most RP2040 GPIO. Adding more channels (32-element), a second SPI bus (mux CS + DAC CS + ADC CS on separate lines), trigger outputs to external equipment, and synchronisation inputs from a second board exhausts 30 GPIO. RP2350B's 48 pins comfortably accommodate all of these.

Concrete allocation with RP2350B:
- SPI0 (ADC10065, DAC, PSRAM): 3 pins + 3 CS = 6 pins
- SPI1 (mux array, 4× CS for 32-channel): 3 pins + 4 CS = 7 pins
- I2C: 2 pins (IMU, EEPROM)
- PWM/PIO (pulser timing, ADC trigger): 4 pins
- USB: 2 pins (D+/D−)
- GPIO (sync in/out, blanking, debug): 8 pins
- **Total: 29 pins → fits RP2350A; with HSTX (8 pins) and expansion: uses RP2350B headroom**

**520 KB SRAM vs 264 KB:** At 65 Msps × 10 bits = 8.125 bytes/sample (10-bit packed), 1 ms acquisition = 65,000 samples = 81 KB. RP2040 can hold ~3 acquisitions in SRAM; RP2350 holds ~6. More useful: the extra SRAM enables on-chip FFT (4096-point FFT of 16-bit = 16 KB temp buffer) or a ping-pong capture buffer without relying on PSRAM.

**M33 FPU:** The PuLsE paper demonstrates envelope detection entirely on-device at 5.8 mW. On RP2350, a Hilbert transform (already done in Python host via `scipy.signal.hilbert`) could run on-chip, removing a round-trip to the host PC and enabling local peak detection. Benchmark: 4096-point FFT on M33 at 150 MHz ≈ 27 µs — well within the 200 Hz PRF inter-pulse gap.

**HSTX peripheral:** The HSTX (High Speed Transmit) peripheral can drive a parallel interface at up to 300 Mbit/s. This is directly relevant as a FIFO interface to an external USB HS bridge (FT2232H or similar), enabling data rates up to 40 MB/s — resolving the USB bottleneck without changing the RP2350 core clock.

### Transplant recommendation

**Use RP2350B** (not RP2350A) for any pic0rick redesign targeting >8 channels or on-chip computation. The 18 additional GPIO pins are not a nice-to-have — they are required for 32-channel mux control + HSTX + full peripheral independence. The RP2350A's identical SRAM and FPU with fewer pins is a poor compromise for this architecture.

**Swap in a Pico 2 (RP2350A) or a bare RP2350B module:** Pico 2 is pin-compatible with Pico for the 40-pin header — for RP2350A. For RP2350B's extra 18 GPIO, a custom PCB is required (QFN-80 footprint). BOM delta for Pico 2 vs Pico: ~$1 (Pico: $4, Pico 2: $5). For bare RP2350B module: ~$3–5 chip cost + custom PCB.

---

## 3. TX/RX on the Same Board — T/R Switch Topologies

**Source systems:** WULPUS (MD0101 + HV2707), TinyProbe (custom ASIC T/R per element), ModulUS/Anatomy (STHVUP32)

### Current pic0rick topology

The MD0100N8-G is a passive T/R limiter/switch: it presents low impedance during transmit (clamping to ±2.5V to protect the receive chain) and presents the signal transparently during receive. The pulser is on a separate PMOD board; the MD0100 lives on the main ADC board, with the transducer connected at the junction.

This works but has two limitations:
1. **Pulser crosstalk:** The pulser PMOD connects to the transducer through a PMOD connector and PCB trace before the MD0100. At ±24V and 6 ns edge rates, this trace radiates and couples into the receive path.
2. **No active blanking:** The MD0100 passively clamps — it doesn't actively disconnect the receive chain during transmit. The AD8331 sees the clamped ±2.5V and takes ~5 µs to recover from saturation.

### What WULPUS's MD0101 offers differently

The MD0101 is an active T/R switch rather than a passive limiter. It explicitly routes the signal between TX and RX modes under control, rather than relying on clamping. Combined with the HV2707 mux, WULPUS's topology is:

```
Transducer → HV2707 mux (8:1) → MD0101 T/R switch → AD8331-equivalent amplifier
                                       ↑
                              Pulser (unipolar +15V)
```

During transmit: MD0101 connects the pulser to the transducer path; during receive: MD0101 disconnects the pulser and connects the amplifier input.

### Transplant 3A — Add active TX/RX isolation via MD0101 in parallel with MD0100

**What changes:** Add MD0101 between the pulser connection and the transducer node, in addition to (or replacing) MD0100. The MD0101's control pin is driven by PWM_G2 (the blanking gate, already in the design) — active low during transmit, releasing the RX path after the transmit pulse.

**What stays:** MD0100 remains as the secondary protection limiter. AD8331 receive chain unchanged.

**Benefit:** Reduces TX-to-RX isolation time from ~5 µs (saturation recovery) to ~100 ns (switch propagation delay). Shortens the dead zone by 4–5×: from ~4 mm to ~0.75 mm in water.

**BOM delta:** MD0101 ~€1.50 (SOIC-8). One GPIO for control. PCB area ~10 × 5 mm.

**Blocker:** MD0101 must be inserted before the MD0100 in the signal path. At ±24V pulser output, verify MD0101's maximum transmit voltage rating (check Microchip datasheet — MD0101 is rated for high-voltage transmit; confirm ±24V ≤ max).

---

### Transplant 3B — Integrate pulser and T/R switch onto main ADC board

**What TinyProbe and ModulUS both do:** TX and RX circuitry are on the same board, with the T/R switch (or HV driver IC) collocated with the transducer connector. This eliminates the PMOD connector as a source of impedance discontinuity and radiation.

**What changes:** Merge pulser (MD1213 + TC6320), HV supply (R2D-0524_P), and T/R switch (MD0100/0101) onto the main ADC board. The pulser PMOD board is eliminated.

**What stays:** ADC10065, AD8331, RP2040, USB connector, MCP4812 all unchanged.

**Benefit:** At ±24V, every millimetre of trace between the pulser and the transducer is a potential radiator. Collocating TX and RX eliminates this trace. The R2D-0524_P footprint (~22 × 13 mm) fits on a slightly larger board.

**BOM delta:** Zero (parts move from PMOD to main PCB). PCB area increase: ~30 × 20 mm.

**Blocker:** The isolated DC-DC (R2D-0524_P) generates significant EMI at its switching frequency (~100-400 kHz). Placing it on the same board as the receive chain requires careful ground planning and possibly a separate switching section. This is manageable but not trivial — pic0rick's original PMOD separation was arguably done for this reason.

---

## 4. Waveform Flexibility — Arbitrary Waveform Generation

**Source systems:** ModulUS (Anatomy slides, Leitner 2026 / Weik 2026 review — STM32H573 + STHVUP32), pic32arick (dual PWM sequencing)

### What ModulUS does

Uses an STM32H573 ARM Cortex-M33 with integrated high-speed DAC to generate arbitrary waveform shapes, fed into the STHVUP32 (STMicroelectronics, 32-channel HV op-amp driver). The STHVUP32 can reproduce any waveform at high voltage — chirp pulses, coded excitation (Golay codes, Barker codes), narrowband sinusoids, multi-frequency bursts.

pic0rick's current pulser produces a **fixed-shape three-level rectangular pulse** — amplitude (±24V) and width are configurable from Python, but the shape is always a rectangular burst. The bandwidth and center frequency are determined by the pulse width; there is no ability to shape the spectrum.

### Why this matters for pic0rick's frequency range

At 30 MHz, a rectangular pulse with 16 ns pulse width has a −6 dB bandwidth of ~62 MHz — much wider than most transducers (~50–70% fractional bandwidth). The excess bandwidth contributes noise. A windowed sinusoidal burst at the transducer center frequency would improve SNR by confining the transmit energy to the transducer's passband.

Additionally, **coded excitation** (e.g. Golay codes) improves SNR by 3–10 dB through pulse compression — transmit a long coded burst, correlate the received echo against the code. This is used in TinyProbe and is standard in research systems.

### Transplant 4A — DDS-based narrowband excitation PMOD

**What changes:** Add an AD9837 DDS IC (Analog Devices, MSOP-10, ~€5, up to 16 MHz sinusoidal output) as a new PMOD. RP2040 SPI controls the DDS frequency, phase, and output. The DDS output feeds the MD1213 pulser driver instead of (or gated with) the RP2040 PWM.

A frequency-specific sinusoidal burst at 5 MHz or 10 MHz would concentrate transmit energy at the transducer's resonance, improving SNR by 2–5 dB vs a rectangular pulse of the same amplitude.

**What stays:** MD1213 + TC6320 ±24V driver (the DDS output, after level shifting, drives the MD1213 input the same way as the PIO PWM does now). AD8331 + ADC10065 receive chain unchanged.

**BOM delta:** AD9837 ~€5 + MSOP-10 PCB ~$10–15 PMOD board. One SPI CS. One analog MUX (TS5A3159, ~€0.50) to select between PIO PWM and DDS output.

**Blocker:** AD9837 maximum output is 16 MHz. For transducers above 16 MHz (pic0rick's key advantage: 30 MHz capability), the DDS is useless. For 5–10 MHz transducers it is useful; for 10–30 MHz it is not. A faster DDS (AD9854 or AD9910) reaches 100 MHz+ but costs ~€20–50 and requires more complex firmware.

**Blocker 2:** The MD1213 is a digital gate driver — it expects a logic-level (0/3.3V) input. A sinusoidal output from the DDS is an analog signal. A comparator (e.g. LMV7219) or an additional analog MUX path is needed to convert the DDS sinusoid to a logic level for the MD1213. Alternatively, feed the DDS output into a dedicated HV op-amp (instead of the MD1213+TC6320 path) — but then the ±24V swing requires a different driver IC.

---

### Transplant 4B — Coded excitation via PIO pattern generation (no new hardware)

**What changes:** Firmware only. The RP2040 PIO already generates the three-level pulse pattern. Extending this to coded excitation (Barker-13, Golay complementary pairs) requires pre-computing the code and outputting it as a PIO state machine program. Each bit of the code = one pulse period; the PIO fires 13 or 128 pulses in the coded sequence.

**What stays:** All hardware unchanged.

**Benefit:** Golay pairs (128 bits each, sent as two sequential acquisitions) provide +21 dB SNR improvement through pulse compression — the single largest SNR gain available with zero BOM cost. At 5 MHz and 65 Msps, a 128-bit Golay code takes 128 × (1/5 MHz) = 25.6 µs transmit time. Within the existing PRF period.

**BOM delta:** €0.

**Blocker:** The RP2040 PIO program must be rewritten to output arbitrary bit sequences rather than a fixed pulse-width pattern. The `pic0lib` Python API (`dac()`, `pulse_adc_trigger()`) must be extended with a `coded_transmit(code_bits)` function. Medium firmware effort; no hardware changes.

---

## 5. Battery / Portable Power

**Source systems:** WULPUS (350 mAh LiPo, 22 mW, 2.5+ days), TinyProbe (500 mAh Li-Po, <1 W), PuLsE (5.8 mW wrist-worn)

### Why it's currently absent from pic0rick

pic0rick is USB-tethered by design: the R2D-0524_P isolated DC-DC requires a stable 5V input, the ADC10065 consumes ~68 mW alone, and the AD8331 adds ~400 mW at full gain. Total ~300–400 mW. At 22 mW, WULPUS runs for 2.5 days on 350 mAh; at 400 mW, pic0rick would need ~1850 mAh for the same duration — a large, heavy cell.

### What's actually portable within current power budget

The Weik 2026 review categorises applications by TRL. Blood pressure monitoring (TRL 7) and respiratory monitoring (TRL 6) require only episodic capture (one acquisition burst per second or less), not continuous streaming. At 1 Hz PRF, duty cycle is ~100 µs/1000 ms = 0.01% — a massive power reduction. Average power at 1% duty cycle ≈ 4–5 mW for the ultrasound front-end (though RP2040 + ADC10065 quiescent current still dominates at ~100 mW).

### Transplant 5A — WULPUS-style LiPo battery pack as power PMOD

**What WULPUS does:** 350 mAh LiPo (single cell, 3.7V) → MCP73831 USB charger → LiPo output to boost converter (+15V HV supply) and 3.3V LDO (MCU/radio).

**Transplant to pic0rick:**
- LiPo (3.7V) → TPS61023 5V boost (SOT-23-6, ~€1.20, 1A output) → feeds the existing R2D-0524_P input (which expects 4.5–9V)
- USB 5V → MCP73831 (SOT-23-5, ~€0.80) → charges LiPo
- Power selection: a P-channel MOSFET or ideal diode (AP2114 or similar) automatically selects USB vs battery
- 3.3V for RP2040: existing LDO on Pico board, fed from 5V boost

**What changes:** New "power PMOD" board with LiPo connector, MCP73831, TPS61023, power selector, and status LED. USB connector repurposed to USB-charging-only mode when battery in use.

**What stays:** R2D-0524_P and ±24V supply chain unchanged. RP2040, ADC, amplifier all unchanged.

**BOM delta:** MCP73831 (~€0.80), TPS61023 (~€1.20), P-ch power selector MOSFET (~€0.30), LiPo cell (~€5–8 for 1000 mAh). PCB for power PMOD: ~$10–15. Total: ~€20–25.

**Run time at 400 mW:** 1000 mAh × 3.7V × 0.9 efficiency / 0.4 W ≈ **8.3 hours** — useful for field deployment without a laptop.

**Run time at duty-cycled 1 Hz PRF:** If average power falls to ~100 mW (RP2040 + ADC quiescent): **37 hours**. Competitive with WULPUS at 22 mW continuous; not as good, but acceptable for episodic monitoring.

**Blocker:** The R2D-0524_P is specified for 4.5–5.5V input. The TPS61023 5V boost output must be regulated within this window under battery discharge (LiPo voltage drops from 4.2V to 3.0V during discharge). Verify TPS61023 regulation accuracy — most switch-mode boost regulators hold output to ±2%, which is within R2D-0524_P's tolerance.

---

## 6. Wireless Data Offload

**Source systems:** WULPUS (nRF52832, BLE 5.0, 320 kbps), TinyProbe (Wi-Fi, 21.6 Mb/s UDP)

### Bandwidth reality for pic0rick data

| Data type | Rate at 50 Hz PRF | Protocol suitable |
|-----------|------------------|-------------------|
| Raw RF (65 Msps × 10 bit, 8000 samples/pulse) | 26 Mbit/s | Wi-Fi only |
| 4× decimated RF (16 Msps eff.) | 6.4 Mbit/s | Wi-Fi marginal |
| Envelope only (1 Msps eff.) | 0.8 Mbit/s | Wi-Fi comfortable |
| Peak positions + amplitude (depth map) | ~5 kbit/s | BLE comfortable |
| Heart rate (1 number/beat) | ~100 bit/s | BLE trivial |

### Transplant 6A — Raspberry Pi Pico W (WiFi + BLE via CYW43439)

**What changes:** Replace the Raspberry Pi Pico (RP2040) with a **Pico W** (RP2040 + CYW43439 WiFi/BLE, $6 vs $4). Pinout is identical — direct drop-in. Firmware adds Wi-Fi socket server (lwIP or Pico SDK WiFi API).

**What stays:** All hardware (ADC, amplifier, pulser, mux) completely unchanged. Pico W is pin-compatible with Pico.

**What you get:**
- **Wi-Fi (2.4 GHz, 802.11b/g/n):** Theoretical 40 Mb/s; practical ~10–20 Mb/s UDP. Sufficient for decimated envelope streaming at 50 Hz PRF.
- **BLE 5.0:** 320 kbps like WULPUS. Sufficient for depth map or heart rate data.

TinyProbe achieves 21.6 Mb/s UDP for 32-channel compressed frames. For pic0rick single-channel with 4× decimation:
- At 50 Hz PRF, 8000 samples/pulse, 4× decimated = 2000 samples × 2 bytes × 50 Hz = **200 KB/s = 1.6 Mbit/s** — well within Wi-Fi capability.

**BOM delta:** €2 (Pico W vs Pico). This is the single cheapest wireless add among all options.

**Blocker:** Pico W shares the SPI bus between the CYW43439 WiFi/BT chip and external SPI peripherals. On the standard Pico W pinout, GPIO 16–19 are used for CYW43 SPI — these may conflict with the ADC10065 SPI if the existing firmware uses the same GPIO numbers. Verify GPIO assignment and reassign as needed. This is a firmware change, not a hardware change.

**Blocker 2:** USB and WiFi cannot both be used simultaneously on Pico W (USB uses the PLL that conflicts with CYW43 clock on RP2040). For Pico W: data goes over WiFi or USB, not both at once. This is acceptable — field deployment uses WiFi; bench development uses USB.

---

### Transplant 6B — nRF52832 BLE module (direct WULPUS transplant)

**What WULPUS does:** Connects nRF52832 to MSP430 via UART. nRF52832 appears as a BLE peripheral to the host computer; the nRF52840 USB dongle bridges BLE to USB CDC on the host.

**Transplant to pic0rick:** Add a nRF52832 module (e.g. Raytac MDBT42Q, UART interface, ~€8) connected to RP2040 UART0. RP2040 sends compressed A-scan data (depth peaks + amplitudes) via UART at 1 Mbaud; nRF52832 packetizes and transmits over BLE.

**What stays:** All ultrasound hardware unchanged. USB still usable for development/charging.

**BOM delta:** nRF52832 module ~€8. UART 2 pins from RP2040. PCB area ~18 × 10 mm. Host needs nRF52840 USB dongle (~€15) if no BLE built-in.

**Vs Pico W:** Pico W is strictly better for new designs (cheaper, built-in, no dongle needed, faster). The nRF52832 path only makes sense if: BLE long-range mode is required, or if the host software already uses WULPUS's BLE protocol and re-use is a priority.

---

## 7. Mechanical / Form-Factor Adaptations

**Source systems:** EchoLite (8.7 g, 33 mW — form factor champion), WULPUS (46×25 mm, silicone-moldable, forearm strap), TinyProbe (83.2×16 mm aperture, 39.9 g), PuLsE (wrist-worn, watch-style)

### 7A — Probe-tip + cable + dongle form factor (WULPUS-inspired)

**What WULPUS does:** The HV PCB is small enough to mount near the transducer (forearm); the MSP430 and nRF52832 are on the main acquisition PCB ~50 mm away. The two boards are connected by a short flex or wire harness. Both boards are enclosed in a thin silicone mold that provides waterproofing and skin-contact compliance.

**Port to pic0rick:** Separate the pic0rick design into two sub-boards:
- **Probe tip** (new, small): MD0100/MD0101 T/R switch, transducer connector (MMCX or bare pad), optional mux first stage. ~20 × 15 mm. Weight ~3–5 g.
- **Backpack/dongle** (existing hardware): RP2040, ADC10065, AD8331, MCP4812, pulser, HV supply, USB connector. ~60 × 40 mm (estimate). Connects to probe tip via coax cable (50Ω, RG-174 or equivalent, 0.5–1 m length).

**Benefit:** The probe tip can be small, conformal, and lightweight. The operator holds the probe tip against tissue; the backpack clips to a lab bench or pocket. This is the same split as commercial handheld probes (head = small, cable = long, handle/cart = large).

**BOM delta:** New probe-tip PCB (~$5–10 at JLCPCB). Coax cable + connectors (~$3–5). No new ICs.

**Blocker:** At 65 Msps, the signal from the transducer is a high-frequency analog signal. A 1 m cable between the transducer and the AD8331 input will attenuate the signal and add noise. Standard practice: put the **first amplifier stage in the probe tip** (near the transducer) to amplify before the cable loss. This means moving Stage 1 of the amplifier (currently AD8331 on the main board) to the probe tip. Adds one low-noise op-amp (e.g. OPA836) to the tip PCB and requires a power supply line in the cable (3.3V).

---

### 7B — Array aperture layout for 8-element linear scanner (TinyProbe-inspired)

**What TinyProbe has:** 83.2 mm × 16 mm transducer aperture, 32 elements at 2.6 mm pitch, designed to fit over the forearm musculature.

**Port to pic0rick:** Design an 8-element linear array PCB (~55 × 15 mm) with:
- 8 transducer element pads at 7 mm pitch (for 5 MHz elements, λ/2 = 0.15 mm — pitch is wider than λ/2 for B-mode, fine for A-mode multi-site)
- Transducer elements: NDT piezo elements (7 mm × 5 mm, e.g. from Vermon or generic supplier) soldered/glued to matching PCB pads
- SMA or MMCX connectors per element (for connection to MAX14866 mux PMOD)
- OR integrate the MAX14866 directly on this PCB and run a single coax to the main board

This enables: 8-site sequential A-mode (muscle thickness, tissue displacement at 8 locations), or SA B-mode imaging (send on element 1, receive on all 8, repeat per element).

**BOM delta:** Small PCB (~$5–10) + 8 piezo elements (~€5–10 each for NDT-grade, or ~€80 for a Vermon 8-element strip if available). Plus epoxy backing and matching layer materials.

**Blocker:** Transducer fabrication is non-trivial. Off-the-shelf 8-element strips exist (Vermon, Imasonic, per `array_sourcing.md`) but cost €800–2500. Using 8 individual NDT elements (e.g. Olympus contact elements) is cheaper but requires careful acoustic coupling and alignment.

---

### 7C — Silicone enclosure for waterproofing (WULPUS-inspired)

**What WULPUS does:** Uses a custom thin-wall silicone mold around both PCBs, with the transducer aperture exposed. The silicone provides: waterproofing (sweat, cleaning), skin-contact compliance, strain relief for the BLE antenna, and a stable strapping attachment point.

**Port to pic0rick's probe tip:** Cast a simple silicone over-mold around the probe-tip PCB (Transplant 7A). The main board (backpack) does not need enclosing for most applications.

**BOM delta:** Platinum-cure silicone (Dragon Skin 10 or Ecoflex 00-30, ~€30 for 500g kit, makes ~50 molds). 3D-printed mold form (~$1 filament). Total per unit: ~€1–3 silicone cost.

**Blocker:** The transducer element must be acoustically coupled through the silicone layer — silicone's acoustic impedance (~1.0 MRayl) is close to water and tissue (~1.5 MRayl), so it is a good coupling medium. A thin (1–2 mm) silicone layer over the transducer face causes minimal loss. The backing and cable entry point must be sealed — requires attention in the mold design.

---

## Summary Transplant Table

| Transplant | Source | Hardware changes | BOM delta | Complexity | Blocker |
|-----------|--------|-----------------|-----------|-----------|---------|
| 1A: Mux on main board | WULPUS | PCB layout only | €0 | Low | None |
| 1B: 32-ch cascade | TinyProbe | 3× PMOD boards | ~€90–150 | Medium | USB FS bandwidth |
| 2: RP2350B upgrade | (OtherSystems demand) | Pico swap or new PCB | €1–5 | Low–Medium | QFN-80 PCB if bare chip |
| 3A: MD0101 T/R switch | WULPUS | 1 IC + GPIO wire | ~€1.50 | Low | Voltage rating check |
| 3B: Pulser on main board | ModulUS/TinyProbe | PCB redesign | €0 parts | High | EMI management |
| 4A: AD9837 DDS PMOD | ModulUS | New PMOD | ~€15–20 | Medium | <16 MHz limit; level shift needed |
| 4B: Coded excitation firmware | TinyProbe/research | Firmware only | €0 | Medium | PIO reprogramming |
| 5A: LiPo battery PMOD | WULPUS | New PMOD board | ~€20–25 | Low–Medium | Boost regulator spec |
| 6A: Pico W (WiFi+BLE) | WULPUS/TinyProbe | Drop-in module swap | €2 | Low | SPI GPIO conflict; USB/WiFi mutex |
| 6B: nRF52832 BLE module | WULPUS | UART add-on | ~€23 | Medium | Host dongle required |
| 7A: Probe-tip + cable split | WULPUS | New small PCB | ~€8–15 | Medium | First stage in tip |
| 7B: 8-element array PCB | TinyProbe | New array PCB | ~€90–2500 | High | Transducer cost/fab |
| 7C: Silicone enclosure | WULPUS | Mold fabrication | ~€1–3/unit | Low | Acoustic coupling |

**Highest ROI with lowest effort:** Transplants 6A (Pico W, €2, drop-in), 4B (coded excitation, €0, firmware), and 3A (MD0101, €1.50, one IC) deliver the most capability per unit of work. The RP2350B upgrade (transplant 2) is the correct foundation for any design that pursues 1B, 3B, or 7B — those other transplants become significantly more tractable with 48 GPIO and M33 FPU.

---

## 8. Original Ideas — Not from OtherSystems

These are additions not derived from any of the five surveyed platforms. They address standalone operation, field usability, and instrumentation coupling — gaps that the surveyed systems do not directly inform.

---

### 8A — SD Card Storage via PIO-SDIO

**Motivation:** Every OtherSystems platform either streams over BLE/WiFi or requires a host PC. A standalone data logger — where the device captures and stores for hours without a laptop or phone in the loop — is not represented in any of the surveyed designs. This is the single largest gap for field deployment.

**Implementation:** The RP2040 has no native SD/SDIO peripheral, but the PIO state machines can implement 4-bit SDIO at up to ~25 Mbit/s. CarlK's `RP2040-SDIO` library provides a working PIO-based SDIO driver with FatFs integration. For RP2350, the additional PIO state machines further simplify the implementation.

**Data rate math:**
- Raw RF: 8000 samples × 2 bytes × 50 Hz PRF = 800 KB/s continuous — within a Class 10 SD card's write speed (~10 MB/s)
- At 1 Hz duty-cycled logging: 16 KB/s — trivial
- A 32 GB card at 800 KB/s: **11 hours** of continuous raw RF. At 1 Hz: essentially unlimited

**What changes:** Add a push-push microSD socket (e.g. Amphenol 101-00303-68 or Molex 503398-1892, ~€0.50) to the main or battery PMOD board. Connect via 6 GPIO (CLK, CMD, D0–D3) to a free PIO block. Add FatFs + RP2040-SDIO library to firmware. Optionally add a status LED (SD write active).

**Data format:** Binary: 4-byte acquisition header (timestamp lower 32 bits, gain setting, PRF index) + raw int16 samples. Readable post-hoc with a Python script and NumPy.

**What stays:** All ultrasound hardware unchanged. USB interface still available for live streaming when connected.

**BOM delta:** microSD socket ~€0.50. SD card not included. 6 GPIO from RP2040 (available if RP2350B used).

**Blocker:** On RP2040, 6 PIO GPIO for SDIO competes for pins with ADC SPI, mux SPI, and pulser GPIO — tight but possible if GPIO are carefully assigned. RP2350B (48 GPIO) removes this constraint entirely. Sustained write speed on cheap SD cards can drop during sector-erase cycles — implement a double-buffer in SRAM to absorb write stalls.

---

### 8B — Standalone A-Scan Display (SPI TFT)

**Motivation:** Without a laptop, there is currently no way to verify signal quality or probe contact in the field. A small on-device display allows one-person field operation: probe in one hand, visual feedback on-board.

**Implementation:** A 240×240 SPI TFT (ST7789 controller, ~€3, 23 × 23 mm active area) displays a live A-scan in oscilloscope style (time on X, amplitude on Y). The RP2040 DMA-drives the frame buffer update between acquisitions. A frame at 240×240×2 bytes = 115 KB; at 50 Hz, the display update rate would require 5.75 MB/s SPI — feasible at RP2040's SPI speed (up to 62.5 Mbit/s). In practice, a 10–20 Hz display refresh is sufficient.

**Display content:** Scrolling A-scan waveform (raw RF or envelope), depth scale in mm, current gain setting (dB), PRF, and optionally a peak-position cursor for distance measurement.

**What changes:** Add ST7789 module connector (4 SPI + RST + DC lines, 6 GPIO total) to the main board or as a PMOD. Firmware adds a display driver and a simple UI state machine (one button to cycle modes).

**What stays:** All ultrasound hardware unchanged.

**BOM delta:** ST7789 TFT module ~€3–5. Optional tactile button ~€0.10. 6 GPIO.

**Blocker:** The RP2040's 264 KB SRAM must hold the display frame buffer (115 KB for 240×240×2) plus the ADC acquisition buffer (16 KB) — total ~131 KB, leaving ~133 KB for firmware stack and FatFs. Tight but workable. RP2350's 520 KB removes this constraint. Alternatively, the ST7789 can operate in a partial-update mode (update only the waveform strip, not the full screen) to reduce buffer requirements.

---

### 8C — External Trigger I/O (BNC)

**Motivation:** Many real-world ultrasound workflows require synchronisation with external equipment: a mechanical scanning stage (position-triggered acquisition for B-mode reconstruction), an ECG monitor (cardiac-gated acquisition), or a second imaging modality. None of the surveyed systems provide a standard BNC trigger interface.

**Implementation:** Two BNC connectors on the board edge:
- **Trigger IN:** RP2040 GPIO (3.3V-tolerant) through a 1 kΩ series resistor and a Schottky clamp (BAT54) to protect against 5V TTL signals from external instruments. A comparator (LM393, ~€0.30) optionally converts a 50Ω-terminated TTL signal to clean 3.3V logic.
- **Trigger OUT:** RP2040 GPIO → 74LVC1G07 open-drain buffer → 50Ω termination → BNC. Outputs a 0–3.3V TTL pulse coincident with each ultrasound transmit event.

**Use cases:**
- Trigger IN from a stepper motor driver: one pulse per motor step → one A-scan per step → compound B-mode image assembled in post
- Trigger IN from ECG R-wave detector: cardiac-gated acquisition for cardiac wall motion
- Trigger OUT to oscilloscope: scope triggers on transmit pulse, shows received echo on second channel (standard NDT debug workflow)

**What changes:** 2× BNC connector footprints (~€0.80 each), 1× LM393 (~€0.30), passive components. 2 GPIO.

**BOM delta:** ~€2.

**Blocker:** None significant. BNC footprints are large (~12 mm diameter) — requires deliberate board layout to accommodate them at the edge. Alternatively, use SMA connectors (smaller, same electrical function).

---

### 8D — Adjustable HV Supply (Programmable Pulser Voltage)

**Motivation:** The R2D-0524_P is a fixed-ratio isolated DC-DC: USB 5V in → ±24V out, no adjustment. Different transducers and different applications need different voltages:
- A 30 MHz high-frequency transducer (thin element, low impedance) may be over-driven at ±24V, causing ringing and dead-zone extension
- A deep-imaging low-frequency transducer (5 MHz, large element) benefits from maximum drive
- Duty-cycling for battery: lower voltage during low-PRF scanning saves power

**Implementation:** Replace the R2D-0524_P with a programmable HV stage:
- A push-pull transformer-based converter with an adjustable feedback divider
- The feedback resistor divider includes a digital potentiometer (MCP4141, SPI, ~€1.50) whose wiper position sets the output voltage setpoint from RP2040 SPI
- Output range: ±10V to ±30V in 256 steps (~78 mV/step)

Alternatively, and simpler: keep the R2D-0524_P as the primary HV source (±24V fixed) and add a series voltage-controlled attenuator (a linear HV regulator using a BCP56 transistor + RP2040 DAC → op-amp error amplifier) to trim the final output. This is only efficient at small trim amounts (±24V trimmed to ±15–24V range) but covers most use cases.

**What changes:** HV supply section redesign. MCP4141 SPI digital potentiometer added. Firmware extends the `dac()` calibration API to include HV setpoint.

**What stays:** MD1213 + TC6320 pulser driver, MD0100 T/R switch, AD8331 receive chain — all unchanged. The HV change affects only the supply rail.

**BOM delta:** MCP4141 ~€1.50 + new HV section PCB. If using the trim approach: ~€2–4 in additional components.

**Blocker:** Linear HV regulation dissipates power as heat: at 200 mA peak pulse current and 9V trim (24V→15V), peak dissipation = 200 mA × 9V = 1.8W — acceptable only in pulsed operation (short duty cycle). Avoid at high PRF. The full switching regulator approach is more efficient but significantly more complex to design and characterise.

---

### 8E — USB-C PD Sink for Field Charging

**Motivation:** When combined with the battery PMOD (Transplant 5A), the existing USB Micro-B connector limits charging current to 500 mA (USB 2.0 spec). For a 1000 mAh cell, this means ~3 hours to full charge. A USB-C Power Delivery sink negotiates higher power (9V/1.5A = 13.5W charging) from any PD-capable charger or power bank, reducing charge time to ~45 minutes.

**Implementation:** The CH224K (JLCPCB stock, ~€0.30, SOIC-8 or SOT-23-6) is a USB-C PD sink controller. It negotiates 5V/9V/12V/20V from PD sources with no firmware — hardware-only configuration via resistors. For pic0rick, configure it to negotiate 9V @ 1.5A:
- CH224K 9V output → MCP73831 input pin (MCP73831 accepts 4.5–6V, so a buck converter is needed to step 9V → 5V before the charger IC)
- OR: use a USB-C PD charger IC that accepts higher voltages directly (e.g. BQ25895, ~€2, accepts 3.9–14V, I²C-programmable)

**What changes:** USB-C connector replaces Micro-B. CH224K or BQ25895 added to battery PMOD. The RP2040's USB D+/D− still connect as USB 2.0 FS (USB-C is backwards compatible).

**BOM delta:** USB-C connector ~€0.50 (vs Micro-B ~€0.30 delta), CH224K ~€0.30 (or BQ25895 ~€2). Total: ~€1–2 incremental over Transplant 5A.

**Blocker:** The BQ25895 (full-featured PD charger) is an I²C device — adds firmware complexity but enables programmatic control of charge current. The CH224K is fully hardware-configured (simpler) but requires the additional 9V→5V buck converter step. Either approach works; the CH224K path is easier to prototype, the BQ25895 path is cleaner for a finished design.

---

### 8F — On-Device Spectral Analysis (RP2350 M33 FPU)

**Motivation:** The current host-side Python workflow calls `scipy.fft` to compute the frequency spectrum of received echoes — useful for transducer characterisation, centre frequency measurement, and bandwidth verification. With RP2350's M33 FPU, this can move on-device: after each acquisition burst, the RP2350 computes a 4096-point FFT and returns a 2048-bin spectrum instead of (or alongside) the raw time-domain data.

**Use cases:**
- Transducer center frequency measurement (peak of |FFT|) — useful for probe QC without a VNA
- Bandwidth measurement (−6 dB points) to set matched-filter coefficients for coded excitation (Transplant 4B)
- Attenuation-vs-depth spectral tracking: ratio of FFT at depth d vs surface echo reveals tissue attenuation coefficient α(f) — a diagnostic parameter

**Implementation:** Pure firmware addition to an RP2350-based pic0rick. CMSIS-DSP library (ARM, open source, already ported to RP2350) provides `arm_rfft_fast_f32()` — a 4096-point real FFT:
- Benchmark on M33 at 150 MHz: ~650 µs for 4096-point FFT (from CMSIS-DSP characterisation data)
- At 200 Hz PRF, inter-pulse gap = 5 ms — 650 µs leaves ample time
- Memory: 4096 × 4 bytes float = 16 KB for the input buffer + 8 KB for the output complex spectrum — within RP2350's 520 KB SRAM

**What changes:** Firmware only. Add CMSIS-DSP as a CMake dependency. Add `fft_mode` acquisition flag to the API: when set, return 2048-bin float32 spectrum instead of raw int16 samples.

**What stays:** All hardware unchanged. Requires RP2350 (M33 FPU) — on RP2040, software floating-point makes the FFT too slow (~8 ms for 4096-point) to run within the PRF interval at >120 Hz.

**BOM delta:** €0.

**Blocker:** The CMSIS-DSP library adds ~20 KB to the firmware binary — not a concern on RP2350 with 2 MB flash. The main limitation is that the FFT result has no phase reference (the RP2040/RP2350 does not timestamp acquisitions at sub-microsecond precision) — this prevents coherent Doppler estimation but does not affect spectral power analysis.

---

## Updated Summary Transplant Table

| Transplant | Source | Hardware changes | BOM delta | Complexity | Blocker |
|-----------|--------|-----------------|-----------|-----------|---------|
| 1A: Mux on main board | WULPUS | PCB layout only | €0 | Low | None |
| 1B: 32-ch cascade | TinyProbe | 3× PMOD boards | ~€90–150 | Medium | USB FS bandwidth |
| 2: RP2350B upgrade | (OtherSystems demand) | Pico swap or new PCB | €1–5 | Low–Medium | QFN-80 PCB if bare chip |
| 3A: MD0101 T/R switch | WULPUS | 1 IC + GPIO wire | ~€1.50 | Low | Voltage rating check |
| 3B: Pulser on main board | ModulUS/TinyProbe | PCB redesign | €0 parts | High | EMI management |
| 4A: AD9837 DDS PMOD | ModulUS | New PMOD | ~€15–20 | Medium | <16 MHz limit; level shift needed |
| 4B: Coded excitation firmware | TinyProbe/research | Firmware only | €0 | Medium | PIO reprogramming |
| 5A: LiPo battery PMOD | WULPUS | New PMOD board | ~€20–25 | Low–Medium | Boost regulator spec |
| 6A: Pico W (WiFi+BLE) | WULPUS/TinyProbe | Drop-in module swap | €2 | Low | SPI GPIO conflict; USB/WiFi mutex |
| 6B: nRF52832 BLE module | WULPUS | UART add-on | ~€23 | Medium | Host dongle required |
| 7A: Probe-tip + cable split | WULPUS | New small PCB | ~€8–15 | Medium | First stage in tip |
| 7B: 8-element array PCB | TinyProbe | New array PCB | ~€90–2500 | High | Transducer cost/fab |
| 7C: Silicone enclosure | WULPUS | Mold fabrication | ~€1–3/unit | Low | Acoustic coupling |
| **8A: SD card (PIO-SDIO)** | Original | microSD socket | ~€0.50 | Low–Medium | GPIO pin budget (RP2350B preferred) |
| **8B: SPI TFT display** | Original | TFT + 1 button | ~€3–5 | Medium | SRAM (fine on RP2350) |
| **8C: BNC trigger I/O** | Original | 2× BNC + buffer | ~€2 | Low | Board edge area |
| **8D: Adjustable HV supply** | Original | HV section redesign | ~€2–10 | Medium–High | Thermal dissipation |
| **8E: USB-C PD sink** | Original | USB-C + CH224K/BQ25895 | ~€1–2 | Low | Buck step-down for charger |
| **8F: On-device spectral FFT** | Original | Firmware only (RP2350) | €0 | Low | Requires RP2350; M33 FPU |

**Standalone capability stack:** Combining 5A (battery) + 8A (SD card) + 8B (display) + 8C (BNC trigger) gives a fully standalone field instrument: battery-powered, self-logging, with on-board waveform display and external trigger synchronisation. BOM delta for the full stack above the baseline: ~€30–40. This is the direction none of the five surveyed systems have taken — they all require a host device for usable data.
