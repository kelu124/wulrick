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
