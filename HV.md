# HV Sources — OtherSystems Survey

*Documents the high-voltage generation approach for each system in `OtherSystems/`. Compiled from available documentation; gaps are noted explicitly where schematics are not public.*

---

## EchoLite

**HV voltage:** Not disclosed.  
**Topology:** Not disclosed.  
**Key components:** None named.  
**Gating:** Not documented.

EchoLite's paper (Leitner & Giordano, IEEE CEEUS Warsaw 2026) is not yet publicly indexed. Total system power is 33 mW — comparable to WULPUS (22 mW) — suggesting a similar low-voltage approach, but this is speculative. No schematics or block diagrams are publicly available.

---

## PuLsE

**HV voltage:** Not stated. Shallow target (~5–10 mm, radial artery at wrist) implies low transmit voltage — estimated < 15 V.  
**Topology:** "Custom low-power single-cycle pulser" — specific design not disclosed.  
**Key components:** None named.  
**Gating:** Pulsed during transmit only. Single element, no HV mux.

PuLsE achieves 5.8 mW total system power — the lowest of any compared system. The PuLsE paper explicitly identifies WULPUS's three overhead contributors: the 8-channel HV mux, the +15 V supply (overkill for wrist depth), and the MSP430 minimum config. PuLsE eliminates the mux and likely runs a lower supply voltage. It uses analog envelope detection rather than raw RF capture, further reducing HV supply requirements.

---

## TinyProbe

**HV voltage:** 64 V peak-to-peak transmit.  
**Topology:** Custom HV driver, FPGA-synchronised per channel. Specific topology not disclosed; implied custom ASIC per element (ETH Zurich group).  
**Key components:** None named — likely a proprietary ETH ASIC driver.  
**Gating:** 16 programmable transmit delay profiles across 32 simultaneous channels for transmit beamforming. High-PRF Doppler mode requires rapid repetitive gating.

64 Vpp is ~4× WULPUS's +15 V. Acoustic pressure scales ∝ V², so TinyProbe delivers ~16× more energy per pulse — enabling 15 cm tissue penetration vs WULPUS's 4 cm. Simultaneously switching 32 channels at 64 V requires a robust HV supply design; the documentation explicitly flags safety and HV layout as a design constraint for skin-contact use. No commercial HV IC is identified.

---

## USoP

**HV voltage:** Not stated. Estimated 10–30 V given 3–5 MHz transducer and shallow-to-mid tissue target.  
**Topology:** Embedded in flexible patch electronics. Not disclosed.  
**Key components:** None named.  
**Gating:** Episodic pulsed acquisition. Single element, no mux.

USoP (Nature Biotechnology) is a closed design — hardware details beyond the paper's supplementary materials are not publicly available. Total system power is 614 mW continuous, 28× WULPUS. The high figure likely reflects a larger battery and display overhead rather than purely HV draw. The flexible patch form factor (serpentine Cu/PI interconnects on silicone) constrains topology: conventional inductors for a boost converter are impractical in a conformal wearable.

---

## WULPUS

**HV voltage:** +15 V (unipolar).  
**Topology:** Boost converter stepping up from the LiPo battery rail (~3.7 V nominal). The specific boost IC is not named in available documentation.  
**Key components:**
- **HV2707** (Microchip) — 8-channel T/R mux, ±80 V rated, SPI-controlled, SOIC-16. Time-multiplexed switching across 8 transducer elements.
- **MD0101** (Microchip) — active T/R switch, ~100 ns RX recovery after TX pulse.
- **MCP1416** (Microchip) — MOSFET gate driver for the unipolar +15 V pulser switch.
- Unnamed boost converter IC.

**Gating:** Pulsed at 50 Hz PRF. Analog amplifier (OPA836) power-gated between acquisitions. Blanking gate active during TX pulse to prevent ADC saturation.

WULPUS is the only fully open-source HV implementation in OtherSystems (Altium schematics and Gerbers public). The unipolar +15 V architecture minimises power: a single high-side switch per element, boost converter sized for short burst duty cycles. The 8-channel HV2707 mux accounts for a meaningful fraction of the 22 mW budget — PuLsE identifies mux elimination as a key lever for a single-element derivative. Full design: `OtherSystems/WULPUS.md`.

---

## Weik 2026 Review

Survey paper (IEEE Reviews in Biomedical Engineering 2026) — introduces no new HV hardware. No specific HV topology, voltage, or component is documented. Relevant observation: the review notes that HV supply design for wearable ultrasound remains "ad hoc" across the field, and that application depth requirements (blood pressure ~2 cm vs cardiac ~15 cm) drive a wide range of transmit voltages with no standardisation across research groups.

---

## pic0rick (ports analysis)

**HV voltage:** ±24 V (bipolar three-level).  
**Topology:** Isolated DC-DC boost. USB 5 V input → ±24 V output via an off-the-shelf isolated converter on a separate PMOD board.  
**Key components:**
- **R2D-0524_P** (Recom) — isolated DC-DC, 5 V in → ±24 V out, 2 W rated. Core HV supply.
- **MD1213K6-G** (Microchip) — HV switch driver for the pulser.
- **TC6320TG-G** — gate driver.
- **MD0100N8-G** (Microchip) — passive T/R limiter, ±2.5 V clamping, ~5 µs RX recovery.
- **MD0101** (Microchip) — optional active T/R switch upgrade, ~100 ns RX recovery.
- **MAX14866** (Maxim/ADI) — optional 8-channel HV mux, ±100 V rated (equivalent to WULPUS's HV2707).

**Gating:** PWM-based pulser controlled by RP2040 PIO. Dedicated blanking gate (PWM_G2) disables RX path during TX pulse. Bipolar ±24 V enables three-level waveforms: +24 V, 0 V, −24 V per half-cycle.

**Notable design choices:**
- **Isolation rationale:** USB 5 V → ±24 V isolated to avoid a ground loop between the HV switching rail and the ADC reference. A non-isolated boost would couple switching noise into the receive chain.
- **PMOD separation:** Pulser on a separate PMOD board physically distances ±24 V transients from the RX signal path. Moving it on-board (Transplant 3B proposal) requires ground-plane partitioning.
- **±24 V vs WULPUS +15 V:** Bipolar drive allows coherent RF waveforms and higher SNR at depth. Unipolar +15 V minimises power but limits biphasic acoustic drive.
- **MD0100 vs MD0101:** Passive limiter gives a 5 µs dead zone (~4 mm non-imaging range in water). MD0101 upgrade cuts this to ~100 ns (~0.75 mm) at the cost of one GPIO and ~$1 BOM increase.
- **2 W supply ceiling:** Adequate for single-element duty-cycled use. Scaling to ≥16 simultaneous TX channels requires a higher-rated HV supply.

Full design: `OtherSystems/pic0rick_ports.md`.

---

## Summary

| System | HV voltage | Topology | Key ICs | Channels | Notable |
|--------|-----------|----------|---------|----------|---------|
| EchoLite | Unknown | Unknown | — | 1 | 33 mW total; HV design not public |
| PuLsE | < 15 V (est.) | Custom low-power pulser | — | 1 | 5.8 mW; lowest power; no mux |
| TinyProbe | 64 Vpp | Custom FPGA-driven HV driver | Custom ASIC (ETH) | 32 simultaneous | 15 cm depth; skin-contact safety noted |
| USoP | ~10–30 V (est.) | Embedded in flex patch (unknown) | — | 1 | 614 mW; closed design; conformal patch |
| WULPUS | +15 V | Boost converter (LiPo → +15 V) | HV2707, MD0101, MCP1416 | 8 (time-mux) | Open-source; 22 mW; 4 cm depth |
| Weik 2026 | N/A (survey) | N/A | — | N/A | Notes HV design is "ad hoc" across field |
| pic0rick | ±24 V | Isolated DC-DC (R2D-0524_P) | MD0100/MD0101, MD1213, MAX14866 opt. | 1 + opt. 8 | USB-powered; bipolar; isolated; open design |
