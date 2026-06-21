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
