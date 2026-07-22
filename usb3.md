# USB Bridge Chip Survey for ultr4rick

Comprehensive comparison of USB bridge chips for high-speed data transfer (>40 MB/s) with composite device support (bulk data + CDC-ACM serial simultaneously). Target platform: ultr4rick (RP2350B).

**Background:** `FT2232H.md` and `CH569.md` cover those two chips in depth. This file surveys the wider landscape — FTDI FT600/FT601, Infineon EZ-USB FX2LP and FX3, WCH CH347, and the RP2350B's own built-in USB — with enough detail to make an informed board-level choice.

---

## ultr4rick data rate requirement

At 65 Msps × 10 bit × 8192 samples/frame:
- Raw per-frame: 65 × 10 × 8192 / 8 = **666 kB/frame**
- At 200–400 Hz PRF (typical for pulsed echo): **130–260 MB/s raw** before averaging
- After 4–16× on-board averaging (standard for SNR): **8–65 MB/s to host**
- Practical target for real-time streaming: **10–20 MB/s sustained** with headroom to 40 MB/s for burst capture

USB 2.0 HS (~40 MB/s practical) is adequate for most operating modes. USB 3.0 SS (>100 MB/s) provides margin for high-PRF burst modes or simultaneous multi-channel acquisition. The composite device requirement — CDC-ACM for control/status alongside bulk for data — is a host-driver convenience that simplifies cross-platform software.

---

## Master comparison table

| Chip | Vendor | USB gen | Practical throughput | Interface to RP2350B | True CDC+bulk composite | External firmware | Price (1 pc) | Complexity |
|------|--------|---------|---------------------|---------------------|------------------------|-------------------|-------------|------------|
| **RP2350B (built-in)** | Raspberry Pi | 2.0 HS | 20–35 MB/s | — (no chip) | ✓ TinyUSB | No | $0 (on-board) | Low |
| **FT2232H** | FTDI | 2.0 HS | ~40 MB/s | 8-bit sync FIFO 245 | ✗ (two vendor channels) | No | ~$4–6 | Low–Med |
| **CH347** | WCH | 2.0 HS | ~5–10 MB/s | SPI up to 60 MHz | ✓ native | No | ~$1–2 | Low |
| **CY7C68013A (FX2LP)** | Infineon (Cypress) | 2.0 HS | ~40 MB/s | 8-bit GPIF / ext FIFO | ✓ firmware | Yes (8051 C) | ~$3–5 | High |
| **CH569** | WCH | 3.0 SS | 50–60 MB/s (8-bit HSPI) / 100–120 MB/s (16-bit) | 8/16-bit HSPI | ✓ firmware | Yes (RISC-V C) | ~$1.5–3 | Med–High |
| **FT600** | FTDI | 3.0 SS | ~200 MB/s | 8-bit sync FIFO | ✗ (vendor bulk only) | No | ~$8–10 | Med |
| **FT601** | FTDI | 3.0 SS | ~400 MB/s | 16-bit sync FIFO | ✗ (vendor bulk only) | No | ~$10–14 | Med |
| **EZ-USB FX3** | Infineon (Cypress) | 3.0 SS | ~300–400 MB/s | 8/16/32-bit GPIF II | ✓ firmware | Yes (ARM C) | ~$8–12 | Very High |

*Throughput figures are sustained bulk transfer to host, not raw USB link rate.*

---

## Per-chip analysis

### RP2350B built-in USB (no extra chip)

The RP2350B contains a USB 2.0 HS OTG PHY and controller. With TinyUSB (the standard USB stack for RP2350), a composite CDC-ACM + vendor bulk device is a 20-line configuration change.

**Throughput:** 20–35 MB/s sustained bulk in practice. This is below the USB 2.0 HS theoretical ceiling (60 MB/s) because:
- The Cortex-M33 must service USB interrupts while running acquisition
- DMA chains to the USB FIFO compete with ADC DMA
- RP2350B dual-core helps: USB handling on core 1, acquisition on core 0

For ultr4rick at 10–20 MB/s required, this is sufficient. Burst-mode acquisition (capture all frames, then upload) can saturate USB at ~35 MB/s without real-time constraints.

**Composite support:** TinyUSB implements true USB composite descriptors. CDC-ACM interface for UART-like commands (e.g., set gain, set PRF, trigger acquisition) coexists with bulk transfer interface for sample data — no driver installation on Linux/macOS, and WinUSB handles the bulk endpoint on Windows.

**Interface:** No external chip. RP2350B USB D+/D- lines connect directly to USB-C connector with 5.1 kΩ CC resistors. Total BOM cost: 2 resistors + USB-C connector.

**Caveats:**
- 35 MB/s ceiling may bind if burst PRF > 200 Hz with no averaging
- USB handling and ADC PIO must share DMA channels; careful DMA ring configuration required
- Cannot achieve >40 MB/s without an external USB chip

**Best for:** Low-cost single-board ultr4rick where 35 MB/s suffices and bill-of-materials simplicity is the priority.

---

### FTDI FT2232H

Documented in `FT2232H.md`. Summary:

- USB 2.0 HS, synchronous FIFO 245 mode: ~40 MB/s
- Two independent channels (A/B); Channel A as sync FIFO, Channel B as async UART → *appears* composite but both enumerate as vendor-class FTDI interfaces, not CDC-ACM
- No custom firmware required; D2XX library or libftdi on host
- Price ~$4–6; widely available, well-documented

**Composite caveat:** FT2232H does not implement CDC-ACM. On Windows, both channels install as COM ports via FTDI VCP driver or as D2XX devices. On Linux, both appear as ttyUSB nodes via ftdi_sio driver. This is functionally similar to CDC composite but requires the FTDI driver — not driver-free like true CDC-ACM + WinUSB.

**Interface to RP2350B:** 8-bit data bus + CLKOUT + RD# + WR# + RXF# + TXE# = 13 GPIO. RP2350B PIO handles this at 60 MHz (Channel A FIFO clock). See `FT2232H.md` for complete pin mapping and PIO programs.

---

### WCH CH347

The CH347 is a USB 2.0 HS bridge with three on-chip interfaces: UART (up to 9 Mbps), SPI (up to 60 MHz), and I²C (up to 3.4 MHz). It enumerates natively as a USB composite device: one CDC-ACM interface for UART and one vendor interface for SPI/I²C — no driver development needed.

**Throughput:** SPI at 60 MHz, 8-bit = 7.5 MB/s theoretical; practical SPI bulk ~5–8 MB/s due to USB transaction overhead. This is well below 40 MB/s.

**Why include it despite low throughput?** CH347 is the easiest path to true composite CDC + bulk with no firmware and minimal GPIO cost. For ultr4rick use cases where SPI is used for sample streaming (averaged, decimated data to a display or command interface) and the full 40 MB/s rate is not required, it is the lowest-complexity option.

**Interface to RP2350B:** SPI slave or SPI master depending on direction. RP2350B SPI peripheral or PIO SPI to CH347 SPI master. 4 GPIO (MOSI, MISO, SCK, CS).

**Caveats:**
- Hard ceiling of ~8 MB/s via SPI — does not meet the >40 MB/s requirement
- Windows requires WCH CH347 driver installation (not inbox)
- Linux: `ch347` kernel module available in mainline since 5.18 for UART mode; SPI requires `spi-ch347` which is an out-of-tree driver or recent kernel (6.4+)
- Not suitable for real-time raw ADC streaming at 65 Msps

**Best for:** Control/status channel only, or low-bandwidth processed data output alongside a second chip for bulk ADC data.

---

### Infineon (Cypress) EZ-USB FX2LP — CY7C68013A

The EZ-USB FX2LP is a classic high-speed USB device controller with an integrated 8051 processor and a configurable parallel interface (GPIF). It has been used in hundreds of data acquisition designs since 2004 — RTL-SDR dongles, logic analyzers, custom ADC boards — and has one of the largest communities of any USB bridge chip.

**Architecture:**
- USB 2.0 HS (480 Mbps)
- 8051 core at 48 MHz (12 clock cycles per instruction for legacy 8051 timing; the FX2LP uses accelerated 8051 with single-cycle instructions on some operations)
- GPIF (General Purpose Interface): configurable waveform-based 8-bit parallel interface
- 8 bulk endpoints, 2 isochronous endpoints
- 16 KB internal FIFO buffer
- External FIFO mode: GPIF can be configured as a slave FIFO, where the external device (RP2350B) writes data and the 8051 streams it to the USB host

**Throughput:** ~40 MB/s in bulk transfer, limited by USB 2.0 HS overhead. GPIF in external FIFO mode with RP2350B writing at 48 MHz: 48 MB/s raw input, USB limited to ~40 MB/s out.

**Composite device:** FX2LP firmware can implement any USB descriptor configuration. A standard composite CDC-ACM + vendor bulk device is a common example in the FX2LP community (Sigrok libsigrok uses this approach). The 8051 firmware configures the endpoints and handles USB control requests; data transfer is handled by GPIF hardware without CPU involvement.

**Interface to RP2350B:**
External FIFO mode (slave FIFO): the RP2350B acts as FIFO master. Signals:
- FD[7:0]: 8-bit data bus → 8 GPIO
- IFCLK: interface clock (RP2350B provides, up to 48 MHz) → 1 GPIO
- SLRD: slave read strobe → 1 GPIO
- SLWR: slave write strobe → 1 GPIO
- FLAGA, FLAGB: endpoint full/empty flags → 2 GPIO
- SLOE: slave output enable → 1 GPIO
- Total: 14 GPIO, similar to FT2232H

RP2350B PIO can drive this interface. The PIO state machine asserts SLWR on the rising IFCLK edge when FLAGB (endpoint not full) is asserted, clocking in data at up to 48 MB/s.

**8051 firmware development:**
Infineon (Cypress) provides the EZ-USB FW Framework — a C library for 8051 that abstracts USB descriptor setup, endpoint configuration, and GPIF waveform programming. The external FIFO example is included in the SDK. Build tools: SDCC (open-source 8051 compiler) or Keil C51.

Typical firmware for a slave FIFO → bulk transfer is ~200 lines of C. The 8051 is not in the data path for GPIF transfers; it handles USB setup packets, endpoint stalls, and vendor commands.

**EEPROM / boot:** The FX2LP can load firmware from an I²C EEPROM at power-up, or from USB via the Cypress Vendor Command bootstrap. A 64 KB EEPROM (e.g., 24LC512) holds the firmware image. At first power, a blank FX2LP enumerates as a generic Cypress device, then the host loads firmware; with EEPROM, it enumerates directly as the target device.

**Price and availability:** CY7C68013A-56LTXC (QFN-56): ~$3–5 at DigiKey/Mouser. Stock generally good (tens of thousands).

**Caveats:**
- 8051 firmware is required; SDCC toolchain setup adds complexity vs FT2232H (no firmware)
- 40 MB/s ceiling — same as FT2232H in sync mode
- 8051 is a 40-year-old architecture; the community is large but the toolchain is dated
- QFN-56 package: 8 × 8 mm, 0.5 mm pitch — manageable but requires reflow, not hand-solderable

**Best for:** Maximum flexibility in USB descriptor design (true composite CDC + bulk), community support, well-characterized external FIFO interface, familiar pattern for SDR/logic-analyzer-style acquisition.

---

### WCH CH569

Documented in `CH569.md`. Summary for comparison:

- USB 3.0 SuperSpeed (5 Gbps physical link)
- RISC-V RV32 core at 120 MHz; user firmware in C via WCH SDK
- HSPI interface to RP2350B: 8-bit PIO at up to 120+ MHz → 50–60 MB/s; 16-bit at ~100 MHz → 100–120 MB/s
- Serial SPI mode: 9.4 MB/s (inadequate for bulk streaming)
- Composite: RISC-V firmware can implement any descriptor; CDC-ACM + bulk is feasible but requires USB 3.0 SS composite descriptor work
- Price: ~$1.5–3; stock variable on LCSC
- Caveats: WCH SDK documentation primarily in Mandarin; WCH community support via WCHLink forums; HSPI 16-bit mode requires 16-bit wide PIO program on RP2350B

See `CH569.md` for full pin mapping, PIO programs, and throughput analysis.

---

### FTDI FT600 / FT601 (FT60xQ series)

The FT600 and FT601 are FTDI's USB 3.0 SuperSpeed bridge chips, introduced in 2016 as successors to the FT2232H for high-throughput applications. They expose a simple synchronous parallel FIFO to the external device — the same conceptual interface as the FT2232H sync FIFO 245 mode, but faster and on USB 3.0 SS.

#### FT600 vs FT601

| Parameter | FT600 | FT601 |
|-----------|-------|-------|
| Data bus | 8-bit | 16-bit |
| FIFO clock | Up to 100 MHz | Up to 100 MHz |
| Max throughput | ~200 MB/s | ~400 MB/s |
| Channels | 1 or 2 × 245 FIFO | 1 or 2 × 245 FIFO |
| Package | QFN-76 (9 × 9 mm) | QFN-76 (9 × 9 mm) |
| Price | ~$8–10 | ~$10–14 |

Both chips connect to the host via USB 3.0 SuperSpeed (5 Gbps physical); the FIFO bus connects to the FPGA/MCU side.

**Architecture:**
- No embedded CPU; the chip contains a USB 3.0 PHY, FIFO controller, and parallel FIFO interface
- Configuration via one-time-programmable (OTP) memory or over USB via the D3XX API
- No custom firmware: like FT2232H, the internal firmware is fixed by FTDI
- Two FIFO channels available; each can be assigned to an endpoint

**Throughput:**
- FT600: 8-bit × 100 MHz = 800 Mbps = 100 MB/s raw FIFO rate; USB 3.0 SS overhead → ~200 MB/s sustained is the rated figure because the FIFO operates in both directions (bidirectional) and the USB 3.0 link allows ~350 MB/s in one direction; in practice ~150–200 MB/s write to host is observed
- FT601: 16-bit × 100 MHz = 1600 Mbps = 200 MB/s raw FIFO, USB SS → ~300–400 MB/s claimed

**Interface to RP2350B (FT600, 8-bit):**

| Signal | Direction | RP2350B GPIO | Notes |
|--------|-----------|-------------|-------|
| DATA[7:0] | Bidirectional | 8 GPIO | Data bus |
| CLK | Output from FT600 | 1 GPIO input | FIFO clock, up to 100 MHz |
| RXF# | Output from FT600 | 1 GPIO input | RX FIFO not empty (data available from host) |
| TXE# | Output from FT600 | 1 GPIO input | TX FIFO not full (safe to write to host) |
| RD# | Input to FT600 | 1 GPIO output | Read strobe |
| WR# | Input to FT600 | 1 GPIO output | Write strobe |
| OE# | Input to FT600 | 1 GPIO output | Output enable for RX direction |
| RESET# | Input to FT600 | 1 GPIO output | Reset |
| **Total** | | **15 GPIO** | |

RP2350B PIO can drive this at 100 MHz (within the 150 MHz PIO ceiling). The PIO state machine is identical in structure to the FT2232H sync FIFO 245 PIO — same handshake signals, same clocked bus pattern — just running at 100 MHz instead of 60 MHz with the USB 3.0 SS link on the other side.

**FT601 (16-bit) with RP2350B:** 16-bit data bus × 100 MHz = 200 MB/s raw. RP2350B has 30 user-accessible GPIO (GPIO 0–29 on standard boards); the 16-bit bus + 7 control = 23 GPIO. Feasible on RP2350B if the ADC bus (10 GPIO) uses the remaining 7 pins, but pin budget becomes tight. Requires careful assignment.

**Composite device support:** The FT600/FT601 enumerate as a single vendor-class bulk device. The D3XX API (FTDI's USB 3.0 driver) exposes the two FIFO channels. There is no CDC-ACM interface; the serial control channel must be implemented as a second vendor bulk channel in software. This is functional but requires the D3XX library and is not driver-free on Windows (D3XX.dll needed). True CDC-ACM + bulk composite is not available without firmware modification — and the FT600/FT601 has no user-programmable firmware.

This is the main limitation of the FT600/FT601: maximum throughput but no true composite support.

**Host driver:**
- Windows: D3XX.dll (FTDI library), not inbox
- Linux: `ftd3xx` library; also `ft60x` driver in staging tree since kernel 5.7 (incomplete — no stable CDC interface)
- macOS: D3XX.dylib

**Price and availability:** FT600Q-B (QFN-76) at DigiKey: ~$8–10. Stock: 200–500 pcs typical. Lead time: 8–14 weeks if out of stock.

**Caveats:**
- No CDC-ACM: control/status must go over the same bulk channel (multiplexing required) or a second USB device
- D3XX driver required; more host-side complexity than FT2232H (which has ftdi_sio in kernel)
- 100 MHz FIFO: RP2350B PIO is at its ceiling; marginal timing
- QFN-76 at 0.5 mm pitch: requires stencil and reflow; not hand-solderable
- FT601 16-bit bus consumes most of RP2350B's GPIO budget

**Best for:** Maximum throughput (200+ MB/s) when CDC composite is not required and host-side complexity is acceptable. Pairs well with an additional CH347 or FT2232H Channel B for the serial control channel.

---

### Infineon (Cypress) EZ-USB FX3

The EZ-USB FX3 is the most powerful option in this survey. It is a USB 3.0 SuperSpeed controller with a 200 MHz ARM926EJ-S core, a 32-bit configurable parallel interface (GPIF II), and full freedom over USB descriptors. It is the chip of choice for USB 3.0 data acquisition systems, SDRs (LimeSDR, Ettus USRP), and industrial cameras.

**Architecture:**
- USB 3.0 SS (5 Gbps) + USB 2.0 HS fallback
- ARM926EJ-S at 200 MHz (with MMU, cache, and DSP extensions)
- GPIF II: 32-bit configurable parallel interface at up to 100 MHz → 400 MB/s raw
- 512 KB internal SRAM (on CYUSB3014); 16 KB DMA buffers per endpoint
- 16 bulk endpoints, 4 isochronous, 4 interrupt
- SPI, I²C, UART serial interfaces on-chip for peripheral access
- SPI boot: loads firmware from SPI flash at power-up (no EEPROM required; SPI flash is simpler)

**Throughput:** 300–400 MB/s in USB 3.0 SS bulk transfer mode; 40 MB/s in USB 2.0 HS fallback. The GPIF II can sustain 400 MB/s into the DMA buffers, which drain via USB 3.0 SS.

**Composite device:** Complete freedom. The ARM firmware can define any USB device descriptor — CDC-ACM + bulk + isochronous + HID, any combination. Standard CDC-ACM + vendor bulk is a 50-line USB descriptor definition in the FX3 Application Framework. This is the most capable composite implementation on this list.

**Interface to RP2350B via GPIF II:**
The GPIF II is a state-machine-based interface that can be configured for many bus protocols. For RP2350B:

- 8-bit GPIF II (sync, RP2350B as master): 8 data + 4 control (PMODE, CLK, valid, ready) = 12 GPIO
- 8-bit at 100 MHz (GPIF II clock from FX3 or RP2350B) → ~100 MB/s raw to FX3 DMA
- 32-bit GPIF II: not viable for RP2350B (32 GPIO bus is more than RP2350B can spare)

RP2350B PIO drives the 8-bit GPIF II synchronous FIFO mode at up to 100 MHz.

**Firmware development:**
- Infineon EZ-USB FX3 SDK (Eclipse-based, free)
- FX3 Application Framework: C library for USB descriptors, DMA setup, GPIF II waveform configuration
- GPIF II Designer: GUI tool that generates C state-machine code for custom interface waveforms
- Build: ARM toolchain (arm-none-eabi-gcc); standard Makefile
- Boot: SPI flash (16 Mbit minimum; e.g., Winbond W25Q16)
- Debug: UART console via FX3 debug UART

A typical FX3 firmware project for GPIF II slave FIFO → USB 3.0 bulk: ~500–1000 lines of C including the FX3 Application Framework callbacks. More complex than FX2LP 8051 firmware, but the ARM is a modern CPU and the SDK is more ergonomic.

**Caveats:**
- **Overkill for ultr4rick**: 300–400 MB/s is an order of magnitude above the 10–20 MB/s requirement; the FX3 is designed for FPGA interfaces
- **Cost**: ~$8–12 vs ~$1.5–3 for CH569; ~$3–5 for FX2LP
- **Firmware**: ARM SDK is required; 2–3 day ramp-up for someone unfamiliar
- **Package**: CYUSB3014-BZXC (BGA-121, 0.8 mm ball pitch) — requires BGA reflow and X-ray inspection; not suitable for prototype hand-assembly; CYUSB3011-BZXI (QFN-121) is better for prototyping
- **Supply**: Infineon acquired Cypress in 2020; FX3 is still produced but lead times of 20–40 weeks have been seen
- **USB 3.0 SS host required** for full throughput; falls back to USB 2.0 HS (~40 MB/s) on older systems

**Best for:** Designs that pair RP2350B with an FPGA (where the FPGA provides 32-bit GPIF II data), or systems requiring >100 MB/s sustained with true composite descriptors and full protocol flexibility.

---

## Composite device support — detailed comparison

True USB composite device = single VID:PID, single device descriptor, multiple interface descriptors (CDC-ACM + vendor bulk in parallel).

| Chip | CDC-ACM interface | Bulk data interface | Windows driver-free? | Linux driver-free? | Notes |
|------|------------------|--------------------|--------------------|------------------|-------|
| **RP2350B + TinyUSB** | ✓ native | ✓ native | ✓ (WinUSB for bulk, CDC via inbox usbser) | ✓ (cdc_acm + usbfs/libusb) | Easiest true composite |
| **FT2232H** | ✗ (vendor class, VCP driver) | ✗ (vendor class, D2XX) | ✗ (FTDI VCP driver) | ~ (ftdi_sio in kernel, not CDC) | Functionally similar but not true CDC |
| **CH347** | ✓ (CDC-ACM, UART) | ~ (vendor bulk via SPI) | ✗ (WCH driver) | ~ (ch347 in kernel ≥6.4) | Limited throughput; CDC is real |
| **FX2LP (CY7C68013A)** | ✓ firmware | ✓ firmware | ✓ (CDC inbox; WinUSB for bulk) | ✓ (cdc_acm + libusb) | Best flexibility at USB 2.0 HS |
| **CH569** | ✓ firmware | ✓ firmware | ✓ | ✓ | Requires RISC-V firmware |
| **FT600/FT601** | ✗ | ✓ (D3XX) | ✗ (D3XX driver) | ~ (staging driver) | No CDC; D3XX only |
| **EZ-USB FX3** | ✓ firmware | ✓ firmware | ✓ | ✓ | Most capable; ARM firmware required |

"Driver-free" means: on a freshly installed OS, the device works without installing a vendor-specific driver package.

---

## Throughput vs. complexity matrix

```
Throughput
400 MB/s │                                          FX3 ◆
         │
200 MB/s │             FT601 ◆          FT600 ◆
         │
100 MB/s │                      CH569 (16-bit) ◆
         │
 50 MB/s │         CH569 (8-bit) ◆
         │
 40 MB/s │  FX2LP ◆     FT2232H ◆
         │
 35 MB/s │  RP2350B ◆
         │
 10 MB/s │  CH347 ◆
         └──────────────────────────────────────────────
         Low        Medium        High       Very High
                    Complexity (firmware + integration)
```

---

## Recommendation matrix

| Scenario | Best choice | Rationale |
|----------|-------------|-----------|
| **Lowest cost, adequate speed (≤35 MB/s)** | RP2350B native USB | No external chip; TinyUSB composite out of the box |
| **40 MB/s + composite + no firmware dev** | FT2232H | Proven, well-documented, no firmware; pseudo-composite acceptable |
| **40 MB/s + true CDC+bulk composite** | FX2LP (CY7C68013A) | Firmware required but community is large; true composite; ~$4 |
| **50–120 MB/s + lowest cost** | CH569 | RISC-V firmware required; best price/throughput at USB 3.0 SS |
| **200 MB/s, no firmware dev** | FT600 | Plug-and-play at high throughput; no CDC composite |
| **Maximum throughput + composite** | EZ-USB FX3 | ARM firmware, full USB 3.0, any descriptor; overkill for ultr4rick |
| **Composite only, data via SPI (≤10 MB/s)** | CH347 | Simplest composite; not for raw ADC streaming |

**For ultr4rick specifically:**

The RP2350B native USB path covers the most likely operating modes (10–35 MB/s) at zero extra cost. If burst-mode or high-PRF capture exceeds 35 MB/s, the CH569 with 8-bit HSPI (50–60 MB/s) is the next step — already documented in `CH569.md`. The FX2LP is the best choice if true CDC-ACM composite is required at 40 MB/s without a firmware development investment on a RISC-V target.

The FT600 and FX3 are both significantly over-spec for ultr4rick's data rates and introduce cost and complexity without a corresponding benefit at the current architecture.

---

## Signal integrity notes

### USB 3.0 SuperSpeed differential pairs (CH569, FT600, FT601, FX3)

USB 3.0 SS uses two differential pairs: TX (board to host) and RX (host to board).

| Parameter | Spec | Notes |
|-----------|------|-------|
| Differential impedance | 90 Ω ± 10% | 45 Ω each trace to adjacent plane |
| Intra-pair skew | ≤ 1.7 ps | ≤ 0.33 mm trace length difference within a pair |
| Inter-pair skew | ≤ 6 ns | TX and RX pairs; manageable |
| Reference plane | Continuous GND | No slots under SS pairs |
| AC coupling caps | 100 nF per line (series) | Required by USB 3.0 spec; C0G 0402 |
| ESD protection | Required | PRTR5V0U2X or similar; <0.5 pF capacitance |
| Trace routing | Away from switching regulators, clock oscillators | SS pairs are 5 Gbps; EMI coupling is significant |

USB 2.0 HS pairs (D+/D−) also present on all chips: 90 Ω differential, same rules as USB 3.0 but less strict on skew (USB 2.0 HS runs at 480 Mbps vs 5 Gbps).

### High-speed parallel FIFO bus (FT2232H, FT600, FX2LP, FX3)

The parallel FIFO bus between the bridge chip and RP2350B is a chip-to-chip interface (not a connector), typically 3–5 cm trace length:

- Keep traces short and equal length (within 0.5 mm for buses above 60 MHz)
- Route over continuous GND plane
- Avoid vias on data bus traces if operating above 80 MHz
- Series termination resistors (22–33 Ω) on each data line reduce reflections if trace > 5 cm

---

*Analysis compiled July 2026 for ultr4rick. Throughput figures from manufacturer datasheets and community benchmarks. FX2LP and FX3 documentation from Infineon EZ-USB SDK. USB 3.0 signal integrity from USB-IF CTS specification Rev. 1.0.*
