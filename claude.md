# claude.md — Agent Skill File: Ultrasound Hardware Comparison

*This file documents how to replicate the research agent that produced the pic0rick vs WULPUS comparison in this repository. It is a skill file: instructions for an AI agent, not a user manual.*

---

## Agent Purpose

This agent performs **systematic technical comparison of open-source ultrasound hardware platforms**. It works from public sources only — GitHub repositories, component datasheets, published papers — and produces structured markdown documents covering hardware design, firmware architecture, power budgets, software ecosystems, and research applications.

The methodology is designed to be reproducible: another agent with access to the same sources should reach substantially the same conclusions.

---

## What Has Been Completed

The comparison between **pic0rick** (kelu124/pic0rick) and **WULPUS** (pulp-bio/wulpus) is complete across seven phases. All output lives in the **kelu124/wulrick** GitHub repository.

See `README.md` for a summary of findings and the full file list. See `todo.md` for what's done and what open gaps remain.

---

## Methodology

### Guiding Principles

1. **No hardware in hand.** All specs come from datasheets, schematics, and published papers. Do not claim measured performance that is not published. When a spec is unavailable, say so explicitly and flag it as a gap.

2. **Block-by-block decomposition.** Compare each functional block independently before synthesizing. Blocks: pulser, TX/RX switch, amplifier/TGC, ADC, power supply, digital control/timing, connectivity, multiplexer/channel count.

3. **Typed spec attribution.** When citing specs, note the source type: **M** = measured (published paper), **D** = datasheet, **C** = calculated from datasheet, **E** = estimated. This prevents datasheet specs from being confused with measured system performance.

4. **Design decision framing.** For each difference between platforms, articulate the design decision: what tradeoff was made, why it was likely made, what it enables, what it constrains. Avoid value judgments ("X is better"); prefer "X enables Y at the cost of Z."

5. **Sequential file structure.** Research is organized into phases. Each phase produces one markdown file. Files are numbered (phase1.md through phase6.md) plus topical deep-dives. This makes it easy to continue from a specific point without re-doing prior work.

---

### Source Priority

| Source type | Trust level | Notes |
|-------------|------------|-------|
| Component datasheet (TI, Maxim, ADI, etc.) | High | Authoritative for component specs; not always for system specs |
| Published IEEE/arXiv paper | High for reported measurements | Authors may not report all relevant specs |
| GitHub repository (schematics, firmware) | High for design intent | May not reflect current production; read commit history |
| GitHub issues/discussions | Medium | Captures real-world problems; may be outdated |
| Community forum posts, blog posts | Low-medium | Cross-check against datasheets |
| Price estimates | Low | Volatile; quote directly for accuracy |

---

## Repository Structure

All files are in the root of **kelu124/wulrick**:

```
wulrick/
├── README.md           ← Entry point: key findings, file index, decision guide
├── claude.md           ← This file: agent skill documentation
├── plan.md             ← Research plan (Phases 1–7); evolves as work progresses
├── todo.md             ← Progress tracker; Done / To Do with gap documentation
├── link.md             ← Source tracker; one paragraph per source
│
├── phase1.md           ← Functional block analysis (pulser → connectivity)
├── phase2.md           ← Hardware performance master table
├── phase3.md           ← Firmware and software ecosystem
├── phase4.md           ← Strengths, weaknesses, specificities; benchmarks
├── phase5.md           ← Licenses, PCB tools, cost, community
├── phase6.md           ← Synthesis: master table, design decisions, gaps, guide
│
├── picorecommendation.md ← Recommendations for pic0rick (hardware + 7 SW improvements)
├── MSP430FR5043.md     ← Deep dive: MSP430FR5043 USS_A, ADC, PGA, FRAM, power modes
├── TUSS4470.md         ← Deep dive: TI TUSS4470 standalone AFE IC
├── TUSvsMSP.md         ← TUSS4470 vs MSP430FR5043 comparative analysis
├── array_sourcing.md   ← Sourcing guide: transducers and arrays for each platform
├── w_discussions.md    ← WULPUS GitHub Discussions/Issues + Vermon 32-ch analysis
└── w_uses.md           ← All known WULPUS research uses (7 papers, full citations)
```

---

## How to Continue the Research

### Starting a Fresh Session

Tell the agent:

> You are continuing a structured comparison of two open-source ultrasound platforms: pic0rick (kelu124/pic0rick) and WULPUS (pulp-bio/wulpus). All work goes into the GitHub repo kelu124/wulrick. Read README.md and todo.md first to understand what's been done and what gaps remain. Then read the specific file most relevant to your next task.

Use the GitHub Contents API to read files:
```
GET https://api.github.com/repos/kelu124/wulrick/contents/{filename}
```
Response includes base64-encoded content. Decode with `Buffer.from(data.content, 'base64').toString('utf8')` in Node.js.

### Pushing Files

```javascript
node --input-type=module << 'EOF'
import { readFileSync } from 'fs';
const REPO = 'kelu124/wulrick';

async function pushFile(path, localPath, message) {
  const content = readFileSync(localPath, 'utf8');
  const b64 = Buffer.from(content).toString('base64');
  let sha;
  const check = await fetch(`https://api.github.com/repos/${REPO}/contents/${path}`, {
    headers: { 'Accept': 'application/vnd.github+json', 'X-GitHub-Api-Version': '2022-11-28' }
  });
  if (check.ok) sha = (await check.json()).sha;
  const body = { message, content: b64 };
  if (sha) body.sha = sha;
  const res = await fetch(`https://api.github.com/repos/${REPO}/contents/${path}`, {
    method: 'PUT',
    headers: {
      'Accept': 'application/vnd.github+json',
      'Content-Type': 'application/json',
      'X-GitHub-Api-Version': '2022-11-28'
    },
    body: JSON.stringify(body)
  });
  const data = await res.json();
  console.log(res.ok ? `✓ ${path} (${res.status})` : `✗ ${path} FAILED: ${data.message}`);
}

await pushFile('filename.md', '/workspace/agent/filename.md', 'commit message');
EOF
```

**Important:** Push files **sequentially**, not in parallel. If two files with existing SHAs are fetched simultaneously and then both pushed, the second push will fail with a 409 SHA conflict because both GETs returned the same SHA.

---

## Key Technical Facts to Carry Forward

*These are the most frequently referenced facts across all phases. A fresh agent should have these in working memory.*

### pic0rick
- ADC: ADC10065, 65 Msps, 10-bit, ENOB 9.6 — from ADI datasheet (D), confirmed in kelu124 documentation
- Amplifier: AD8331 TGC, 7.5–55.5 dB, controlled via MCP4812 12-bit DAC over SPI
- Pulser: MD1213 + TC6320, three-level bipolar ±24V — produces ±VPULSE and 0V states for ring-down suppression
- TX/RX isolation: MD0100N8-G passive limiter (TVS-based, not an active switch)
- Mux PMOD: MAX14866, 8 channels, ±80V HV tolerance, SPI 16-bit shift register
- MCU: RP2040 (dual-core ARM Cortex-M0+, 133 MHz), PIO state machines for nanosecond-resolution timing
- Connectivity: USB (tethered), ~600 Mbps theoretical; actual throughput determined by firmware
- Power: ~300–400 mW estimated (uncharacterized; ADC10065 alone is 68.4 mW)
- Host library: pic0lib (Python); data stored in HDF5; Jupyter-based workflow
- License: TAPR OHL (hardware) + GPLv3 (software) — copyleft
- PCB: KiCad, OSHWA certified FR000023, JLCPCB-compatible
- Tindie price: ~$200–400 pre-built

### WULPUS
- ADC: MSP430FR5043 SDHS, 8 Msps, 12-bit — hardware ceiling, not firmware-configurable
- ENOB: **not published by TI** — this is a known gap
- Amplifier: MSP430 PGA (−6.5 to +30.8 dB, 46 steps) + external op-amp (+10 dB fixed) = 40.8 dB total, **fixed during acquisition**
- Pulser: MSP430 USS_A PPG (0–5 MHz, 0–30 pulses) → MOSFET driver → +15V unipolar
- HV mux: 8-channel SPI-controlled HV multiplexer, provisionally HV2707T-C/R8X (Supertex/Microchip) — **not confirmed from BOM**
- BLE: nRF52832, 320 kbps effective throughput — this is the primary data bottleneck
- FRAM: 64 KB, ~10¹⁵ write cycles, non-volatile, DMA-filled during acquisition
- Standby power: 450 nA (LPM4+RTC), 35 nA (LPM4.5 — not used at 50 Hz PRF)
- Total system power: 22 mW measured (includes HV supply, op-amp, nRF52832, MSP430 active+sleep)
- Form factor: 46×25 mm, 9 g
- Connectivity: BLE wireless + nRF52840 USB dongle on host side
- Host: Python GUI with WulpusUssConfig dataclass; 68-byte binary config packet
- Firmware: 3 separate C projects (TI CCS for MSP430, Segger Embedded Studio for nRF52832, Nordic SDK for dongle)
- PCB: Altium (≈$10k/yr license); Gerbers exported for fabrication
- License: SHL-0.51 (hardware) + Apache 2.0 (software) — permissive
- WULPUS-Pro: referenced, not yet public — 16 ch, +30V, programmable TGC, CMUT bias, 100 kHz–10 MHz

### TUSS4470 (TI)
- Standalone AFE IC — **no ADC, no MCU**
- Frequency range: 40 kHz – 1 MHz (industrial sonar/flow metering range)
- Transmit: H-bridge bipolar, up to 24V, transformer-less (VDRV capacitive charge pump)
- Receive: log-amp, 94 dB range (19–113 dB), demodulated envelope output on VOUT
- Output: **analog envelope only** — raw RF reconstruction not possible
- Power: 57.5 mW active (at 5V), 7.5 mW standby — not viable for wearable use
- Price: $1.63 (qty 1)
- Config: SPI (configuration only; echo data comes out on VOUT pin, not SPI)
- Datasheet: SLAS982

### MSP430FR5043
- Integrated MCU + AFE on single die — the WULPUS acquisition chip
- USS_A PPG: 0–5 MHz, 0–30 pulses, 4Ω output impedance, programmable duty cycle
- SDHS ADC: 8 Msps (hardware ceiling), 12-bit, sigma-delta, single-ended, DMA-connected to FRAM
- PGA: −6.5 to +30.8 dB, 46 steps, ~0.81 dB/step, **fixed during acquisition** (no TGC)
- FRAM: 64 KB, SRAM-speed access, non-volatile — enables LPM4.5 complete shutdown with state retention
- DMA: ≥6 channels, hardware-triggered from SDHS; CPU not involved during acquisition
- Power modes: LPM4+RTC = 450 nA; LPM4.5 = 35 nA (250 µs wake, too slow for 50 Hz PRF)
- Supply: 1.8–3.6V (single-cell LiPo direct, no regulator needed)
- Datasheet: SLASEC4

---

## Task Templates

### Adding a new chip deep dive

> Write `{ChipName}.md` — a deep dive on the {ChipName} IC. Cover: (1) overview and target application, (2) each functional block with spec table, (3) what each block enables and constrains, (4) power consumption, (5) supply voltages and package, (6) known uses, (7) constraints and limitations. Use the pattern in MSP430FR5043.md or TUSS4470.md as a template. Source all figures from the datasheet; note the datasheet part number at the top. Flag any specs not published.

### Adding a comparison document

> Write `{X}vs{Y}.md` comparing {X} and {Y}. Frame the comparison asymmetrically if the devices are not symmetric (e.g., component vs integrated system). Cover: (1) what each device is, (2) side-by-side spec table, (3) integration level diagram, (4) the most architecturally significant difference (output type, frequency range, etc.), (5) gain architecture, (6) power profile with duty-cycle arithmetic, (7) development ecosystem, (8) cost at system BOM level (not just IC price), (9) what each is optimised for, (10) which to choose and when. Use TUSvsMSP.md as a template.

### Continuing a phase

> Continue the comparison research in kelu124/wulrick. Read todo.md for current status. Read the most recent phase file. Then: [specific task]. Push when done and report the key findings.

### Scraping a GitHub repo

> Scrape {owner}/{repo}/discussions and {owner}/{repo}/issues. For discussions: capture the title, date, author, comment count, and a detailed summary of the thread content. For issues: group by theme (hardware, firmware, documentation, feature requests). Then write {output-file}.md with structured summaries per thread and a synthesized observations section at the end. Push to kelu124/wulrick.

---

## Output Format Conventions

- **Tables**: Use markdown GFM tables with aligned columns. Parameter | Value | Notes columns for spec tables.
- **Spec tables**: Include source type notation (D, M, C, E) in the Notes column for key figures.
- **Gaps**: When a spec is missing or unconfirmed, say so explicitly: "Not published", "Provisionally identified as X; not confirmed from BOM", "Requires bench measurement".
- **Tone**: Technical, precise, neutral. Avoid superlatives. "WULPUS achieves 22 mW" not "WULPUS impressively achieves 22 mW".
- **Code**: Use markdown fenced code blocks with language specifiers. JavaScript push snippets go in `javascript` blocks.
- **File headers**: Each file starts with a `# Title` H1 followed by a brief italic context line explaining what the file contains and where the data came from.

---

## Known Issues and Workarounds

**GitHub SHA conflict on parallel push:** If two existing files are fetched simultaneously and then both pushed, the second push fails with HTTP 409 because both GETs returned the same (now-stale) SHA. Fix: push files sequentially with `await` between each call.

**Altium BOM not readable:** WULPUS PCB files are in Altium binary format. The HV mux part number cannot be confirmed directly. Workaround: identify from schematic functional analysis and cross-reference with known HV mux families (Supertex HV2707, Microchip HV20820, Maxim MAX14866).

**MSP430FR5043 ENOB not published:** TI does not publish an ENOB figure for the USS_A SDHS ADC. This is the single most important uncharacterized spec for dynamic range assessment. Cannot be filled without bench measurement or TI application engineering contact.

**WULPUS-Pro not yet public:** Referenced in multiple papers and the README as the successor platform (16 ch, +30V, programmable TGC, CMUT bias, 100 kHz–10 MHz). Monitor pulp-bio GitHub for new repositories. As of 2026-06, no public repository exists.
