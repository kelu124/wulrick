# ultr4rick — Connector Reference

Connector survey for the ultr4rick platform (RP2350B-based). Covers display output (HDMI, VGA) and transducer/RF interfaces. Transducer connector deep-dive is in `phase5.md §5.5`; this file adds display output and collects all connector types in one place.

---

## Display Output Connectors

### Why display output on an ultrasound board?

ultr4rick targets standalone operation: field inspection without a laptop, bedside monitoring without a PC, demo/conference use. A native display output enables:

- Real-time A-scan waveform display (amplitude vs depth, single frame)
- B-mode image output (2D cross-section after synthetic aperture reconstruction on-board or offloaded)
- Gain / TGC curve overlay
- Status and acquisition metadata (PRF, depth, gain, battery)

Both HDMI and VGA are candidates depending on the target display environment. HDMI is preferred for any new design; VGA is retained as a fallback for legacy lab monitors and oscilloscopes with video input.

---

## HDMI (Type A / Type C)

### Physical connector

| Parameter | HDMI Type A (standard) | HDMI Type C (mini) |
|-----------|----------------------|-------------------|
| **Pins** | 19 | 19 |
| **Body size** | 13.9 × 4.45 mm | 10.42 × 2.42 mm |
| **Mating depth** | 5.15 mm | 3.8 mm |
| **Board footprint** | Mid-size | Compact — preferred for ultr4rick |
| **PCB type** | Through-hole or SMT tab | SMT only |
| **Part example** | ASSMANN A-HDMI-A-R-SMT | GCT HDMI-019-02-F-SMT-TR (Type C) |

HDMI Type C (mini) is the right choice for ultr4rick: it saves board space, is lighter, and is available in rugged SMT variants with proper locking tabs. Type A is acceptable if a pre-built HDMI cable ecosystem is more important than footprint.

### Signal lines

HDMI Type A/C carries 19 pins:

| Signal | Pins | Notes |
|--------|------|-------|
| TMDS Data 0, 1, 2 | 6 pins (3 diff pairs) | Video data, TMDS encoded |
| TMDS Clock | 2 pins (1 diff pair) | Pixel clock × 10 / 2 (for TMDS) |
| DDC SDA / SCL | 2 pins | I²C, 100 kHz, for EDID read |
| HPD | 1 pin | Hot Plug Detect — pulled high by display |
| +5V | 1 pin | 5V supply from source to display (~55 mA max by spec) |
| CEC | 1 pin | Consumer Electronics Control — optional, can float |
| ARC / eARC | 1 pin | Audio Return Channel — not needed for ultrasound |
| Shield / GND | 5 pins | Return and cable shield |

For a DVI-compatible HDMI output (video only, no audio, no HDCP), the mandatory signals are: **4 TMDS differential pairs + DDC I²C + HPD + 5V supply + GND**. CEC, ARC, and eARC can be left unconnected or pulled to GND through a resistor.

### RP2350B implementation: HSTX

The RP2350B includes an HSTX (High-Speed Transmit) peripheral designed for DVI/HDMI-compatible output:

- **8 HSTX output pins** → 4 differential pairs (3 data + 1 clock) mapped directly to HDMI TMDS
- **TMDS encoding in hardware** — the HSTX peripheral generates 8b/10b TMDS symbols, offloading the CPU
- **Clock rates:** pixel clock up to ~150 MHz, supporting up to 1280×720@60 Hz (720p) or 1024×768@60 Hz; 1920×1080@30 Hz has been demonstrated with careful tuning
- **DMA-fed scanline buffer** — frame data in SRAM or PSRAM is streamed via DMA to the HSTX FIFO at scanline rate; dual-core allows one core to reconstruct while the other drives display
- **No external serializer needed** — the HSTX output can be DC-coupled (for DVI) or AC-coupled (for HDMI/DVI compatibility) directly to the connector with only passive components

The pico-extras repository includes a working DVI/HDMI library (`libdvi`) targeting RP2040 PIO; RP2350B with native HSTX is simpler and more stable. The HSTX pinout on RP2350B is fixed: GPIO 12–19 (HSTX0–HSTX7).

### Signal integrity requirements

TMDS differential pairs require careful PCB treatment:

| Requirement | Spec | Notes |
|-------------|------|-------|
| **Differential impedance** | 100 Ω ± 10% | 50 Ω each trace to adjacent GND plane |
| **Trace width** | ~85–90 µm on JLC04161H-7628 (4-layer, 0.155 mm prepreg) | Use controlled-impedance stackup |
| **Intra-pair length match** | ≤ 0.5 mm (≤ 3.3 ps skew) | Match P and N traces within each pair |
| **Inter-pair length match** | ≤ 5 mm | Match all 4 pairs (3 data + clock) to each other |
| **Reference plane** | Continuous GND plane under all TMDS traces | No splits, no vias, no copper pours crossing under pairs |
| **Routing layer** | Top or inner signal layer adjacent to GND plane | Avoid bottom-layer TMDS if it increases via count |
| **Via count** | Zero vias on TMDS pairs if possible | Each via adds ~0.2–0.5 ps skew and ~0.3 dB insertion loss at 740 MHz |
| **Trace spacing** | ≥ 3× trace width between pairs (edge-to-edge) | To maintain differential mode and suppress crosstalk |
| **ESD protection** | Required — TVS array across each pair | See below |

#### ESD protection

HDMI connectors are user-facing and ESD-exposed. Required:

| Part | Clamping | Capacitance | Package | Notes |
|------|----------|-------------|---------|-------|
| **PRTR5V0U2X** (NXP) | ±5V | 0.25 pF | SOT-363 | 2-channel, 0.25 pF each — very low capacitance, good for TMDS |
| **TPD4E05U06** (TI) | ±6V | 0.35 pF | SC-70 | 4-channel in one package — covers two TMDS pairs |
| **USBLC6-4SC6** (STMicro) | ±5V | 0.5 pF | SOT-23-6 | 4-channel; slightly higher C but widely available |

Place ESD devices as close to the HDMI connector as possible (within 5 mm), before the AC coupling capacitors. Route: connector pin → ESD device → AC cap → HSTX GPIO.

#### AC coupling (HDMI-compatible DVI)

Insert 100 nF capacitors in series on each TMDS line (both P and N):

- Capacitor type: 0402 C0G/NP0, 100 nF, 10 V
- Placement: between ESD device and the HSTX GPIO pad
- Purpose: prevents DC bias from the RP2350B 3.3V HSTX output from violating HDMI/DVI receiver common-mode requirements; AC coupling is required by the DVI specification

Total passive count per TMDS pair: 2 × ESD clamp + 2 × 100 nF cap = 4 components per pair × 4 pairs = 16 passives for the full HDMI signal path (excluding DDC, HPD, 5V).

#### 5V supply

HDMI sources must provide 5V on pin 18 (Type A) / pin 14 (Type C), max 55 mA. This powers the display's DDC/EDID ROM and HPD pull-up. On ultr4rick:

- If board runs on 5V input (USB-C PD or external 5V): connect directly via 27Ω series resistor for ESD current limiting
- If board runs on 3.3V only: use a small 3.3V → 5V boost or level-shift the 5V from the USB VBUS rail

#### DDC / EDID

DDC is I²C at 100 kHz from the display to the source. The RP2350B can read the display's EDID to auto-configure resolution. This requires:

- 2× 4.7 kΩ pull-up resistors to 3.3V on SDA and SCL
- Series 47 Ω resistors on SDA/SCL to protect against cable capacitance
- Connect to any I²C-capable RP2350B GPIO (not HSTX GPIO)

For a fixed-resolution design (e.g., always 720p60), DDC can be ignored and resolution hardcoded, but EDID read is good practice to detect monitor capabilities.

### Use case summary — HDMI for ultr4rick

| Scenario | Output type | Resolution target |
|----------|------------|-------------------|
| Standalone A-scan display (waveform only) | DVI-compatible HDMI | 640×480 or 800×600 |
| B-mode image (real-time or replay) | DVI-compatible HDMI | 1280×720 (720p) |
| Field NDT (connect to job-site monitor) | Full HDMI (with EDID negotiation) | Auto, typically 1920×1080 |
| Conference/demo | Full HDMI | 1280×720 or 1920×1080 |

---

## VGA

### Physical connector

Standard VGA uses a DE-15 (HD-15) female connector on the source board. The connector is large — 30 × 12 mm body — which is a significant board-space penalty for ultr4rick's compact target. VGA is retained as an option for:

- Legacy lab oscilloscopes with VGA monitor input
- Industrial displays without HDMI
- Teaching/lab contexts where VGA monitors are still in stock

For new designs, prefer HDMI. Include VGA only if a specific display target requires it, or as a second connector option on a larger board.

| Parameter | Value |
|-----------|-------|
| **Standard** | VESA VGA (DE-15, 15-pin HD D-Sub) |
| **Signals** | 3 analog RGB (0–0.7V, 75Ω), H-sync (TTL), V-sync (TTL) |
| **Resolution range** | 640×480 (VGA) to 1280×1024 (SXGA) — analog bandwidth limited |
| **Max bandwidth** | 350 MHz analog (component clock) at high res; 25 MHz for 640×480 |
| **Connector part example** | Amphenol L77SDA15SA4CH4F (right-angle SMT), TE 5747845-4 (through-hole) |
| **Connector size** | ~31 × 12 mm (right-angle); dominates board edge |

### RP2350B implementation

The RP2350B has no dedicated analog output. VGA requires an external approach:

#### Option A — R-2R resistor DAC + PIO (lowest cost, simplest)

Use PIO to generate H/V sync and a bitstream; an R-2R resistor ladder on GPIO pins produces the analog RGB voltages.

- 3-bit per channel (8 levels): 9 GPIO pins for RGB + 2 for H/V sync = 11 GPIO total
- 4-bit per channel: 12 GPIO + 2 sync = 14 GPIO total
- Pixel clock: PIO runs at system clock (up to 150 MHz) with divider; 640×480@60Hz requires 25.175 MHz pixel clock → PIO clock divider of 5.96 from 150 MHz system clock
- Output resistors: 75Ω termination at connector; R-2R ladder values: R = 150Ω, 2R = 300Ω (standard VGA R-2R values for 3.3V GPIO driving 75Ω)
- Signal quality: adequate for 640×480 and 800×600; above 1024×768 the R-2R output is too slow without a dedicated buffer

The Raspberry Pi Pico SDK and pico-playground include working VGA examples using this approach.

#### Option B — Dedicated VGA DAC (higher quality)

A small VGA DAC chip converts digital RGB to the 0–0.7V analog levels with proper drive strength:

| Part | Bits | Max pixel clock | Supply | Package | Notes |
|------|------|----------------|--------|---------|-------|
| **THS7316** (TI) | — (video amp, not DAC) | 250 MHz | 3.3/5V | SOIC-8 | RGB buffer/filter; pair with PWM DAC |
| **ADV7123** (ADI) | 10-bit | 330 MHz | 3.3V | LQFP-48 | Full DAC; parallel RGB input |
| **CH7301C** (Chrontel) | 24-bit | 165 MHz | 3.3V | TQFP-64 | HDMI → VGA encoder; takes TMDS input |

For ultr4rick, if VGA is needed, **Option A (R-2R)** is the pragmatic choice: zero additional ICs, 640×480 is more than sufficient for A-scan or a simple B-mode image, and the 11 GPIO pins are a manageable cost. Option B makes sense only if 1080p VGA or high-quality analog output is required.

### Signal integrity — VGA

VGA is single-ended analog, considerably more forgiving than TMDS, but still has requirements:

| Requirement | Spec | Notes |
|-------------|------|-------|
| **RGB impedance** | 75 Ω to GND | Each color channel terminated at the connector |
| **Voltage swing** | 0–0.7 V | 1V p-p including sync on composite; component = 0.7V for white |
| **DC offset** | 0 V (DC-coupled to monitor) | No AC coupling on VGA RGB; signals are DC-referenced |
| **H/V sync levels** | TTL (0/3.3V) | Standard 3.3V GPIO drives sync directly |
| **Sync polarity** | Positive or negative | Encoded in EDID / mode line; most monitors auto-detect |
| **Trace length** | Keep RGB traces short and equal | Differential skew between R, G, B shows as color fringing |
| **ESD** | Recommended on all 15 pins | Standard ESD TVS array (e.g., PRTR5V0U2X) per 2 channels |
| **EMI** | Ferrite beads on RGB lines (optional) | 600 Ω @ 100 MHz, e.g., BLM15AX601SN1D, help reduce radiated EMI |

The 75Ω termination resistors belong on the board, close to the connector. The R-2R DAC output nodes must each be terminated to GND via 75Ω before the connector to set the correct output impedance; without termination, reflections from the cable cause ghosting.

---

## Summary: HDMI vs VGA for ultr4rick

| Criterion | HDMI (Type C, DVI-compatible) | VGA (R-2R DAC) |
|-----------|------------------------------|----------------|
| **Image quality** | Digital, lossless | Analog, adequate for A-scan |
| **RP2350B path** | HSTX (native, hardware TMDS) | PIO R-2R (software, GPIO) |
| **PCB area** | Mini HDMI: ~11 × 6 mm | DE-15: ~31 × 14 mm |
| **GPIO cost** | 8 (HSTX, fixed) | 11–14 (flexible) |
| **Component count** | ~20 (4 pairs × 4 passives + DDC) | ~18–24 (R-2R resistors) |
| **External chip needed** | No | No (R-2R) or Yes (Option B DAC) |
| **Signal integrity effort** | High — impedance control, length match | Low — analog, forgiving |
| **Max resolution** | 1920×1080 (720p practical) | 640×480 practical |
| **Display compatibility** | Universal (2010 onward) | Legacy-compatible |
| **Recommended for ultr4rick** | **Yes — primary display output** | Optional — secondary / legacy |

**Recommendation:** Include HDMI Type C as the primary display connector on ultr4rick. Reserve GPIO pins for VGA R-2R only if the target use case involves legacy lab displays. The two are mutually exclusive on pin assignment (HSTX GPIO 12–19 cannot double as R-2R outputs), so the choice must be fixed at PCB design time unless both connectors are populated with a switch.

---

## Transducer Connectors (reference)

For full analysis of transducer connectors across all surveyed systems, see **`phase5.md §5.5`**. Summary:

| System | Connector | Standard |
|--------|-----------|----------|
| pic0rick | SMA edge-launch | Lab universal — any 50Ω probe |
| WULPUS | Hirose DF52-16S-0.8H(21) FFC | Wearable — thin, short, proprietary |
| ultr4rick (planned) | SMA (inherit from pic0rick) + FFC option for mux PMOD | Dual-mode |

For ultr4rick, SMA is the correct primary transducer connector for the same reasons as pic0rick (Vostrikov's Obstacle 3: avoid transducer lock-in). The MAX14866 mux PMOD can add 8-element array support via a 2×8 pin header, matching the pic0rick expansion path.

---

*Analysis compiled July 2026 for ultr4rick PCB design. RP2350B HSTX specs from RP2350 datasheet Rev. 1 and pico-extras/libdvi. HDMI specs from HDMI 1.4 specification and IEC 61076-3-106. VGA specs from VESA CVT standard and Raspberry Pi Pico SDK vga_composite example.*
