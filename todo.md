# Wulrick — To Do / Done

## Done

- [x] Create `wulrick` GitHub repo (kelu124/wulrick)
- [x] Initial research: pic0rick hardware specs, software ecosystem, use cases, creator
- [x] Initial research: WULPUS hardware specs, software ecosystem, use cases, creators (ETH Zurich IIS)
- [x] Write initial `plan.md` — 5-phase comparison research plan
- [x] Write `todo.md` — this file
- [x] Push `plan.md` and `todo.md` to repo
- [x] Revise `plan.md` — add Phase 1 independent technical design analysis; reframe as strengths/weaknesses/specificities mapping; expand to 6 phases
- [x] Push updated `plan.md` and `todo.md`

---

## To Do

### Phase 1 — Independent Technical Design Analysis

- [ ] Read pic0rick main board schematic from kelu124/pic0rick
- [ ] Read pic0rick pulser PMOD schematic
- [ ] Read pic0rick HV supply board schematic
- [ ] Reconstruct pic0rick full signal chain from schematics
- [ ] Read WULPUS Acquisition PCB schematic (Altium files, pulp-bio/wulpus)
- [ ] Read WULPUS High-Voltage PCB schematic
- [ ] Reconstruct WULPUS full signal chain from schematics
- [ ] Document and analyze key components: pic0rick (RP2040, AD8331, MCP4812, MD1210, TC6320, ADC)
- [ ] Document and analyze key components: WULPUS (MSP430FR5043, nRF52832, HV mux, MOSFET driver, amp chain)
- [ ] Assess PCB architecture: pic0rick (modularity, PMOD headers, layer count)
- [ ] Assess PCB architecture: WULPUS (two-PCB split, miniaturization constraints, HV isolation)
- [ ] Draw system block diagrams for each device
- [ ] Summarize what each design explicitly optimizes for and explicitly trades away

### Phase 2 — Hardware Performance

- [ ] Compare RP2040 PIO ultrasound timing vs MSP430FR5043 dedicated USS peripheral
- [ ] Quantify axial resolution: 60 Msps vs 8 Msps for typical transducer frequencies
- [ ] Analyze TGC (dynamic gain) benefit vs WULPUS fixed gain
- [ ] Evaluate WULPUS 8-channel multiplexing capabilities
- [ ] Compare bipolar vs unipolar transmit topology
- [ ] Determine pic0rick USB power draw
- [ ] Benchmark WULPUS BLE data throughput vs raw ADC rate

### Phase 3 — Software and Firmware

- [ ] Clone and run pic0rick MicroPython examples
- [ ] Clone and build WULPUS C firmware (MSP430 + nRF52832 + dongle)
- [ ] Evaluate pyUn0-lib API coverage and documentation
- [ ] Evaluate WULPUS Python host tools and GUI
- [ ] Trace and benchmark full data pipelines (USB vs BLE)
- [ ] Check kelu124/us_rf_processing for B-mode reconstruction demos
- [ ] Survey signal processing tooling gaps in both ecosystems

### Phase 4 — Strengths, Weaknesses, and Specificities

- [ ] Document pic0rick strengths (high Msps, TGC, PMOD modularity, Python ecosystem, OSHWA cert)
- [ ] Document pic0rick weaknesses (USB-only, single channel, no power characterization, not medical-grade)
- [ ] Document pic0rick specificities (PIO-based timing, three-level bipolar pulser)
- [ ] Document WULPUS strengths (<25 mW, 8 channels, BLE, 13g form factor, published results)
- [ ] Document WULPUS weaknesses (8 Msps limit, fixed gain, BLE bandwidth, C toolchain, Altium lock-in)
- [ ] Document WULPUS specificities (MSP430 USS peripheral, dual-MCU power gating)
- [ ] Collect quantitative benchmarks from published papers for both devices

### Phase 5 — Openness, Community, Cost

- [ ] Verify pic0rick PCB tool (KiCad vs other)
- [ ] Identify Altium export formats in WULPUS repo
- [ ] Confirm pic0rick BOM cost from Tindie / repo
- [ ] Estimate WULPUS BOM cost from files and PCBWay
- [ ] GitHub activity audit: stars, forks, issues, last commit
- [ ] Find community channels for each project

### Phase 6 — Synthesis

- [ ] Master comparison table (all dimensions)
- [ ] Strengths/weaknesses/specificities summary card per device
- [ ] Key design decisions that most differentiate the platforms
- [ ] Gap analysis: what neither covers
- [ ] Cross-pollination notes
