# Current Trends in Ultrasound Wearables: Spotlight on System Architecture

**Reference:** Weik, Nauber, Kaiser, Kirsch, Kunz, Schierling, Leitner, Benini, Liu, Zhou, Hampe, Fettweis, Herzog, Kupsch — IEEE Reviews in Biomedical Engineering, 2026
**DOI:** 10.1109/RBME.2026.3664011
**IEEE Xplore:** https://ieeexplore.ieee.org/document/11418920
**License:** CC BY
**PDF:** `pdf/Weik2026_WearableUltrasoundReview.pdf`
**Type:** Survey/review paper (21 pages, 2018–mid-2025 literature)

---

## What It Is

A comprehensive review of wearable ultrasound system architectures, co-authored by Christoph Leitner and Luca Benini (ETH Zurich — the WULPUS/TinyProbe/ModulUS group) alongside 12 researchers from other institutions. It surveys applications, Technology Readiness Levels, and design patterns across the field.

---

## Systems Cited

The paper reviews the full wearable ultrasound ecosystem. Systems from the OtherSystems/ comparison appear explicitly:

| System | Reference in paper | Notes |
|--------|--------------------|-------|
| WULPUS | [103] | "9g, multi-channel A-mode scanning, low power" |
| PuLsE | [88] | "compact wrist-worn ultrasound heart rate monitor" |
| USoP | [15] | "one of the first wearable imaging platforms" |
| TinyProbe | [109] | "32-channel multi-modal wireless ultrasound probe" |
| ModulUS | [113] | ETH group's modular sandbox platform |

**pic0rick is not cited.** This is consistent with its positioning as an instrumentation/research tool outside the biomedical wearable literature stream.

---

## Application TRL Landscape

| Application | TRL | Status |
|-------------|-----|--------|
| Prenatal care (GE Novii+) | 9 | Commercially available |
| Blood flow velocity (FloPath) | 8 | Large-scale deployment ready |
| Blood pressure monitoring | 7 | Validated in clinical settings |
| Cardiac imaging | 6 | Multi-subject human demos; clinical validation pending |
| Respiratory monitoring | 6 | Multi-subject human demos; pending operational deployment |
| Cerebral/transcranial Doppler | 6 | Multi-subject human demos |
| Neuromodulation | 4 | In-vitro phantom only |

WULPUS's primary applications (carotid, muscle) fit TRL 6–7: demonstrated on human subjects, not yet commercially validated.

---

## Central Argument

> *"Current wearable ultrasound development remains highly ad hoc with most prototypes being application-specific and custom-designed. Modular and reusable platforms could accelerate development across application domains."*

The paper argues for decoupling system-level engineering from domain-specific innovation — exactly the design philosophy that separates pic0rick (fully open, reconfigurable instrumentation) from WULPUS (optimized for specific monitoring tasks). The ETH group positions **ModulUS** as their answer to this need.

Market context: medical wearables market expected to exceed $3.7B by 2030 (8.2% CAGR), with ultrasound identified as a significant contributor.

---

## Relevance to pic0rick and WULPUS

1. **Leitner and Benini are co-authors** — this is the ETH group's 2026 field-level assessment. Their push for modular platforms is self-referential (ModulUS) but also validates the broader architectural direction.

2. **pic0rick's absence is meaningful.** The biomedical wearable ultrasound literature does not include pic0rick as a reference platform. This confirms the framing in `phase4.md` and `OtherSystems/README.md`: pic0rick is a complementary instrumentation tool, not a competitor to WULPUS within the medical wearables domain.

3. **TRL framework contextualizes the comparison.** WULPUS has pushed carotid and muscle monitoring to TRL 6–7 — validated in human subjects, approaching (but not at) clinical deployment. pic0rick has no published TRL for any application because it is not positioned for medical deployment.

4. **The "modular platform" argument** — central to this review — is the theoretical foundation for what pic0rick does in practice (via PMOD expansion, Python API, flexible timing). The review suggests this approach is where the field needs to go; pic0rick already embodies it for instrumentation use, while ModulUS is the ETH group's attempt to bring it to wearable deployment.

---

## Source

PDF downloaded from IEEE Xplore (CC BY license). Full text in `pdf/Weik2026_WearableUltrasoundReview.pdf`.
