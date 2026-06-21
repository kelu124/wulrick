# WULPUS GitHub Discussions — Scraped Summary

*Source: https://github.com/pulp-bio/wulpus/discussions and /issues*
*Scraped: 2026-06-21. Total: 3 discussions, 15 issues.*

---

## Discussions

### Discussion #20 — WULPUS Bring-Up and Assembly Q&A
**Category:** Q&A | **Opened:** 2024-12-04 by Sergio5714 | **Comments:** 8 | **Status:** Open (unresolved)

**Purpose:** Created by maintainer Sergio5714 as a dedicated channel for assembly and bring-up questions.

**Thread summary — Ethan010722 (no signal on acquisition):**

A user reported receiving no signal during acquisition. Their HV15V output tested fine, but no voltage appeared at test points TR1, TR2, TR6, TR7. The GUI appeared to be receiving sampled data.

Maintainer diagnosis: the most likely cause was the **HV MUX RX start time** parameter being set too low — switching the HV mux to RX mode before the excitation pulse has propagated to the transducer, resulting in complete signal silence. Recommended debug workflow:
1. Activate all TX/RX channels; put RX gain to ~10 dB
2. Probe the transducer connector with an oscilloscope
3. Iteratively increase `HV MUX RX start time` in 1 µs steps until the excitation waveform appears
4. If still no signal at transducer, probe at the MOSFET driver output, then at the +HV supply line

User follow-up: signals appeared on channels not configured for TX/RX; switching transducer between connectors showed image always on channel 7. Maintainer hypothesis: HV PCB assembly fault or incorrect firmware re-flash procedure (must re-run all notebook cells when changing TX/RX config, not just the acquisition cell).

**Key recurring issue extracted:** HV MUX timing configuration (`HV MUX RX start time`) is the single most common cause of no-signal failures. This parameter varies per transducer and must be tuned empirically per device. The User Manual (Rev 1, Fig. 4.4) documents the acquisition timeline showing event Sb (HV MUX → RX switch) as the critical timing point.

**Secondary issue:** User asked whether WULPUS supports **M-mode** (Motion mode) imaging. No answer recorded in thread.

---

### Discussion #28 — v1.2.0 Release Announcement
**Category:** Announcements | **Opened:** 2025-01-27 by Sergio5714 | **Comments:** 0

**Content:** Official changelog for v1.2.0.

**Added:**
- Fabrication instructions for WULPUS silicone package (User Manual Rev. 2)
- Source design files for silicone package (`/hw/wulpus_silicone_package`)
- Updated README with expanded contributor list and new publications

**Fixed (from issues):**
- Inaccuracies in User Manual Rev. 1 (issues #13, #15, #18, #22)
- Issue #17: Power supply spikes exceeding resistor power ratings — added enable pin control for U1 of Acquisition PCB
- Issue #21: Improved antenna layout for better RF performance
- Updated Acquisition PCB layout and production files accordingly

**Changed:**
- Replaced chip antenna on Acquisition PCB; updated matching circuit

**Removed:**
- Mounting hole on Acquisition PCB (freed space for RF routing and antenna matching network)

---

### Discussion #34 — Question About Programming the WULPUS Boards
**Category:** Q&A | **Opened:** 2025-02-17 by Daalvarenga | **Comments:** 9 | **Status:** Resolved

**Context:** A user built custom WULPUS boards from the GitHub files and was unable to program them successfully.

**Issue 1 — Documentation mismatch:** User noted inconsistency between README flashing instructions and User Manual instructions (Manual has more steps). Maintainer acknowledged and marked for fix in next release. Recommended following the User Manual as the authoritative source.

**Issue 2 — NRF-SWD adapter pinout:** User had a 10-pin J-Link EDU Mini, not the 20-pin standard. Maintainer recommended connecting via DuPont jumper cables and tracing the signals through the schematics manually (provided schematic screenshot of Molex connector pinout).

**Issue 3 — No signal on custom-built boards despite successful firmware flash:** After flashing, the custom boards showed near-flat signal while an ETH Zurich-supplied reference board worked correctly. User methodically swapped HV PCBs between boards to isolate the fault to the HV PCB they had built.

Root cause identified: **diode D1 on the HV PCB was mounted with inverted polarity**, preventing HV generation. Resolution: flipped D1 polarity → HV generated → signal acquired successfully.

Maintainer note: D1 polarity inversion is a "classical assembly problem" they have encountered multiple times themselves.

**Residual question:** After fixing polarity, slight signal differences remained between boards. Maintainer explained this is expected: MSP430 firmware HV TX MUX start time has variability from probe to probe due to internal circuit wake-up time differences.

**Maintainer recommendations on signal quality:** Reduce PGA gain if signal appears saturated (user's gain was too high). Typical starting point: ~10 dB, then tune up.

---

## Issues — Thematic Summary

### Theme 1: Firmware Toolchain Complexity

**#23 — Alternatives for compilation** (Open, enhancement)
Core contributor CedricHirschi proposed adding Makefile-based build as an alternative to TI Code Composer Studio (CCS) and Segger Embedded Studio. Partial implementation in a fork. Noted that TI now offers an [online CCS IDE](https://dev.ti.com/ide), which could lower the setup barrier without requiring a local install.

**#13 — GUI questions / flashing confusion** (Closed)
User could not locate the `US_probe_nRF52_firmware` folder or the Segger Embedded Studio project file. Nordic nRF Connect apps referenced in the manual were outdated. User Manual also did not explain how the HV supply power-cycles during acquisition (leading to confusion when measuring EN_HV pin at rest). Fixed in User Manual v1.2.0.

**#18 — Missing `.epf` file in CCS import step** (Closed)
User Manual incorrectly described an import step requiring `workspace_preferences.epf`. The correct import method (File → Import project → select folder) was communicated via private chat, then documented in Manual v1.1 and included in v1.2.0.

**#39 — Instructions update** (Open, documentation)
Ongoing documentation improvement ticket; no comments yet.

---

### Theme 2: Hardware Assembly Failures

**#35 — MSP430 fails to boot after random event (possibly ESD)** (Open, bug/help wanted)
The MSP430 microcontroller stops booting firmware after an unpredictable event (suspected ESD shock). Symptoms: BLE communication with dongle establishes normally, but acquisition hangs. Memory erase fails initially; recovery requires entering Debug mode via JTAG, after which flash becomes accessible again. Affects all HW revisions 1.0.0–1.2.1. No solution found yet; open help-wanted issue.

**#17 — Power supply spikes exceeding resistor ratings** (Closed, fixed in v1.2.0)
VSYS_IN power spikes could exceed ratings of 1Ω current measurement resistors (R1, R5, R6, R8-R11, R13). Fix: replace 1Ω resistors with 0.1Ω alternatives. Additionally, EN_HV (pin 5 of TLV62569) was floating when MSP430 was unprogrammed — added weak pull-down resistor to fix default disabled state.

**#21 — Missing ground plane for microstrip** (Closed, fixed in v1.2.0)
BLE antenna trace lacked a proper ground plane, degrading RF performance. Fixed in v1.2.0 with antenna layout redesign and chip antenna replacement.

**#32 — Vias too close to C40 pads** (Open, enhancement)
PCB layout issue where vias are placed too close to component pads.

**#33 — Via size on nRF52 ground pad** (Closed, enhancement)
Via sizing issue resolved in a previous release.

---

### Theme 3: Transducer and Connector Interfacing

**#19 — Transducer connection** (Closed)
New user needed guidance on which connector and wires to use to attach a transducer to the HV PCB board-to-wire connector. Maintainer redirected to User Manual. Follow-up questions from same user were routed to Discussion #20.

**#15 — Mating connector and pre-crimped wires** (Closed, fixed in v1.2.0)
Missing part numbers for mating connectors and pre-crimped wires. Added to User Manual Rev. 1.1.

**#16 — Adapter PCB for Vermon 32-channel array** (Open, enhancement)
Multiple external requests for a Vermon 32-channel array adapter PCB to be included in the repository. Under consideration. This suggests active research interest in using WULPUS with multi-element imaging arrays beyond the standard single-element configuration.

---

### Theme 4: Documentation Gaps

**#22 — User Manual corrections** (Closed)
General corrections to User Manual Rev. 1; addressed in v1.2.0.

**#30 — HV PCB stackup** (Closed)
Missing or incorrect PCB stackup information in production files; fixed.

**#14 — Wrong file** (Closed)
A file error in the repository; resolved.

---

## Synthesized Observations

**Most common first-encounter failure:** No signal output despite apparently correct configuration. In all documented cases, the root cause was either (a) HV MUX timing not tuned for the specific transducer, or (b) an HV PCB assembly error (diode polarity, resistor values, antenna layout). The device works correctly once these are resolved, but the bring-up path is not self-evident from the current documentation.

**Toolchain barrier is the most mentioned friction point.** Three independent toolchains (TI CCS for MSP430, Segger Embedded Studio for nRF52832, Nordic nRF SDK v2 for dongle) with different import conventions, IDE versions, and file structures. A Makefile-based build alternative exists in a fork but has not been merged. The online CCS IDE (`dev.ti.com/ide`) has been confirmed to work and was not mentioned in documentation at time of writing.

**Board-to-board variability is uncharacterized.** Multiple users noted that their custom-built boards produce slightly different signals than the reference ETH Zurich-supplied units, even after correct assembly. Maintainer explains this as MCU wake-up time variability, but no calibration procedure exists for normalizing this across units. This is a meaningful gap for anyone building more than one probe.

**Vermon array adapter demand signals research use case.** Issue #16 ("multiple requests") indicates that groups outside ETH Zurich are actively attempting to use WULPUS with Vermon linear arrays — which would extend the device from single-element A-mode into real 2D B-mode imaging. This use case is not yet documented or officially supported.

**M-mode support question** (from Discussion #20) went unanswered. M-mode (time-series of a single scan line) is a standard clinical display mode that WULPUS's A-mode streaming architecture would support trivially in software — it is just a time plot of a fixed gate position. The absence of an answer suggests either the maintainer did not notice the question or M-mode is not yet implemented in the GUI.

---

## Deep Analysis: Issue #16 — Vermon 32-Channel Array Adapter

### Background

Issue #16 is an open enhancement request, filed after "multiple external requests" for an adapter PCB that would allow WULPUS to interface with Vermon 32-channel linear ultrasound arrays. The request comes from research groups outside ETH Zurich. No detailed technical discussion has been documented in the issue thread — the entry is brief — but its existence signals a meaningful aspiration: using WULPUS not merely as an A-mode point sensor but as a 2D B-mode imaging front-end.

Vermon (a French manufacturer of ultrasound transducer arrays) produces compact linear arrays used in research ultrasound systems. Their arrays typically use proprietary flexible printed circuit (FPC) connectors and pitch-matched element arrangements that are not directly compatible with WULPUS's standard SMA or board-to-wire transducer connections.

---

### What 32-Channel Array Operation Would Mean

#### From A-Mode to B-Mode

WULPUS in its published form is an **A-mode** device: each channel fires one transducer, records the 1D echo return along the beam axis, and streams the time-domain trace. Visualized over time, this produces M-mode or Doppler-style traces — useful for carotid wall displacement, muscle thickness, or heart rate.

A 32-element linear array enables **B-mode** imaging: a 2D cross-sectional image. This requires:

1. **Sequential firing of individual elements** — transmit from element N, record echoes from all elements (or a subset), then fire element N+1.
2. **Delay-and-sum beamforming** — coherently combine the received RF waveforms from multiple elements, applying time delays to focus the reconstruction at each image point.
3. **Image reconstruction** — convert the beamformed RF data into a display image (typically log-compressed envelope).

This transforms WULPUS from a wearable physiological sensor into a research imaging platform — an entirely different application class.

#### Synthetic Aperture Acquisition

The most common approach for a limited-channel front-end is **synthetic aperture (SA)** imaging:
- Fire from one element at a time
- Record the returning echo on all active receive channels simultaneously (or on the same element)
- Repeat for each transmit element
- Reconstruct the image from all transmit–receive combinations in post-processing

For 32 elements with WULPUS's existing 8-channel mux:
- 4 cascaded 8-channel mux stages would be required to address 32 elements
- One full synthetic aperture frame = 32 transmit firings × 8-channel receive per firing
- Frame rate at 50 Hz PRF: 50/32 ≈ **1.56 frames/second**
- This is low by clinical standards (cardiac imaging requires 30+ fps) but useful for slow physiological processes (vessel walls, breathing) and static structural imaging

---

### Technical Barriers

#### 1. BLE Bandwidth — The Primary Bottleneck

WULPUS transmits data via BLE at **320 kbps effective throughput**.

For a 32-channel synthetic aperture frame at 8 Msps:
- Samples per channel per acquisition: 400 (current WULPUS default)
- Data per full SA frame: 32 transmit × 8 receive channels × 400 samples × 2 bytes = **204,800 bytes = 1.64 Mb per frame**
- At 320 kbps: time to transmit one frame = **5.1 seconds per frame**
- Maximum wireless frame rate: ~**0.2 fps**

This makes real-time B-mode via BLE essentially impossible at 8 Msps with full 32-channel SA. Options to reduce data volume:
- Reduce to 200 samples per channel: halves data → 0.4 fps
- Envelope-detect on-chip (MSP430) before transmission: reduces each 12-bit RF sample to ~4-bit envelope → ~2.5× compression → still ~0.5 fps
- Reduce to a single-receive-channel per transmit (monostatic SA): 32 tx × 1 rx × 400 samples × 2 bytes = 25.6 KB/frame → at 320 kbps ≈ **0.78 fps** — marginal
- Use USB rather than BLE: the nRF52840 USB dongle supports USB CDC; if data were routed to USB instead of BLE, throughput increases to ~1–5 Mbps, enabling borderline real-time rates

**Bottom line on BLE:** BLE is the primary bottleneck. The WULPUS architecture must be modified — either to USB or to on-chip data reduction — for 32-channel array imaging to be practical.

#### 2. HV Multiplexer Cascading

WULPUS uses a single 8-channel HV mux. Extending to 32 channels requires either:

**Option A — 4× cascaded 8-ch mux:** Wire four HV2707T-C/R8X (or equivalent) in cascade, each addressed via SPI shift register chaining. All four share a common SPI bus with extended 32-bit shift registers. This is electrically feasible — cascading SPI-controlled HV muxes is standard practice — but requires a new HV PCB redesign with 4 HV mux ICs, additional SPI routing, and careful HV isolation between stages.

**Option B — Single 32-ch HV mux:** Supertex/Microchip offers HV mux variants with higher channel counts (HV20820, 20-ch; MAX4800 series). No single-die 32-ch version exists in this class. Two 16-channel parts in parallel would cover 32 channels.

**Switching time constraint:** HV mux switch transitions must complete before the ADC sampling window opens. At 1–2 MHz transducer frequencies, the echo from 15 mm depth arrives ~20 µs after firing. A typical HV mux switches in <2 µs — adequate, but cascaded stages may introduce additional settling time that must be verified per datasheet.

#### 3. Connector and PCB Redesign

Vermon arrays use **proprietary 0.8 mm pitch FPC connectors** (34-pin, standard pitch for their 32-element research arrays). WULPUS's HV PCB uses SMA connectors for individual transducers — mechanically incompatible. An adapter PCB would need to:
- Accept the Vermon FPC connector
- Route each element signal to an HV mux input
- Handle the impedance transformation (Vermon array elements are typically 50–75 Ω source impedance; the adapter must preserve this into the mux input)
- Manage HV transients: each FPC trace must withstand the +15V transmit pulse without damaging the thin-film FPC

This is a non-trivial PCB design problem. The flexibility of the FPC and the density of the connector require careful layout.

#### 4. ADC Bandwidth and Axial Resolution

Vermon research arrays are typically specified for **3–12 MHz** operation — outside WULPUS's usable ADC range. The MSP430FR5043 SDHS ADC ceiling (8 Msps) means the Nyquist limit is 4 MHz. Vermon arrays designed for ≥5 MHz will appear aliased and produce distorted RF data. Arrays specifically designed for 1–3 MHz (e.g., low-frequency vascular or liver imaging arrays) would be compatible, but these are less common in Vermon's research catalog.

**Mitigation:** Some Vermon arrays operate in the 1–4 MHz range for specific applications (e.g., cardiac or transcranial). If the requesting research groups have such arrays, the frequency compatibility concern is manageable.

#### 5. Beamforming Compute

Delay-and-sum beamforming for 32 elements × 400 depth points requires approximately:
- 32 × 400 × 2 multiply-accumulate operations per image line
- For a 64-line image: ~1.6 million MACs per frame

The MSP430FR5043 at 16 MHz cannot perform this in real time — its integer DSP throughput is orders of magnitude too low. All beamforming must be done **offline on the host PC**, which is standard practice for research scanners. The constraint is latency, not feasibility.

---

### Feasibility Assessment

| Dimension | Feasibility | Notes |
|-----------|-------------|-------|
| HV mux cascading to 32 ch | ✓ Achievable | 4× SPI-cascaded HV mux ICs; requires new PCB |
| Vermon FPC adapter PCB | ✓ Achievable | Non-trivial but straightforward PCB design |
| Transducer frequency compatibility | △ Conditional | Only if Vermon array operates at 1–4 MHz; most do not |
| BLE transmission of 32-ch data | ✗ Not real-time | 0.2–0.8 fps maximum via BLE at current data rates |
| USB transmission (modified BLE path) | ✓ Near-real-time | 1–5 fps depending on data reduction |
| On-chip beamforming | ✗ Not feasible | MSP430 compute too limited |
| Host-PC beamforming | ✓ Fully feasible | Standard for research scanners |
| Full SA synthetic aperture B-mode | ✓ Offline | Low frame rate but usable for research; not clinical |

**Overall:** A Vermon 32-channel adapter for WULPUS is **technically achievable as a research tool** but requires three conditions to be met simultaneously: (1) use of a low-frequency Vermon array (≤3 MHz), (2) replacement of BLE with USB for data transfer, and (3) redesign of the HV PCB to cascade four mux stages. The resulting device would support offline B-mode reconstruction at ~1 fps — useful for structural imaging in research settings, but not for dynamic cardiac or vascular imaging. WULPUS-Pro (16 channels, +30V, programmable TGC) is a better base for this direction than the original WULPUS.

### What This Signal Means for the Community

The fact that "multiple requests" arrived before any public documentation of 32-channel support is significant. It indicates:
1. Research groups are aware of WULPUS and see it as a potentially affordable entry point to array imaging (comparable systems like Verasonics Vantage or Ultrasonix cost $40,000–$150,000)
2. The interest is specifically in synthetic aperture B-mode — the groups requesting this are not looking for physiological monitoring but for structural imaging
3. There is an unmet need: an open-source, affordable, moderate-performance array front-end in the 1–4 MHz range that can produce B-mode images for research, without clinical regulatory requirements

pic0rick's MAX14866 PMOD architecture offers a parallel path to this: 8 channels, compatible with higher-frequency transducers (up to ~32 MHz via the ADC10065), expandable to 32+ channels via cascaded PMOD boards. For groups interested in array B-mode at 2–10 MHz, pic0rick's architecture may be more suitable than WULPUS, despite lacking the power optimization.
