# Ultrasound Array Sourcing Guide

*Compatible linear arrays and single-element transducers for pic0rick and WULPUS. All information compiled from published datasheets, distributor listings, and community reports. Prices are approximate and subject to change.*

---

## 1. Summary: Frequency Ranges and Device Compatibility

| Transducer frequency | pic0rick (ADC10065, 65 Msps) | WULPUS (SDHS, 8 Msps) |
|----------------------|-------------------------------|------------------------|
| 40 kHz – 200 kHz | ✓ (oversampled) | ✓ (within Nyquist) |
| 200 kHz – 1 MHz | ✓ | ✓ (1 MHz = Nyquist limit) |
| 1 MHz – 2 MHz | ✓ | ✓ (primary WULPUS range) |
| 2 MHz – 4 MHz | ✓ | △ (edge of Nyquist; aliasing likely above 3 MHz) |
| 4 MHz – 10 MHz | ✓ | ✗ (above Nyquist for 8 Msps ADC) |
| 10 MHz – 30 MHz | ✓ (65 Msps supports up to ~32 MHz) | ✗ |

The fundamental compatibility boundary: **pic0rick works with virtually any transducer up to ~32 MHz; WULPUS is practically limited to 1–3 MHz transducers.**

---

## 2. Single-Element Transducers

### 2.1 Olympus (formerly Panametrics) V-Series — NDT Immersion / Contact

The V-series is the most commonly referenced family in the pic0rick community and wider open-source ultrasound context. These are unfocused or weakly focused single-element transducers, originally for industrial NDT, but widely used in research due to availability, known specs, and LEMO 00 connector standard.

| Model | Frequency | Element size | Focal length | Notes |
|-------|-----------|--------------|--------------|-------|
| V106 | 2.25 MHz | 0.375″ (9.5 mm) | Unfocused | Low-frequency baseline; WULPUS-compatible |
| V110 | 5 MHz | 0.25″ (6.3 mm) | Unfocused | Common in pic0rick experiments |
| V111 | 10 MHz | 0.25″ (6.3 mm) | Unfocused | Standard NDT reference frequency |
| V112 | 10 MHz | 0.5″ (12.7 mm) | Unfocused | Larger element, more sensitive |
| V125 | 2.25 MHz | 0.5″ (12.7 mm) | Unfocused | Liquid immersion; WULPUS-compatible |
| V306 | 2.25 MHz | 0.25″ (6.3 mm) | Focused | Good near-field resolution at low freq |

**Connector:** LEMO 00 (standard for most V-series). Adapts to SMA via LEMO-to-SMA cable.

**Approximate prices (new, qty 1):** $200–$600 per transducer depending on model. Used / surplus prices on eBay or from US Ultrasonics: $30–$150.

**Compatibility:**
- pic0rick: All V-series; direct connection via LEMO-to-SMA cable
- WULPUS: V106, V125, V306 (2.25 MHz); V110 (5 MHz) will alias significantly

**Community use documented:** HP2121, Kretz AW145BA, ATL3 (repurposed medical), and generic 5 MHz contact transducers have been used with pic0rick (kelu124 field reports). V-series are the recommended starting point for controlled experiments.

---

### 2.2 Olympus (Videoscan) Contact Transducers

Similar to V-series but designed for direct contact with solid materials (no couplant medium required for dry coupling). Frequency range 0.5–25 MHz. Same LEMO 00 connector standard.

| Model | Frequency | Use case |
|-------|-----------|----------|
| V110-RM | 5 MHz | Contact NDT; pic0rick compatible |
| V116 | 7.5 MHz | Higher frequency NDT |
| V122 | 10 MHz | Fine-grain contact NDT |

Prices similar to immersion V-series. Less commonly found surplus.

---

### 2.3 Generic NDT Contact Transducers (Chinese Manufacturers)

Available from Alibaba, AliExpress, and NDT supply distributors. Specifications vary significantly; some are well-characterized, many are not.

| Typical frequency | Connector | Price range | Notes |
|-------------------|-----------|-------------|-------|
| 1 MHz | BNC | $20–50 | WULPUS-range; quality varies |
| 2.25 MHz | BNC | $20–50 | Most commonly used with WULPUS |
| 5 MHz | BNC | $15–40 | pic0rick range; acceptable for research |
| 10 MHz | BNC | $20–60 | ADC10065 capable; some have poor bandwidth |

**Adapters:** BNC to SMA adapters are inexpensive ($2–5) and reliable for prototype work. For the WULPUS board-to-wire connector, coaxial cable with bare-wire termination is used directly.

**Recommendation:** Generic transducers are appropriate for initial bring-up and algorithm development. For published research, use characterized transducers (Olympus or equivalent) with published center frequency, bandwidth, and sensitivity specs.

---

### 2.4 Repurposed Medical Transducers

The pic0rick community has documented use of surplus single-element probes from legacy medical equipment:

| Identified model | Original equipment | Frequency (est.) | Notes |
|------------------|--------------------|------------------|-------|
| HP 2121 | HP/Philips echocardiography | ~2–3 MHz | Used by kelu124; repurposed A-mode probe |
| Kretz AW145BA | Kretz/GE ultrasound | ~3–5 MHz | Documented in kelu124 field reports |
| ATL3 | ATL imaging system | ~3 MHz | Repurposed; WULPUS-range compatible |

These are not available new. Found via eBay, Craigslist, medical surplus dealers. Prices: $10–80. **Frequency and specs are partially or fully undocumented** — require bench characterization before use in controlled experiments. Connector types vary (proprietary in many cases; adapting requires disassembly or custom cable).

---

## 3. Linear Array Transducers

### 3.1 Vermon Research Arrays (France)

Vermon is a French manufacturer of research-grade ultrasound transducer arrays. They supply linear, phased, and annular arrays to academic and industrial research customers.

| Product line | Typical elements | Frequency range | Pitch | Application |
|--------------|-----------------|-----------------|-------|-------------|
| Research Linear | 32–128 | 1–10 MHz | 0.3–0.6 mm | Vascular, cardiac, general research |
| Compact Linear | 32 | 2–5 MHz | 0.3 mm | Wrist-worn / wearable research |
| Annular arrays | 4–8 | 2–10 MHz | Custom | Depth scanning without beam steering |

**Connector:** Proprietary FPC (flexible printed circuit) connectors. 32-element arrays typically use 34-pin 0.8 mm pitch Micro-Coax or custom FPC. **Not directly compatible with pic0rick or WULPUS without a custom adapter PCB.**

**Approximate prices:** Research arrays are priced on a per-project basis. Based on academic publications and procurement discussions:
- 32-element arrays: ~€800–2,500 depending on configuration and quantity
- Custom arrays: €2,000–10,000+

**Compatibility:**
- pic0rick: Electrically feasible with custom FPC-to-SMA adapter board + MAX14866 PMOD cascade. Frequency compatibility depends on specific array frequency (2–10 MHz pic0rick range). **No commercially available adapter.**
- WULPUS: Issue #16 specifically requests a Vermon 32-ch adapter. Frequency compatibility only for Vermon arrays ≤3 MHz. Requires BLE → USB migration for useful frame rates. See `w_discussions.md` deep analysis.

**How to obtain:** Direct contact with Vermon (vermon.com). No catalog online ordering. Academic pricing available with institutional affiliation. Lead times typically 4–12 weeks.

---

### 3.2 Imasonic Research Arrays (France)

Imasonic is another French manufacturer known for high-quality NDT and research arrays. Less commonly referenced in open-source ultrasound literature but available through similar channels.

| Product | Elements | Frequency | Notes |
|---------|----------|-----------|-------|
| Linear arrays | 32–256 | 1–20 MHz | NDT and biomedical; custom FPC connectors |
| Annular | 4–16 | 2–20 MHz | Multi-zone focusing |

**Prices:** Similar to Vermon range; €500–3,000+ depending on configuration.

**Compatibility:** Same constraints as Vermon — requires custom adapter for pic0rick or WULPUS. Frequency compatibility with WULPUS only for low-frequency variants.

---

### 3.3 Olympus / Evident NDT Linear Arrays

Olympus (now Evident) produces phased array and linear array probes for industrial NDT. These are designed for phased array ultrasonic testing (PAUT) instruments but can be driven by research front-ends with appropriate adapters.

| Model | Elements | Frequency | Notes |
|-------|----------|-----------|-------|
| 5L64-A12 | 64 | 5 MHz | 64-element; standard NDT PAUT probe |
| 10L32-A10 | 32 | 10 MHz | 32-element; common research starting point |
| 2.25L16-A10 | 16 | 2.25 MHz | WULPUS-compatible frequency |

**Connector:** LEMO 1B (for single-element) or proprietary Olympus phased array connector (PA connectors, 26-pin or 26/2-pin). PA probes require adapter — these are available from Olympus and from third-party suppliers (~$200–500 for adapter cables).

**Prices (new):** $800–3,000 per probe. Used: $100–600 on eBay.

**Compatibility:**
- pic0rick: The 5 MHz or 10 MHz arrays are well within pic0rick's ADC range. The 64-element 5L64 would require 8 MAX14866 PMODs cascaded (64 ch). The 32-element 10L32 requires 4 PMODs.
- WULPUS: The 2.25L16 (16-element, 2.25 MHz) is within WULPUS's frequency range. 16 channels would require two cascaded 8-ch mux stages — requires PCB modification.

---

### 3.4 Ultrasonics International / US Ultrasonics (Surplus Transducers)

US-based surplus dealer. Stocks large volumes of refurbished, tested, and surplus medical and NDT transducers.

**URL:** us-ultrasonics.com (verify currency — note this was current as of 2026)

**Typical inventory:**
- Olympus V-series (all frequencies): $40–150 each
- Single-element contact probes (various manufacturers): $15–60
- Occasional multi-element probes from decommissioned medical systems

**Recommended for:** Initial transducer acquisition, algorithm development, budget projects. Not recommended for applications requiring documented frequency response or published sensitivity curves.

---

## 4. Phased Array Probes (Biomedical)

### 4.1 Repurposed Medical Phased Array Probes

Phased array probes from legacy ultrasound systems (Philips, GE, Siemens, ATL) occasionally appear on the surplus market. These are designed for cardiac and vascular imaging at 1–8 MHz.

| Probe type | Typical elements | Frequency | Connector |
|------------|-----------------|-----------|-----------|
| Cardiac (S-series) | 64–128 | 2–4 MHz | Proprietary (system-specific) |
| Vascular (linear) | 48–128 | 5–12 MHz | Proprietary (system-specific) |
| Endoscopic/TEE | 32–64 | 3–7 MHz | Proprietary |

**Key challenge:** Medical probe connectors are proprietary and machine-specific. A GE probe will not physically mate with a Philips system. Breaking out individual element signals requires either a known pinout (available for some common probes via ultrasound engineering community resources) or a custom breakout PCB.

**Known breakout projects:**
- The open-ultrasound community (atl3-kelu124 fork and similar) has documented pinouts for some ATL/Philips probes
- Some Alpinion probes are used with open systems due to documented connector pinouts

**Prices:** $50–300 for the probe body. Breakout cable/adapter: $50–200 or DIY.

**Compatibility:** Potentially compatible with pic0rick (for 2–5 MHz probes) or WULPUS (for 1–3 MHz probes) once element signals are broken out. Frame rates limited by channel count and mux capacity.

---

## 5. Wearable / Flexible Transducers

### 5.1 CMUTs (Capacitive Micromachined Ultrasonic Transducers)

CMUTs are microfabricated arrays with potential for wearable integration (thin, flexible). Relevant to WULPUS-Pro which explicitly adds CMUT bias support (±30V).

**Key suppliers:** Kolo Medical (now Exo Imaging), CMUT arrays from Fraunhofer IBMT, academic custom fabrication.

**Frequency:** 1–15 MHz depending on design.

**Limitation:** Not commercially available as off-the-shelf components. Academic collaborations or custom procurement only. Prices typically $1,000–10,000+ per die.

**Compatibility:** WULPUS-Pro (with ±30V CMUT bias) is the intended target. Standard WULPUS cannot drive CMUT bias voltage. pic0rick's ±24V could potentially drive CMUT if bias requirement is met separately.

---

### 5.2 Piezo Film (PVDF) Single-Element Transducers

PVDF (polyvinylidene fluoride) transducers are flexible, lightweight, and available from several manufacturers. Used in wearable ultrasound prototypes.

| Manufacturer | Frequency | Notes |
|-------------|-----------|-------|
| Measurement Specialties / TE Connectivity | 1–15 MHz | Round and rectangular elements; standard industrial offerings |
| Precision Acoustics | 0.5–50 MHz | Research-grade PVDF; calibrated |
| Piezotech (Arkema) | Varies | PVDF-TrFE film for research |

**Prices:** $20–200 per element depending on frequency and form factor.

**Compatibility:**
- WULPUS: PVDF elements at 1–2 MHz are compatible. PVDF has lower sensitivity than PZT; gain should be set to maximum (40.8 dB system gain).
- pic0rick: Any PVDF element up to ~30 MHz is within ADC range. TGC compensates for PVDF's lower sensitivity.

**Connector:** Typically bare coaxial cable or BNC. Adapts straightforwardly to either platform.

---

## 6. Comparison Table — Quick Reference

| Source | Type | Frequency | Elements | Price range | pic0rick compat. | WULPUS compat. | Connector |
|--------|-------|-----------|----------|-------------|-----------------|-----------------|-----------|
| Olympus V106 | Single element | 2.25 MHz | 1 | $50–300 | ✓ | ✓ | LEMO 00 |
| Olympus V110 | Single element | 5 MHz | 1 | $50–300 | ✓ | ✗ | LEMO 00 |
| Olympus V111 | Single element | 10 MHz | 1 | $80–400 | ✓ | ✗ | LEMO 00 |
| Generic NDT 2.25 MHz | Single element | 2.25 MHz | 1 | $20–50 | ✓ | ✓ | BNC |
| Generic NDT 5 MHz | Single element | 5 MHz | 1 | $15–40 | ✓ | ✗ | BNC |
| Vermon 32-ch | Linear array | 2–8 MHz | 32 | €800–2,500 | ✓ (custom adapter) | △ (custom adapter + BLE→USB) | Proprietary FPC |
| Olympus 2.25L16 | Phased array | 2.25 MHz | 16 | $200–800 | ✓ | ✓ (custom HV PCB) | PA connector |
| Olympus 10L32 | Linear array | 10 MHz | 32 | $400–2,000 | ✓ (4× PMOD) | ✗ | PA connector |
| Medical surplus (ATL3, HP2121) | Single element | ~2–5 MHz | 1 | $10–80 | ✓ | △ (2 MHz only) | Proprietary |
| PVDF 1–2 MHz | Single element | 1–2 MHz | 1 | $20–100 | ✓ | ✓ | Bare coax |

---

## 7. Recommended Starting Points

### For pic0rick users starting from scratch:
1. **Olympus V110 (5 MHz)** — well-characterized, surplus available, LEMO 00 standard, directly within pic0rick's target range. Order from US Ultrasonics or eBay. Add LEMO-to-SMA adapter cable.
2. **Generic 5 MHz contact probe (BNC)** — half the price, adequate for algorithm development and initial imaging. Add BNC-to-SMA adapter.

### For WULPUS users starting from scratch:
1. **Generic 2.25 MHz contact probe (BNC)** — lowest cost, directly within WULPUS's 1–2 MHz sweet spot. Connect via bare coaxial to HV PCB board-to-wire connector.
2. **Olympus V106 (2.25 MHz)** — characterized and documented; appropriate for research requiring published transducer specs.

### For users wanting to explore array imaging:
1. **Contact Vermon** for a 32-element low-frequency array (specify 1–3 MHz for WULPUS compatibility, or 3–5 MHz for pic0rick). Budget €1,000–2,500 and several weeks lead time. Expect to design a custom adapter PCB.
2. **Olympus 2.25L16 (surplus)** — 16 elements at 2.25 MHz, occasional eBay availability. WULPUS-compatible frequency; requires two cascaded HV mux stages. PA connector breakout is the primary mechanical challenge.

---

## 8. Notes and Caveats

**Coupling medium:** Contact probes require gel or water immersion for efficient acoustic coupling. Standard medical ultrasound gel (available from medical suppliers, ~$5–15 for 250 ml) is appropriate. Water immersion setups (aquarium tanks, water tanks) are commonly used in NDT and open-source ultrasound labs for repeatable characterization.

**Cable impedance:** Ultrasound transducer cables are typically 50 Ω coaxial. Short cables (<1 m) minimize reflections. Both pic0rick and WULPUS are designed for 50 Ω transducer impedance.

**Frequency matching:** Transducer bandwidth is typically ±50% of center frequency (e.g., a 5 MHz transducer responds usefully from 2.5–7.5 MHz). The transmit excitation frequency should be matched to the transducer center frequency for maximum efficiency. Both pic0rick and WULPUS allow frequency configuration.

**Price uncertainty:** Ultrasound transducer pricing is volatile and highly quantity-dependent. Research prices are typically higher than production volume prices. All prices in this document are estimates based on publicly available information as of 2026.
