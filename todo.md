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
- [x] Create `link.md` — research source tracker with paragraph descriptions per source
- [x] Push `link.md`
- [x] Phase 1 complete: `phase1.md` written and pushed
  - Pulser: MD1213+TC6320 three-level bipolar ±24V (pic0rick) vs MSP430 USS PPG +15V unipolar (WULPUS)
  - TX/RX: MD0100N8-G passive switch (pic0rick) vs SPI-controlled 8-ch HV mux (WULPUS)
  - Amp: AD8331 TGC 48 dB + MCP4812 DAC (pic0rick) vs integrated MSP430 PGA + power-gated op-amp, 40.8 dB fixed (WULPUS)
  - ADC: ADC10065 65 Msps 10-bit ENOB 9.6 (pic0rick) vs MSP430 SDHS 8 Msps 12-bit (WULPUS)
  - Power: ~300–400 mW estimated USB-tethered (pic0rick) vs 22 mW measured battery (WULPUS)
  - Timing: RP2040 PIO (flexible) vs MSP430 USS_A sequencer (automated, efficient)
  - Connectivity: USB High Speed (pic0rick) vs BLE 320 kbps via nRF52832 (WULPUS)

---

## To Do

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
