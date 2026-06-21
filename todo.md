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
- [x] Phase 2 complete: `phase2.md` written and pushed
  - Master performance table: 65 Msps vs 8 Msps, 48 dB TGC vs 40.8 dB fixed, ±24V vs +15V, 22 mW vs ~300–400 mW
  - Flagged uncharacterized specs: system SNR, dynamic range, imaging depth not published for either device
  - ADC10065 alone (68.4 mW) exceeds WULPUS total system power (22 mW)
  - WULPUS-Pro (successor) noted: 16 ch, +30V, TGC, CMUT bias
- [x] Phase 3 complete: `phase3.md` written and pushed
  - Firmware: 1 PIO+C project (pic0rick) vs 3 C projects / 3 toolchains (WULPUS)
  - Host software: pic0lib Python API (numpy, HDF5, Jupyter) vs WulpusUssConfig dataclass + 68-byte binary protocol + ipywidgets GUI
  - WULPUS frame format documented: START\n + 2-byte acq num + 1-byte config ID + 800 bytes RF data
  - Customization barrier: pulse timing configurable from Python host (both); acquisition sequence requires C firmware edit (both)
- [x] Phase 4 complete: `phase4.md` written and pushed
  - pic0rick strengths: 65 Msps transducer range, TGC, PMOD modularity, Python-first, OSHWA, ecosystem
  - pic0rick weaknesses: USB-only, single channel base, power uncharacterized, not medical-grade, single author
  - WULPUS strengths: 22 mW, 8 channels, BLE, 9g, IEEE publications, ETH Zurich backing
  - WULPUS weaknesses: 8 Msps ceiling, fixed gain, BLE cap, 3 C toolchains, Altium lock-in
  - Published benchmarks table: pic0rick unmatched on frequency range; WULPUS unmatched on biomedical validation
- [x] Phase 5 complete: `phase5.md` written and pushed
  - Licenses: TAPR OHL + GPLv3 (pic0rick, copyleft) vs SHL-0.51 + Apache 2.0 (WULPUS, permissive)
  - PCB tools: KiCad free (pic0rick) vs Altium ~$10k/yr (WULPUS)
  - Costs: pic0rick DIY $100–$200, Tindie ~$200–$400; WULPUS DIY $250–$500, no pre-built
  - Community: 71 stars / 1 contributor / Matrix (pic0rick) vs 110 stars / 3 contributors / GitHub Discussions (WULPUS)
  - Reproduceability: both achievable; pic0rick easier (KiCad, JLCPCB-ready, Tindie)
- [x] Phase 6 complete: `phase6.md` written and pushed
  - Master comparison table across hardware, power, form factor, performance, software, openness, cost, community
  - Per-device summary cards with core technical identity, target user, excels/falls-short
  - 6 key differentiating design decisions with explicit tradeoff rationale
  - Gap analysis: 11 identified gaps (SNR, ENOB, PRF, resolution, imaging depth, HV mux part, etc.)
  - Cross-pollination notes: what each project could adopt from the other
  - Decision guide table: 16 use-case rows with recommended device

---

## To Do

*All phases complete.*
