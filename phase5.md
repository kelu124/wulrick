# Phase 5: Openness, Community, and Cost

---

## 5.1 Open-Source Posture

| Dimension | pic0rick | WULPUS |
|-----------|----------|--------|
| Hardware license | TAPR Open Hardware License | Solderpad v0.51 (SHL-0.51) |
| Software license | GNU GPLv3 | Apache License 2.0 |
| Documentation license | CC BY-SA 3.0 | CC BY 4.0 |
| OSHWA certified | Yes (FR000023, Oct 2024) | No |
| PCB source tool | KiCad (open source, free) | Altium Designer (proprietary, ~$10k/yr) |
| PDF schematics exported | Yes | Yes |
| Gerber files exported | Yes | Yes |
| BOM available | Yes (CSV) | Yes (PDF + XLSX) |
| 3D models | Yes (.step) | Yes (.dwf, assembly drawings) |

### License comparison

TAPR OHL (pic0rick) and Solderpad v0.51 (WULPUS) are both hardware-specific open licenses. Both allow free use, modification, and distribution with attribution. Key differences:
- TAPR OHL requires derivative hardware to also be released under TAPR OHL (copyleft-style)
- Solderpad v0.51 is permissive (BSD-like), allowing use in proprietary derivatives without releasing changes

Apache 2.0 (WULPUS software) is more permissive than GPLv3 (pic0rick software): Apache 2.0 allows embedding in proprietary software; GPLv3 requires derivative software to be GPL-licensed.

### PCB tool accessibility

pic0rick uses KiCad — open source, free, runs on Linux/macOS/Windows, actively maintained. Anyone can open, inspect, and modify the schematics and PCB layout without cost. WULPUS uses Altium Designer — industry standard but expensive (approximately $10,000/year). The WULPUS team exports PDF schematics and Gerbers, making replication possible without Altium, but hardware modifications require Altium access or lossy format conversion.

**Practical replication verdict:** pic0rick is easier to replicate from source. WULPUS requires either Altium access or working from exported PDFs and Gerbers (which allows fabrication but not editing).

---

## 5.2 Cost

### pic0rick

| Item | Cost (estimated) |
|------|-----------------|
| Pre-built kit (Tindie) | ~$200–$400 range (specific price not confirmed; listed as <$500) |
| Main ADC board (DIY, JLCPCB) | ~$30–$60 PCB + ~$50–$100 components |
| Pulser PMOD (DIY, JLCPCB) | ~$20–$40 PCB + ~$30–$60 components |
| HV supply board (DIY, JLCPCB) | ~$10–$20 PCB + ~$10–$20 components |
| Raspberry Pi Pico (included on board) | ~$4 |
| KiCad (EDA tool) | Free |
| **Total DIY estimate** | **~$100–$200 for core system** |

The PCB files are JLCPCB-ready (Gerbers + BOM + CPL pick-and-place files). The JLCPCB assembly service can produce populated boards at competitive prices.

### WULPUS

| Item | Cost (estimated) |
|------|-----------------|
| Pre-built kit | Not available (DIY only) |
| Acquisition PCB (PCBWay, populated) | ~$150–$300 (v1.2.1 listed on PCBWay) |
| HV PCB (PCBWay, populated) | ~$80–$150 |
| Altium Designer (for modifications) | ~$10,000/year (or use exported files only) |
| nRF52840 USB dongle | ~$10–$20 (Nordic DK or clone) |
| **Total DIY estimate** | **~$250–$500 for core system** |

WULPUS is more expensive to reproduce because: (a) no pre-built option, (b) higher component cost (dual MCU, integrated BLE, more switches), (c) smaller production volumes. The PCBWay listings provide a reference for outsourced assembly.

### Cost comparison

| Aspect | pic0rick | WULPUS |
|--------|----------|--------|
| Pre-built availability | Yes (Tindie) | No |
| DIY PCB tool | Free (KiCad) | Expensive (Altium) or PDF/Gerber only |
| Estimated system cost | $100–$400 | $250–$500 |
| Component count (approx.) | ~25–30 per board, 3 boards | Higher (dual MCU, 5 schematic sheets) |

---

## 5.3 Community and Maintenance

| Metric | pic0rick | WULPUS |
|--------|----------|--------|
| GitHub stars | 71 | 110 |
| GitHub forks | 26 | 26 |
| Open issues | 2 | 6 |
| Contributors | 1 (kelu124) | 3 (Vostrikov, Frey, Hirschi) |
| Last commit | June 8, 2026 | June 21, 2026 (today) |
| GitHub Discussions | Disabled | Enabled |
| Community channel | Matrix chat (matrix.to) | GitHub Discussions |
| Academic papers using it | Zenodo paper (kelu124) | 5+ IEEE papers (ETH Zurich group) |
| Institutional backing | Individual (kelu124) | ETH Zurich IIS / PULP group |

### Assessment

**pic0rick:** Single-maintainer project with active recent commits and a small but present community (Matrix chat). The OSHWA certification and Tindie availability show the project's seriousness. Risk: single-author bus factor. Upside: kelu124 is responsive and actively developing (last commit 2 weeks before this analysis).

**WULPUS:** Multi-contributor research platform backed by an active academic group at ETH Zurich. GitHub Discussions is active. Multiple IEEE publications provide credibility and a body of evidence. Risk: academic research platforms sometimes stagnate when the lead PhD students graduate. Current activity (commit today) suggests the project is still in active development.

---

## 5.4 Reproduceability Assessment

**Can you build a pic0rick from scratch?**
Yes. KiCad source files, JLCPCB-ready BOMs, and Gerbers are all in the repo. Component sourcing: all parts are standard commercial ICs (ADC10065, AD8331, MCP4812, MD0100, MD1213, TC6320, Pico) available on Mouser/Digikey. Estimated lead time from ordering to assembled board: 2–4 weeks via JLCPCB assembly. Alternatively, available pre-built on Tindie.

**Can you build a WULPUS from scratch?**
Yes, with more effort. PDF schematics and Gerbers are available for fabrication. BOMs are provided (PDF/XLSX). The Altium source files are present but only useful if you have Altium. Component sourcing: MSP430FR5043 and nRF52832 are standard commercial ICs. The main challenge is the 3 separate firmware toolchains and programmers needed (MSP430 JTAG + nRF52 SWD for both the probe and dongle). WULPUS includes programming PCBs in the repo for this reason. Estimated lead time: 3–6 weeks, more complex setup than pic0rick.

---

## 5.5 Transducer Connectivity

*Source: hardware pick-place files and BOMs for pic0rick (kelu124/pic0rick) and WULPUS (pulp-bio/wulpus); OtherSystems files for remaining entries. Verified July 2026.*

### Connector types across surveyed systems

| System | Connection type | Part / standard | Notes |
|--------|----------------|-----------------|-------|
| **pic0rick** | Coaxial — SMA | JOHNSON 142-0701-801 (Amphenol 132357-11), edge-launch PCB | Single transducer port on ADC board. Mux board (MAX14866 PMOD) adds a 2×8 2.54 mm pin header for 8-element array connections |
| **WULPUS** | Flat flex cable (FFC) | Hirose DF52-16S-0.8H(21), 16-pin, 0.8 mm pitch | Confirmed from HV PCB pick-place file. 16 pins = 8 elements × (signal + GND). Board-to-board (HV ↔ Acquisition PCB): Samtec LSHM-120 40-pin, 0.5 mm |
| **TinyProbe** | FFC (wifi board); main array not released | Hirose DF52-2S-0.8H(21) on wifi_6_board (2-pin); main FPGA board hardware not in public repo | DF52-2S likely serves a single-element cal or sync channel. The 32-ch array connects the main board via a TE 2199230-5 card edge (67-pos, 0.5 mm) |
| **USoP** | No connector — embedded interconnect | Serpentine Cu/PI traces encapsulated in silicone | Cable-free by design. PZT elements, control IC, and battery all co-embedded in one flexible patch |
| **PuLsE** | Unknown | — | No published hardware. Wrist-worn single-element; likely direct bond or spring contact to PCB pad |
| **EchoLite** | Unknown | — | Paper not indexed as of 2026-06-30; no hardware files available |
| **ModulUS** | Unknown | — | Mentioned in Anatomy slides only; no public hardware files. Modular research platform likely uses SMA or custom coax per its multi-board architecture |

### U.FL / IPEX / MHF — absence confirmed

No U.FL (also marketed as IPEX MHF) connector appears in any of the hardware files for pic0rick, WULPUS, or TinyProbe. This is unsurprising:

- U.FL is rated to ~500 mating cycles and 0.5 W maximum power — marginal for repeated transducer swaps at ±24V pulse levels
- U.FL's 1.13 mm cable diameter is sub-optimal for maintaining 50Ω impedance at ultrasound frequencies (3–30 MHz)
- The wearable systems (WULPUS, PuLsE) optimise for flatness and flexibility rather than the coaxial architecture U.FL serves

### Connector choice as design philosophy

The connector type is a direct signal of the intended use context:

| Connector family | Use context | Systems |
|-----------------|-------------|---------|
| SMA | Lab instrument, interchangeable probes, 50Ω matched | pic0rick, ModulUS (likely) |
| Hirose DF52 FFC | Wearable: thin, light, short run, no impedance matching needed | WULPUS, TinyProbe |
| Embedded serpentine flex | Fully integrated patch: no connector at all | USoP |
| 2.54 mm pin header | Low-cost multi-element array connection (research/prototype) | pic0rick mux PMOD |

The FFC approach (WULPUS) and SMA approach (pic0rick) represent opposite trade-offs. FFC is lighter and thinner — correct for a wearable where the transducer-to-PCB distance is millimetres and both are inside a silicone mold. SMA is heavier and bulkier but universal — any lab probe with an SMA pigtail works immediately. The FFC lock-in contributes directly to Vostrikov's sustainability obstacle #3 (see §5.6).

---

## 5.6 Open-Source Sustainability: Vostrikov's Three Obstacles

*Source: Vostrikov, S. "Open-Source Wearable Ultrasound Platforms: A Rocky Path from Lab Prototype to Public Knowledge." IEEE CEEUS Workshop, Warsaw, 2026. (Rheonics GmbH / formerly ETH Zurich IIS/PULP.) PDF: `pdf/rheonics_ceeus2026.pdf`.*

Vostrikov defines open-source hardware sustainability as a three-tier progression:

> **Tier 1 — Public repo:** files are available  
> **Tier 2 — Reproducible:** someone else can build it  
> **Tier 3 — Sustainable:** the community can maintain it without the original authors

Most systems reach Tier 1. Few reach Tier 3. The three obstacles he identifies are the barriers between tiers for WULPUS specifically, and for the wearable ultrasound open-source ecosystem generally.

### Obstacle 1 — PCB tooling lock-in (Altium)

WULPUS hardware source files are in Altium Designer format. Altium Designer costs approximately $10,000/year for a commercial licence. Anyone wanting to modify the PCB — change a connector, fix a layout bug, adapt for a different transducer, reduce board size — either needs Altium access or must work from the exported Gerbers and PDF schematics, which are read-only.

This creates a *modification barrier*: replication (Tier 2) is achievable from Gerbers, but community modification (Tier 3) requires proprietary tooling. Vostrikov's recommendation: migrate to KiCad.

**pic0rick position:** Uses KiCad exclusively. Any contributor can open, modify, and re-export the PCB source without cost. This is the correct design choice for a community-maintained platform.

### Obstacle 2 — Multiple firmware toolchains

WULPUS firmware requires three separate development environments:

1. **TI Code Composer Studio (CCS)** — for the MSP430FR5043 acquisition MCU
2. **Segger Embedded Studio** — for the nRF52832 BLE MCU on the probe
3. **Nordic nRF5 SDK** — for the nRF52840 USB dongle firmware

Each has its own installer, project format, and licensing model. Setting up a complete WULPUS build environment is a multi-hour exercise. A contributor who fixes one firmware bug must navigate three toolchains to test end-to-end. This creates a *reproduction barrier* for the firmware half of the project even when hardware Tier 2 is achieved.

Vostrikov frames this as a consequence of the dual-MCU architecture: MSP430 (TI ecosystem) + nRF52 (Nordic ecosystem) = two incompatible toolchains by construction, plus a third for the dongle.

**pic0rick position:** Uses a single toolchain — the Raspberry Pi Pico SDK (CMake, arm-none-eabi-gcc) — for all firmware. The host software is pure Python. One toolchain, one SDK, one Python environment.

### Obstacle 3 — Transducer incompatibility

WULPUS uses a Hirose DF52-16S-0.8H(21) flat flex connector (16-pin, 0.8 mm pitch) on the HV PCB, matched to its specific 8-element transducer array geometry. Standard commercial ultrasound probes — which use Lemo 00, BNC, SMA, Fischer, or proprietary connector types — do not plug into WULPUS without a custom adapter or re-wiring.

This creates a *customisation barrier*: users cannot bring their own probe from an existing lab inventory. They are dependent on the specific transducer configurations the ETH group tested and qualified. Groups with different anatomical targets, frequencies, or probe geometries cannot simply swap the transducer.

Vostrikov identifies this as the largest practical barrier to adoption outside specialist groups, because the transducer is often the most application-specific and irreplaceable part of a user's setup.

**pic0rick position:** Uses SMA (JOHNSON 142-0701-801 edge-launch). SMA is a universal RF connector standard. Any commercial ultrasound probe with an SMA pigtail, and any home-built transducer with an SMA jack, connects directly. The MAX14866 mux PMOD adds an 8-element expansion path via a standard 2×8 header — accessible to any user with appropriate cabling, without a custom FFC harness.

### Summary

| Obstacle | WULPUS | pic0rick |
|----------|--------|----------|
| **1. PCB tooling** | Altium Designer (~$10k/yr) | KiCad (free, open source) |
| **2. Firmware toolchains** | 3 (TI CCS + Segger + Nordic SDK) | 1 (Pico SDK + Python) |
| **3. Transducer compatibility** | Proprietary FFC; custom transducer required | SMA; standard commercial probes compatible |
| **Tier 3 (Sustainable) achievable?** | Barrier at all three | No barriers identified |

Vostrikov's three-tier openness ratings from the CEEUS 2026 slides explicitly score pic0rick as: *"Open from inception, High reproducibility, Strong docs"* — the highest rating among the surveyed platforms. This matches the analysis above: pic0rick avoids all three obstacles by construction, not by retrofitting. The choice of KiCad, the Pico SDK, and SMA connectors are each individually mundane, but together they constitute a complete sustainability stack that WULPUS lacks.
