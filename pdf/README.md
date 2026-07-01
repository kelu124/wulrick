# pdf/ — Papers and Schematics

*PDFs committed to this repository for offline reference. All files are publicly available from their original sources.*

---

## AnatomyWearableUltrasoundSystem.pdf

**Source:** https://zenodo.org/records/21030966
**Title:** Anatomy of a Wearable Ultrasound System — Open-Sourcing
**Authors:** Christoph Leitner, Marco Giordano (ETH Zurich, Integrated Systems Laboratory)
**Venue:** IEEE CEEUS Workshop, Warsaw, 2026
**Format:** Presentation slides (23 pages)

### Relevance to the pic0rick vs WULPUS comparison

This is a workshop presentation from the same ETH Zurich group that built WULPUS and TinyProbe, given in 2026 — after both devices were published. It is the most direct synthesis of what they have learned across their wearable ultrasound platform series.

**Design philosophy framing.** The core design principle the authors articulate is: *"Every Echo Must Justify Its Cost."* This framing — treating acoustic, electrical, and computational resources as equally constrained — provides the conceptual vocabulary missing from the individual WULPUS papers. It explains design choices that the phase1–6 analysis identified as "architectural decisions" without a unifying rationale: fixed gain (echo cost minimization over depth flexibility), 8 Msps ADC ceiling (sampling rate traded for power), BLE over USB (telemetry cost justified by application).

**5-system comparison table.** The slides include a side-by-side comparison of WULPUS, USoP, EchoLite, TinyProbe, and PuLsE — exactly the five systems documented in `OtherSystems/`. This is the primary source for the specs in that folder. The table uses volume (cm³) rather than weight (g) as a form-factor metric, alongside per-channel power (mW) and center frequency (MHz). TinyProbe is highlighted as achieving the best efficiency: 2 mW/MHz.

**System anatomy taxonomy.** The paper decomposes wearable ultrasound into five canonical blocks:
1. **TX** — pulser + T/R switch (acoustic energy generation)
2. **RX** — LNA → PGA → ADC (signal conditioning)
3. **Control** — sequencer + measurement controller
4. **Compute** — RF → Envelope/IQ → Feature → Semantic (data compression pipeline)
5. **Host** — user I/O, storage, cloud

This maps directly onto the functional block structure used in `phase1.md` of this comparison. The "Compute" block — the pipeline from raw RF to semantic output — is the block where pic0rick and WULPUS diverge most: pic0rick offloads everything to the host PC; WULPUS compresses on-chip (envelope detection, then BLE transmission of reduced-bandwidth data).

**Three design knobs.** The slides identify the three primary parameters that drive system design:
1. **RX channel count** — each channel adds linearly to data rate and power
2. **TX center frequency** — governs Nyquist rate, axial resolution, and penetration depth (inverse relationship)
3. **PRF (Pulse Repetition Frequency)** — temporal resolution, with hard ceiling at c/(2·d_max) to prevent echo aliasing

**ModulUS platform.** The presentation describes ModulUS — a modular, open-source wearable ultrasound development sandbox using STM32H573 ARM Cortex-M33 MCU with a four-board architecture (TxRx, AFE, Control & Compute, Motherboard). This is the ETH group's current platform for exploration beyond WULPUS and TinyProbe. It includes bandwidth reduction circuitry (downconversion/filtering before ADC), and the STHVUP32 pulser.

**Direct connection to this research.** The Anatomy paper is, in effect, a retrospective design rationale document for everything the WULPUS line represents. For the `phase6.md` synthesis in this repository — particularly the gap analysis and cross-pollination notes — this paper provides the ETH group's own assessment of what matters and what doesn't in wearable ultrasound design. Its primary message: integration efficiency (mW/MHz/channel) is the right figure of merit, and both pic0rick and WULPUS are being superseded by platforms optimizing this metric explicitly.

---

## rheonics_ceeus2026.pdf

**Source:** https://rheonics.com/wp-content/uploads/2026/06/ceeus-2026-presentation.pdf
**Title:** Open-Source Wearable Ultrasound Platforms: A Rocky Path from Lab Prototype to Public Knowledge
**Author:** Sergei Vostrikov (Rheonics GmbH; formerly ETH Zurich IIS/PULP group — lead engineer on WULPUS and TinyProbe)
**Venue:** IEEE CEEUS Workshop, Warsaw, 2026
**Format:** Presentation slides

### Relevance to the pic0rick vs WULPUS comparison

This is Vostrikov's post-ETH retrospective on the open-source wearable ultrasound ecosystem, delivered from an industry vantage point (Rheonics). It directly addresses the openness, reproducibility, and sustainability of WULPUS and similar platforms, and names pic0rick explicitly in the comparative landscape.

**Confirmed hardware components (resolves todo.md Gap #1).** The slides document WULPUS's full front-end component list. Previously unconfirmed or approximate entries in WULPUS.md are now confirmed:
- HV mux: **HV2707** (was "~HV2707T provisional")
- T/R switch: **MD0101** (new — not previously documented)
- MOSFET driver: **MCP1416** (new — not previously documented)
- Amplifier: **OPA836** (was "external op-amp, fixed +10 dB")

**WULPUS community impact.** Vostrikov reports 100+ probes manufactured worldwide, 12+ academic and industrial partner groups, and 5+ derivative systems — quantifying the impact that the WULPUS papers do not directly state.

**WULPUS lifecycle.** May 2021 (idea) → Oct 2022 (IUS paper) → April 2023 (v1.0.0 GitHub) → April 2024 (full release) → Feb 2026 (open PCBA). The 18-month lag between IUS paper and first public release, and the further 12-month lag to fully open PCB sources, illustrates the "rocky path" of the title.

**Three-tier openness model.** The paper defines openness as a progression: *Public repo* (code/files available) → *Reproducible* (someone else can build it) → *Sustainable* (community can maintain without original authors). Most systems reach tier 1; few reach tier 3. pic0rick is explicitly rated "Open from inception, High reproducibility, Strong docs" — a better score than WULPUS, which is noted as "Vendor-dependent" due to Altium PCB tooling. This validates the phase5.md analysis of PCB tooling as a reproducibility barrier.

**Open-source landscape map.** The slides compare WULPUS, TinyProbe, USoP, pic0rick, open-UST, and SimpleRick on openness dimensions. pic0rick is listed alongside WULPUS as a peer platform, not a lesser tool — consistent with the framing in this repository.

**Lessons learned.** Vostrikov identifies three barriers to sustainable open-source hardware: (1) Altium dependency creates a modification barrier (→ switch to KiCad), (2) three C toolchains (TI CCS + Segger + Nordic SDK) create a firmware barrier, (3) transducer incompatibility limits user customization. All three align with constraints identified in `phase5.md` and `todo.md`.

---

## PuLsE_Giordano2025_arXiv.pdf

**Source:** https://arxiv.org/abs/2410.16219
**Title:** PuLsE: Accurate and Robust Ultrasound-based Continuous Heart-Rate Monitoring on a Wrist-Worn IoT Device
**Authors:** Marco Giordano, Christoph Leitner, Christian Vogt, Luca Benini, Michele Magno
**Venue:** IEEE Internet of Things Journal, Vol. 12, 2025
**DOI:** 10.1109/JIOT.2025.3557089 (check; arXiv version may differ)

See `OtherSystems/PuLsE.md` for analysis.
