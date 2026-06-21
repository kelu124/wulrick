# Wulrick — To Do / Done

## Done

- [x] Create `wulrick` GitHub repo (kelu124/wulrick)
- [x] Initial research: pic0rick hardware specs, software ecosystem, use cases, creator
- [x] Initial research: WULPUS hardware specs, software ecosystem, use cases, creators (ETH Zurich IIS)
- [x] Write `plan.md` — detailed 5-phase comparison research plan
- [x] Write `todo.md` — this file
- [x] Push `plan.md` and `todo.md` to repo

---

## To Do

### Phase 1 — Hardware Deep-Dive

- [ ] Read pic0rick schematic and PCB files from kelu124/pic0rick
- [ ] Read WULPUS Altium PCB files and schematics from pulp-bio/wulpus
- [ ] Compare RP2040 PIO ultrasound timing vs MSP430FR5043 dedicated ultrasound peripherals
- [ ] Analyze analog front-end: AD8331 TGC (pic0rick) vs fixed-gain amp (WULPUS)
- [ ] Quantify axial resolution impact: 60 Msps/10-bit vs 8 Msps/12-bit ADC
- [ ] Compare transmit paths: three-level bipolar ±24V (pic0rick) vs +15V unipolar (WULPUS)
- [ ] Measure/estimate power draw of pic0rick from USB
- [ ] Benchmark WULPUS battery life at various acquisition rates

### Phase 2 — Software and Firmware

- [ ] Clone and run pic0rick MicroPython examples
- [ ] Clone and build WULPUS C firmware (MSP430 + nRF52832 + dongle)
- [ ] Evaluate pyUn0-lib API coverage and documentation quality
- [ ] Evaluate WULPUS Python host tools and GUI usability
- [ ] Trace and benchmark full data pipelines (USB vs BLE throughput)
- [ ] Check kelu124/us_rf_processing for B-mode reconstruction demos
- [ ] Survey signal processing tooling gaps in both ecosystems

### Phase 3 — Use Case Analysis

- [ ] Test pic0rick with a biomedical transducer (if possible)
- [ ] Test WULPUS with an NDT transducer (if applicable)
- [ ] Map transducer frequency compatibility for each platform
- [ ] Collect and compare published SNR/resolution/depth benchmarks
- [ ] Review all published papers: Zenodo (pic0rick), IEEE UFFC 2022 (WULPUS)

### Phase 4 — Openness, Community, Cost

- [ ] Confirm pic0rick BOM cost from repo/Tindie
- [ ] Estimate WULPUS BOM cost from Altium files and PCBWay listing
- [ ] Check PCB tool accessibility: KiCad vs Altium for reproduction
- [ ] Audit GitHub activity: stars, forks, open/closed issues, last commit
- [ ] Find community channels for each project (Discord, forums, mailing lists)

### Phase 5 — Synthesis

- [ ] Draft side-by-side comparison table (all dimensions from plan.md)
- [ ] Write "choose X if…" decision guide
- [ ] Identify hybrid approach possibilities
- [ ] Identify gaps: use cases neither platform covers well
- [ ] Final comparative summary document
