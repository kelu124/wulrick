# Wulrick — To Do / Done

## Done

- [x] Create `wulrick` GitHub repo (kelu124/wulrick)
- [x] Initial research: pic0rick hardware specs, software ecosystem, use cases, creator
- [x] Initial research: WULPUS hardware specs, software ecosystem, use cases, creators (ETH Zurich IIS)
- [x] Write initial `plan.md` — 5-phase comparison research plan
- [x] Write `todo.md` — this file
- [x] Push `plan.md` and `todo.md` to repo
- [x] Revise `plan.md` v2 — add Phase 1 technical design analysis, reframe as strengths/weaknesses/specificities
- [x] Revise `plan.md` v3 — restructure Phase 1 by functional block (pulser, mux, TGC, ADC, power, timing, connectivity); add methodology/output format notes; incorporate Luc's guidance (datasheet+schematic only, no hardware, prose+tables output)
- [x] Push updated `plan.md` and `todo.md`

---

## To Do

### Phase 1 — Technical Design Analysis by Functional Block

**Pulser (TX Excitation)**
- [ ] Read MD1210 + TC6320 datasheets (pic0rick three-level bipolar pulser)
- [ ] Identify WULPUS MOSFET driver part from schematic; read datasheet
- [ ] Compare bipolar vs unipolar excitation: bandwidth, transducer efficiency, ringing
- [ ] Assess ±24V vs +15V voltage levels: penetration depth implications

**TX/RX Switch and Multiplexing**
- [ ] Identify pic0rick RX protection components from schematic
- [ ] Identify WULPUS HV multiplexer part; read datasheet
- [ ] Identify WULPUS TX/RX switch part; read datasheet
- [ ] Analyze 8-channel multiplexed approach vs single-channel

**TGC / Amplifier**
- [ ] Read AD8331 datasheet: noise figure, bandwidth, gain curve, input impedance
- [ ] Identify WULPUS amplifier part from schematic; read datasheet
- [ ] Analyze MCP4812 DAC role in pic0rick gain curve
- [ ] Compare TGC (dynamic) vs fixed gain tradeoffs

**ADC**
- [ ] Identify pic0rick external ADC part from schematic; read datasheet (bandwidth, ENOB, interface)
- [ ] Read MSP430FR5043 USS ADC specs: ENOB, integrated anti-aliasing
- [ ] Calculate theoretical axial resolution for each (c/2fs for typical tissue)
- [ ] Compare 10-bit @ 60 Msps vs 12-bit @ 8 Msps: dynamic range vs temporal resolution

**Power Supply**
- [ ] Identify pic0rick HV generation topology from HV board schematic
- [ ] Identify WULPUS HV generation topology from HV PCB schematic
- [ ] Analyze modular vs integrated HV supply: noise, flexibility, complexity
- [ ] Estimate pic0rick power from RP2040, ADC, AD8331 datasheet figures

**Digital Control and Timing**
- [ ] Read RP2040 PIO documentation: timing resolution, limitations for ultrasound
- [ ] Read MSP430FR5043 USS peripheral docs: what the dedicated sequencer automates
- [ ] Compare PIO flexibility vs USS peripheral efficiency
- [ ] Analyze WULPUS dual-MCU split: power gating and synchronization

**Connectivity**
- [ ] Determine actual USB throughput used by pic0rick (from firmware)
- [ ] Determine WULPUS decimation/compression to fit 8 Msps into 320 kbps BLE
- [ ] Compare data pipeline latency: USB vs BLE

### Phase 2 — Hardware Performance

- [ ] Compile key performance figures (single comparison table from datasheets + papers)
- [ ] Flag uncharacterized specs
- [ ] Collect SNR, dynamic range, imaging depth, resolution from published papers

### Phase 3 — Software and Firmware

- [ ] Survey pyUn0-lib API and documentation
- [ ] Survey WULPUS firmware repo structure and acquisition configuration
- [ ] Compare MicroPython vs C toolchain for customization ease
- [ ] Evaluate WULPUS Python GUI scope
- [ ] Trace and document full data pipelines for both devices

### Phase 4 — Strengths, Weaknesses, Specificities

- [ ] Verify and quantify pic0rick weaknesses (USB-only, single channel, power unknown)
- [ ] Investigate pic0rick specificities (PIO timing ceiling, three-level pulser implications)
- [ ] Verify and quantify WULPUS weaknesses (8 Msps limit, fixed gain, BLE ceiling)
- [ ] Investigate WULPUS specificities (MSP430 USS constraints, dual-MCU overhead)
- [ ] Extract quantitative benchmarks from Zenodo paper (pic0rick) and IEEE UFFC 2022 (WULPUS)

### Phase 5 — Openness, Community, Cost

- [ ] Verify pic0rick PCB tool; check Gerber exports
- [ ] Check WULPUS Altium export formats available
- [ ] Confirm pic0rick BOM cost (Tindie + component BOM)
- [ ] Estimate WULPUS BOM cost
- [ ] GitHub activity audit for both repos
- [ ] Find community channels

### Phase 6 — Synthesis

- [ ] Master comparison table (all functional blocks + performance + ecosystem)
- [ ] Per-device strengths/weaknesses/specificities summary
- [ ] Key differentiating design decisions
- [ ] Gap analysis
- [ ] Cross-pollination notes
