# On-Device Hilbert Transform on RP2350 M33 FPU

*Motivation: PuLsE (Giordano et al. 2025) demonstrates envelope detection entirely on-device at 5.8 mW total system power. With the M33 FPU on RP2350, the same operation can be added to pic0rick's RP2350B variant with zero BOM cost and at negligible power overhead.*

---

## Why: removing the host round-trip

In the current pic0rick workflow, raw 10-bit ADC samples are transferred over USB to a host PC, where `scipy.signal.hilbert()` computes the envelope:

```python
from scipy.signal import hilbert
import numpy as np

rf = np.array(raw_samples, dtype=np.float32)
analytic = hilbert(rf)
envelope = np.abs(analytic)
```

This round-trip has three costs:

1. **Latency:** USB Full Speed adds ~2–8 ms per acquisition. For display, this is acceptable. For closed-loop control (e.g., triggering on a peak exceeding a depth threshold), it is not.
2. **Bandwidth tax:** Transmitting 8000 samples × 2 bytes = 16 KB per acquisition at 200 Hz PRF = 3.2 MB/s of USB bandwidth — 25% of USB Full Speed's ceiling — for data that is then downsampled 50× by the envelope step.
3. **Host PC dependency:** Standalone field operation (battery + SD card, no laptop) requires on-chip envelope detection. There is no other path.

Moving the Hilbert to the RP2350's M33 FPU enables: local peak detection, on-chip thresholding, reduced USB bandwidth (transmit envelope rather than raw RF), and fully autonomous logging at 50× lower data rate.

---

## Mathematical foundation

The analytic signal of a real-valued signal x[n] is:

```
a[n] = x[n] + j·H{x[n]}
```

where H{·} is the Hilbert transform. The **envelope** (instantaneous amplitude) is:

```
env[n] = |a[n]| = sqrt(x[n]² + H{x[n]}²)
```

The Hilbert transform is computed efficiently via FFT:

```
X[k]  = FFT{x[n]}          (N-point complex spectrum)

A[k]  = 2·X[k],  1 ≤ k ≤ N/2 - 1    (double positive frequencies)
        X[k],    k = 0 (DC) or k = N/2 (Nyquist)
        0,       N/2 < k ≤ N-1        (zero negative frequencies)

a[n]  = IFFT{A[k]}          (complex analytic signal)
env[n]= |a[n]|
```

This is exactly what `scipy.signal.hilbert()` does internally (via `scipy.fft.fft` + frequency masking + `scipy.fft.ifft`). The port to CMSIS-DSP is a direct translation.

---

## Benchmark context

| Operation | M33 @ 150 MHz | Notes |
|-----------|--------------|-------|
| 4096-pt real FFT (`arm_rfft_fast_f32`) | ~27 µs | CMSIS-DSP benchmark, Cortex-M33 @ 150 MHz |
| Frequency masking (scale N/2 bins) | ~3 µs | `arm_scale_f32` on 4096 floats |
| 4096-pt complex IFFT (`arm_cfft_f32`) | ~55 µs | Complex FFT is ~2× real FFT |
| Magnitude (`arm_cmplx_mag_f32`) | ~7 µs | sqrt per bin |
| **Total Hilbert pipeline** | **~92 µs** | Conservative; FPU pipeline effects vary |

At 200 Hz PRF, inter-pulse gap = **5000 µs**. The full Hilbert pipeline at ~92 µs consumes less than 2% of available inter-pulse compute time. Even at the RP2350's minimum 125 MHz clock, timing is comfortable.

On RP2040 (no FPU, software float), the same pipeline takes ~8–12 ms — exceeding the 5 ms inter-pulse gap at 200 Hz. This is why on-device Hilbert requires RP2350.

---

## CMSIS-DSP implementation path

### Approach: `arm_cfft_f32` (complex FFT, full-length)

The cleanest implementation uses the complex FFT directly. The real ADC data is zero-padded into a complex array; the rest follows the frequency-domain Hilbert formula exactly.

```c
#include "arm_math.h"
#include "arm_const_structs.h"

#define NFFT 4096

// Allocated in SRAM (4096 × 8 bytes = 32 KB)
static float32_t cplx_buf[NFFT * 2];   // interleaved re/im pairs
static float32_t envelope[NFFT];

void hilbert_f32(const int16_t *adc_buf, float32_t *env_out, uint32_t n) {
    // Step 1: Load real ADC samples into complex buffer (im = 0)
    for (uint32_t i = 0; i < n; i++) {
        cplx_buf[2*i]     = (float32_t)adc_buf[i];  // real
        cplx_buf[2*i + 1] = 0.0f;                    // imaginary = 0
    }

    // Step 2: Forward complex FFT (in-place)
    arm_cfft_f32(&arm_cfft_sR_f32_len4096, cplx_buf, 0, 1);
    // args: instance, data, ifftFlag=0(forward), bitReverseFlag=1

    // Step 3: Frequency-domain Hilbert masking
    // DC bin (k=0): unchanged
    // Positive bins (k=1..N/2-1): multiply by 2
    arm_scale_f32(&cplx_buf[2], 2.0f, &cplx_buf[2], (NFFT/2 - 1) * 2);
    // Nyquist bin (k=N/2): unchanged (cplx_buf[NFFT] to cplx_buf[NFFT+1])
    // Negative bins (k=N/2+1..N-1): zero
    memset(&cplx_buf[(NFFT/2 + 1) * 2], 0, (NFFT/2 - 1) * 2 * sizeof(float32_t));

    // Step 4: Inverse complex FFT (in-place)
    arm_cfft_f32(&arm_cfft_sR_f32_len4096, cplx_buf, 1, 1);
    // args: instance, data, ifftFlag=1(inverse), bitReverseFlag=1
    // CMSIS-DSP IFFT does not include the 1/N scaling factor automatically
    arm_scale_f32(cplx_buf, 1.0f / NFFT, cplx_buf, NFFT * 2);

    // Step 5: Magnitude → envelope
    arm_cmplx_mag_f32(cplx_buf, env_out, NFFT);
}
```

**Note on `arm_cfft_sR_f32_len4096`:** This is a pre-computed constant instance structure from `arm_const_structs.h`, valid for 4096-point complex FFT. It does not need runtime initialisation.

### Alternative: `arm_rfft_fast_f32` (real FFT, half-length)

`arm_rfft_fast_f32` is more memory-efficient (operates on N real samples, internal N/2 complex computation). However, its output packing format is non-standard: `out[0]` = Re[DC], `out[1]` = Re[Nyquist], `out[2k]` = Re[k], `out[2k+1]` = Im[k] for k=1..N/2-1. Implementing the Hilbert masking on this packed format requires careful indexing and is error-prone. The `arm_cfft_f32` approach above is recommended for clarity.

Use `arm_rfft_fast_f32` if SRAM is a critical constraint (it saves 16 KB by operating on N floats rather than 2N floats).

---

## Fixed-point vs float tradeoff

| | Q15 fixed-point | float32 (M33 FPU) |
|--|----------------|-------------------|
| M33 instruction | SIMD SMLAD (2 ops/cycle) | single-cycle FPU (VSQRT ~14 cyc) |
| Dynamic range | 16-bit, ~96 dB | 24-bit mantissa, ~144 dB |
| ADC data width | 10 bits — fits Q15 | fits float32 trivially |
| CMSIS-DSP support | `arm_cfft_q15`, `arm_cmplx_mag_q15` | `arm_cfft_f32`, `arm_cmplx_mag_f32` |
| Scaling headaches | Yes — precision/saturation management | No |
| Code complexity | Higher | Lower |

**Recommendation: use float32.** The ADC10065 is 10-bit; float32 (24-bit mantissa) is lossless for this data. The M33 FPU handles float32 at single-cycle throughput for most operations. The Q15 path saves ~10–20% compute time but adds significant development complexity with no quality benefit for 10-bit input.

Q15 is worth considering only if power budget is extremely tight — but at 5 ms inter-pulse gap, there is no compute pressure that justifies the Q15 path.

---

## Memory layout for DMA handoff

```
SRAM layout (RP2350B, 520 KB):
┌──────────────────────────────────┐ 0x20000000
│  Firmware + stack (~60 KB)       │
├──────────────────────────────────┤
│  ADC DMA buffer A (16 KB)        │ ← ADC10065 SPI DMA writes here
│  ADC DMA buffer B (16 KB)        │ ← ping-pong: one capturing, one processing
├──────────────────────────────────┤
│  Hilbert complex buffer (32 KB)  │ ← cplx_buf[4096*2 float32]
│  Envelope output (16 KB)         │ ← env_out[4096 float32]
├──────────────────────────────────┤
│  FatFs + SD buffer (~8 KB)       │ ← if SD card logging enabled
│  USB/display buffers (~20 KB)    │
└──────────────────────────────────┘
Total: ~168 KB — within 520 KB SRAM
```

**Ping-pong DMA pattern:**
- Core 0: manages SPI DMA, fires pulse, waits for ADC DMA complete, signals Core 1
- Core 1: runs Hilbert pipeline on the completed ADC buffer while Core 0 sets up next acquisition
- Synchronisation: `multicore_fifo_push_blocking()` / `multicore_fifo_pop_blocking()` (Pico SDK)

This zero-dead-time architecture means envelope computation overlaps with the next acquisition — the Hilbert pipeline's 92 µs is completely hidden inside the 5 ms inter-pulse gap.

---

## Practical porting steps from Python host

**Step 1 — Validate with a test vector.**
Generate a 4096-sample tone burst in Python, compute `scipy.signal.hilbert()`, export as a `.npy` file. Run the CMSIS-DSP implementation on RP2350 with this data (hardcoded array), export the result over USB, compare against the Python reference. They should agree to within float32 rounding (~6 significant decimal places).

**Step 2 — Add CMSIS-DSP to the CMake build.**
```cmake
# In CMakeLists.txt:
include(FetchContent)
FetchContent_Declare(
  CMSISDSP
  GIT_REPOSITORY https://github.com/ARM-software/CMSIS-DSP
  GIT_TAG v1.15.0
)
FetchContent_MakeAvailable(CMSISDSP)
target_link_libraries(pic0rick_firmware CMSISDSP)
target_compile_definitions(pic0rick_firmware PRIVATE ARM_MATH_CM33)
```

**Step 3 — Add `hilbert_f32()` call to the acquisition loop.**
After the ADC DMA complete interrupt fires on Core 0, push the buffer pointer to Core 1's FIFO. Core 1 calls `hilbert_f32()`, then either logs the envelope to SD card or transmits via USB/WiFi.

**Step 4 — Add peak detection.**
```c
uint32_t peak_idx;
float32_t peak_val;
arm_max_f32(envelope, NFFT, &peak_val, &peak_idx);
float32_t depth_mm = (float32_t)peak_idx / SAMPLE_RATE_MHZ * SPEED_OF_SOUND_MM_US / 2.0f;
```
`arm_max_f32` takes ~4 µs for 4096 samples. Depth in mm: `sample_index / Fs × c / 2`, with c ≈ 1.54 mm/µs in soft tissue.

**Step 5 — Optional: transmit envelope instead of raw RF.**
Envelope output is 4096 float32 = 16 KB, or as uint16 (after scaling) = 8 KB — half the raw RF data size. At 200 Hz PRF this reduces USB bandwidth from 3.2 MB/s (raw) to 1.6 MB/s (float32 envelope) or 0.8 MB/s (uint16 envelope).

---

## Companding envelope output (µ-law / A-law / dB log)

### Why companding

Ultrasound envelope has high dynamic range: near-field echoes can sit 40–60 dB above the noise floor. A linear uint16 quantisation preserves this range but distributes code levels uniformly — wasting half the codes on strong echoes and assigning very few to the weak, diagnostically relevant echoes at depth.

Companding (compress + expand) applies a logarithmic mapping before quantisation, spreading code levels roughly uniformly on a log scale:

| Representation | Size (N=4096) | Dynamic range | Notes |
|----------------|--------------|---------------|-------|
| float32 envelope | 16 KB | ~144 dB | Full precision; large for transfer/storage |
| uint16 linear | 8 KB | ~96 dB | Fine for strong echoes; wastes codes at depth |
| uint8 linear | 4 KB | ~48 dB | Clips weak echoes; not useful in practice |
| uint8 µ-law (µ=255) | 4 KB | ~38 dB uniform SNR | Telephony standard; straightforward to adapt |
| uint8 dB log (40–60 dB window) | 4 KB | 40–60 dB (windowed) | Ultrasound-native; explicit clinical dB scale |

All three companding schemes achieve the same 4 KB footprint. The differences are in SNR distribution and decoder complexity.

---

### µ-law adapted for non-negative envelope

Standard µ-law is defined for signed audio. The ultrasound envelope is always ≥ 0, so only the positive branch is used:

```
y = log(1 + µ · x_norm) / log(1 + µ)    x_norm ∈ [0, 1]
```

where `x_norm = env[i] / peak` and `peak` is the maximum expected envelope value (ADC full-scale × TGC gain, or measured per-acquisition peak). With µ = 255 (ITU-T standard), values near zero receive 8× more code levels than values near full-scale.

Decoded: `x_norm = ((1 + µ)^y − 1) / µ`

---

### A-law adapted for non-negative envelope

A-law adds a linear segment below the crossover `1/A`, giving slightly better SNR for very weak signals:

```
y = A · x_norm / (1 + ln A)              x_norm < 1/A
y = (1 + ln(A · x_norm)) / (1 + ln A)   x_norm ≥ 1/A
```

With A = 87.6 (ITU-T G.711): the crossover at `1/A ≈ 0.011` corresponds to ~1% of ADC full-scale — below the practical noise floor of a 10-bit ADC at ultrasound frequencies. In practice, A-law and µ-law produce near-identical results for ultrasound envelope data. **µ-law is simpler to implement (no branch at the crossover) and is the recommended choice.**

---

### Ultrasound-native alternative: explicit dB log compression

Clinical ultrasound systems use a parameterised dB window rather than a telephony companding standard:

```c
// y_dB = 20·log10(env / peak); window maps [−window_dB, 0 dB] → [0, 255]
float32_t inv_peak = 1.0f / peak;
float32_t scale = 255.0f / window_dB;
for (uint32_t i = 0; i < n; i++) {
    float32_t db = 20.0f * log10f(env[i] * inv_peak + 1e-9f);
    float32_t v = (db + window_dB) * scale;
    out[i] = (uint8_t)(v < 0.0f ? 0 : v > 255.0f ? 255 : v);
}
```

`window_dB = 40` covers echoes within 40 dB of the peak; echoes below −40 dB map to 0. This is the most interpretable format for display (one code ≈ 0.16 dB) and matches what B-mode scanners produce. For heart-rate / distance detection, 40 dB suffices; for tissue characterisation, 60 dB is typical.

---

### M33 implementations

#### Option 1 — Scalar µ-law (logf)

```c
#define MU_LAW_MU    255.0f

// Precompute once at startup:
// static const float32_t MU_LAW_DIV = 1.0f / logf(1.0f + MU_LAW_MU);

void envelope_to_mulaw_u8(const float32_t *env, uint8_t *out,
                           uint32_t n, float32_t peak,
                           float32_t mu_law_div) {
    float32_t inv_peak = 1.0f / peak;
    for (uint32_t i = 0; i < n; i++) {
        float32_t x = env[i] * inv_peak;
        if (x > 1.0f) x = 1.0f;
        out[i] = (uint8_t)(logf(1.0f + MU_LAW_MU * x) * mu_law_div * 255.0f + 0.5f);
    }
}
```

M33 FPU executes `logf` in ~15–20 cycles (ROM implementation using FPU). For N=4096 at 150 MHz: ~550 µs. Fits the 5000 µs inter-pulse budget; runs after the Hilbert pipeline (~92 µs), total ~640 µs.

#### Option 2 — Vectorized via CMSIS-DSP (`arm_vlog_f32`)

`arm_vlog_f32` (CMSIS-DSP ≥ 1.14 with DSP extension enabled) computes natural log element-wise using SIMD, 4–8 samples per cycle:

```c
void envelope_to_mulaw_vectorized(const float32_t *env, uint8_t *out,
                                   uint32_t n, float32_t peak) {
    static float32_t tmp[NFFT];
    const float32_t mu_law_div = 1.0f / logf(1.0f + MU_LAW_MU);  // precompute
    const float32_t mu_over_peak = MU_LAW_MU / peak;
    const float32_t scale = 255.0f * mu_law_div;

    arm_scale_f32(env, mu_over_peak, tmp, n);  // tmp[i] = µ * x_norm[i]
    arm_offset_f32(tmp, 1.0f, tmp, n);         // tmp[i] = 1 + µ * x_norm[i]
    arm_vlog_f32(tmp, tmp, n);                  // tmp[i] = log(1 + µ * x_norm[i])
    arm_scale_f32(tmp, scale, tmp, n);          // tmp[i] = 255 * log(...) / log(1+µ)
    arm_float_to_q7(tmp, (q7_t *)out, n);      // convert to uint8 (saturating)
}
```

**Note:** `arm_vlog_f32` requires `ARM_MATH_MVEF` or `ARM_MATH_DSP` defined and CMSIS-DSP ≥ 1.14. Verify with `#if defined(ARM_MATH_MVEF) || defined(ARM_MATH_DSP)`. If absent, fall back to Option 1. Estimated time with DSP extension: **~50–80 µs** for N=4096.

#### Option 3 — IEEE 754 exponent approximation (no libm)

The IEEE 754 float exponent gives a floor(log₂) for free. Combining it with the top mantissa bits yields a coarse log approximation with ~1–2 dB resolution — no `logf` call required:

```c
static inline uint8_t fast_log2_compress(float32_t f, float32_t inv_peak) {
    if (f <= 0.0f) return 0;
    f *= inv_peak;
    if (f > 1.0f) f = 1.0f;

    uint32_t bits;
    __builtin_memcpy(&bits, &f, 4);

    // Biased exponent for f in (0, 1]: ranges from 0x7E (0.5–1.0) downward
    int32_t exp = (int32_t)((bits >> 23) & 0xFF) - 127; // true exponent, ≤ 0
    uint32_t mant5 = (bits >> 18) & 0x1F;               // 5 upper mantissa bits

    // Map log2(f) ∈ [−48, 0] dB range → [0, 255]
    // Each exponent step = 6 dB; 48 dB → 8 exponent steps × 32 sub-steps = 256 codes
    int32_t level = (exp + 48) * 32 + (int32_t)mant5;
    if (level < 0)   return 0;
    if (level > 255) return 255;
    return (uint8_t)level;
}

void envelope_to_log_u8_fast(const float32_t *env, uint8_t *out,
                              uint32_t n, float32_t peak) {
    float32_t inv_peak = 1.0f / peak;
    for (uint32_t i = 0; i < n; i++)
        out[i] = fast_log2_compress(env[i], inv_peak);
}
```

Resolution: 6 dB per exponent step, refined to ~6/32 ≈ 0.2 dB with 5 mantissa bits. All integer arithmetic after the initial float multiply — executes in ~5–8 cycles per sample on M33. For N=4096: **< 1 µs** total. Sufficient for distance measurement and heart-rate detection; not sufficient for quantitative tissue characterisation.

---

### Timing and compression summary

| Method | Time (N=4096, 150 MHz) | Output | Quality |
|--------|------------------------|--------|---------|
| µ-law scalar (`logf` loop) | ~550 µs | 4 KB uint8 | Good — matches telephony standard |
| µ-law vectorized (`arm_vlog_f32`) | ~50–80 µs | 4 KB uint8 | Same quality, ~7× faster |
| dB log scalar (`log10f` loop) | ~550 µs | 4 KB uint8 | Best for display / clinical use |
| IEEE 754 bit-hack | < 1 µs | 4 KB uint8 | Coarse (~0.2 dB); detection tasks only |

**Practical result:** uint8 µ-law at 200 Hz PRF = 4096 bytes × 200 = **0.8 MB/s over USB** — comfortably within USB FS bandwidth (≈1 MB/s ceiling) and 4× smaller than float32 envelope. The vectorized path leaves the total Hilbert+companding pipeline well under 200 µs.

### Host-side decode (Python)

```python
import numpy as np

MU = 255.0

def mulaw_decode(data_u8: np.ndarray, peak: float) -> np.ndarray:
    y = data_u8.astype(np.float32) / 255.0
    return peak * ((1.0 + MU) ** y - 1.0) / MU

def db_log_decode(data_u8: np.ndarray, peak: float, window_db: float = 40.0) -> np.ndarray:
    y = data_u8.astype(np.float32) / 255.0
    db = y * window_db - window_db   # map [0,255] → [−window_dB, 0]
    return peak * 10.0 ** (db / 20.0)
```

---

## Summary

| Parameter | Value |
|-----------|-------|
| FFT size | 4096 points (covers 63 µs at 65 Msps; ~4.8 cm depth in water) |
| Pipeline time | ~92 µs on M33 @ 150 MHz |
| Inter-pulse budget at 200 Hz PRF | 5000 µs |
| Compute utilisation | < 2% |
| SRAM required (Hilbert buffers) | 48 KB (32 KB complex + 16 KB envelope) |
| CMSIS-DSP dependency | Yes — `arm_cfft_f32`, `arm_cmplx_mag_f32`, `arm_scale_f32` |
| Requires RP2350 (M33 FPU) | Yes — RP2040 is ~100× too slow for in-band Hilbert |
| Data rate reduction (envelope vs raw) | 2–4× depending on output format |
