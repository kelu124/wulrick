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
