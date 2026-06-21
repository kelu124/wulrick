# Wulrick — Research Link Tracker

Every external source used during this research is logged here with a description of its contents and why it was consulted.

---

## pic0rick — Primary Sources

### https://github.com/kelu124/pic0rick
The main open-source repository for the pic0rick device, created by kelu124. Contains full hardware design files (KiCad schematics and PCBs for the ADC board, pulser board, HV supply board, MUX board, and VGA board), firmware source code (RP2040 PIO drivers, MicroPython), BOMs in CSV format, and Gerber files for manufacturing. This is the primary source for all hardware-level analysis of pic0rick.

### https://un0rick.cc/pic0rick
The official product page for pic0rick on the un0rick.cc documentation site. Provides a high-level description of the device's purpose, hardware overview, and links to getting-started resources. Used for initial orientation to the project's design goals and intended users.

### https://un0rick.cc/
The home page for the un0rick open hardware ecosystem — the broader family of ultrasound boards that pic0rick belongs to, including the earlier FPGA-based un0rick and lit3rick boards. Provides context for how pic0rick evolved from prior designs and what the ecosystem offers (documentation, use cases, partner integrations).

### https://zenodo.org/records/10968504
The Zenodo-hosted academic paper describing the pic0rick system. Authored by kelu124, this is the primary peer-reviewed reference for the device. Contains technical description, design rationale, and performance characterization. Used for design decision rationale and any published performance figures.

### https://www.tindie.com/products/kelu124/pic0rick-a-pico-ultrasound-pulse-echo-system/
The Tindie product listing where pic0rick is sold pre-assembled. Provides the retail price, a product description for a general maker audience, and links to documentation. Consulted for cost information and accessibility assessment.

### https://www.hackster.io/news/kelu-ghosh-s-raspberry-pi-pico-w-powered-pic0rick-delivers-low-cost-ultrasound-experimentation-128fce4dc114
A Hackster.io news article covering the pic0rick launch. Provides a journalist's summary of the device's capabilities and positioning relative to earlier FPGA-based designs. Used as a secondary source confirming the device's purpose and target audience.

### https://un0rick.cc/use_cases
The use cases page of the un0rick site, listing published applications of the un0rick/pic0rick platform. Used to understand what the device has been successfully applied to in practice (NDT, tomography, array imaging, etc.).

### https://www.opensourceimaging.org/project/un0rick/
The Open Source Imaging initiative's entry for the un0rick project. A brief summary aimed at the open medical imaging community. Confirms the project's open-source status and provides a secondary description of capabilities.

### https://certification.oshwa.org/fr000023.html
The Open Source Hardware Association (OSHWA) certification page for pic0rick, certification number FR000023, awarded October 2024. Confirms the device's open hardware status and provides the official record of its certification.

### https://raw.githubusercontent.com/kelu124/pic0rick/main/hardware/adc/BOM_PICO_ADC_Test.csv
The Bill of Materials for the pic0rick ADC main board, in CSV format. Lists all 23 component types used on the board including: ADC10065CIMT/NOPB (10-bit 65 Msps ADC), AD8331ARQZ (TGC amplifier), MCP4812-E/MS (12-bit SPI DAC), MD0100N8-G (HV protection MOSFET), and the Raspberry Pi Pico. The primary source for component identification on the main board.

### https://raw.githubusercontent.com/kelu124/pic0rick/main/hardware/pulser/pulser_panel/jlcpcb/production_files/BOM-panel.csv
The Bill of Materials for the pic0rick pulser board panel. Identifies the core pulser components: MD1213K6-G (MOSFET driver), TC6320TG-G (high-voltage gate driver), SN74LVC2G04DBVT (logic inverters), and R2D-0524_P (isolated DC-DC converter). Primary source for pulser circuit component identification.

### https://raw.githubusercontent.com/kelu124/pic0rick/main/hardware/mux/jlcpcb/production_files/BOM-MAX14866_PMOD.csv
The Bill of Materials for the pic0rick MUX PMOD board. Identifies MAX14866UTM+T as the 8-channel analog multiplexer. This board is a PMOD expansion module, not part of the core system, but relevant for understanding the extensibility architecture.

### https://github.com/kelu124/pic0rick/raw/main/hardware/pulser/pulser_v1.0/pulser_schematics.pdf
The PDF schematic for the pic0rick pulser board (v1.0), downloaded for analysis. A 337 KB PDF showing the full pulser circuit. Consulted for understanding the pulser topology, component connections, and design decisions.

---

## WULPUS — Primary Sources

### https://github.com/pulp-bio/wulpus
The main open-source repository for the WULPUS device, maintained by the PULP group at ETH Zurich IIS. Contains full hardware design files (Altium schematics and PCBs for the Acquisition PCB and HV PCB), firmware for MSP430FR5043 and nRF52832/nRF52840, Python host software, and complete fabrication files. The primary source for all hardware-level analysis of WULPUS.

### https://github.com/pulp-bio/wulpus/raw/main/hw/wulpus_acquisition_pcb/docs/wulpus_acq_pcb_schematics.PDF
PDF export of the WULPUS Acquisition PCB schematics (1.4 MB), generated from Altium Designer by Sergei Vostrikov. Contains five schematic sheets: power supply (00_POWER), nRF52 BLE module (01_NRF52), MSP430 microcontroller (02_MSP430), MSP430 analog front-end (03_MSP430_AFE), a module called MR_WOLF (04_MR_WOLF), and the IMU (05_IMU). Downloaded locally for component and topology analysis.

### https://github.com/pulp-bio/wulpus/raw/main/hw/wulpus_hv_pcb/docs/wulpus_hv_mux_schematics.PDF
PDF export of the WULPUS High-Voltage PCB schematics (547 KB). Contains two schematic sheets: power management (00_POWER) and the transducer multiplexer circuit (01_TRANSDUCERS). Downloaded locally for analysis of the HV supply topology and 8-channel mux implementation.

### https://github.com/pulp-bio/wulpus/raw/main/hw/wulpus_acquisition_pcb/docs/wulpus_acq_pcb_bom.PDF
PDF export of the WULPUS Acquisition PCB Bill of Materials (150 KB), generated by Altium Designer. Lists all components on the acquisition board. Downloaded locally; text extraction pending (PDF uses binary encoding from Altium).

### https://github.com/pulp-bio/wulpus/raw/main/hw/wulpus_hv_pcb/docs/wulpus_hv_mux_bom.PDF
PDF export of the WULPUS HV PCB Bill of Materials (147 KB). Lists all components on the HV/mux board. Downloaded locally; text extraction pending.

### https://ieeexplore.ieee.org/document/9958156/
The primary IEEE paper on WULPUS: "WULPUS: a Wearable Ultra Low-Power Ultrasound probe for multi-day monitoring of carotid artery and muscle activity," presented at IEEE EMBC 2022. The authoritative published reference for WULPUS performance figures, design decisions, and validation results. Used for measured performance data (power consumption, data rates, published SNR/application results).

### https://ieee-uffc.org/post/news/wulpus-open-source-wearable-ultra-low-power-ultrasound-probe
A news post by the IEEE Ultrasonics, Ferroelectrics, and Frequency Control (UFFC) Society covering the WULPUS open-source release. Provides a community-facing summary of the device's specifications and design philosophy. Used as a secondary source confirming key specs and the open-source release context.

### https://www.researchgate.net/publication/365952632_WULPUS_a_Wearable_Ultra_Low-Power_Ultrasound_probe_for_multi-day_monitoring_of_carotid_artery_and_muscle_activity
ResearchGate entry for the WULPUS IEEE paper. Provides the abstract and metadata, accessible without IEEE subscription. Used to confirm published specifications and author affiliations (ETH Zurich IIS, PULP group).

### https://www.pcbway.com/project/shareproject/Wearable_Ultra_Low_Power_UltraSound_acquisition_system_with_BLE_connectivity_ad31c35b.html
PCBWay community page for the WULPUS Acquisition PCB. Provides an independent listing with board specifications and cost information for manufacturing. Consulted for cost estimation and as a secondary source on board physical specs.

---

## Component Datasheets

### https://www.ti.com/product/ADC10065
Texas Instruments product page for the ADC10065, the 10-bit 65 Msps ADC used on the pic0rick main board. Key specs: 10-bit resolution, 65 Msps sampling rate, ENOB 9.6 bits, SNR 59.6 dB at 11 MHz input, 68.4 mW power, 400 MHz input bandwidth, single +3.0V supply, 28-pin TSSOP. Used for ADC performance characterization in Phase 1.

### https://www.analog.com/en/products/ad8331.html
Analog Devices product page for the AD8331 ultralow noise VGA (Variable Gain Amplifier) with TGC, used on the pic0rick main board. Provides gain range (7.5–55.5 dB), noise figure, bandwidth, and SPI-compatible gain control interface specs. Consulted for TGC/amplifier block analysis. (Note: page returned 502 on first fetch; data sourced from datasheet included in the pic0rick repo at `hardware/adc/datasheet/AD8331_8332_8334-3119220.pdf`.)

### https://www.ti.com/product/MSP430FR5043
Texas Instruments product page for the MSP430FR5043, the dedicated ultrasound microcontroller used in WULPUS. Key features: integrated USS_A (Ultrasonic Sensing Solution) peripheral with 12-bit SDHS ADC at up to 8 Msps, programmable pulse generator (PPG), integrated PHY with 4 Ω output driver, 16 MHz clock, 64 KB FRAM, 450 nA standby. The USS peripheral automates pulse generation and acquisition sequencing, eliminating the need for external ultrasound timing circuits. Used for digital control and timing block analysis in Phase 1.

---

## Component Datasheets — Phase 1 (continued)

### https://ww1.microchip.com/downloads/en/DeviceDoc/20005713B.pdf
Microchip/Supertex MD1213 high-speed dual MOSFET driver datasheet. The MD1213K6-G is the gate driver used in the pic0rick pulser board to drive the TC6320 power MOSFETs. Key specs: 2A peak source/sink current, 6 ns rise/fall times with 1000 pF load, 4.5–13V supply, 12-lead QFN. Designed specifically for medical ultrasound pulser circuits.

### https://ww1.microchip.com/downloads/en/DeviceDoc/20005697A.pdf
Microchip/Supertex TC6320 dual N/P-channel MOSFET datasheet. The TC6320TG-G is the power switch used in the pic0rick three-level pulser. Key specs: 200V breakdown (N and P channel), 7Ω / 8Ω Ron, integrated gate-Zener clamps and gate-source resistors. Forms the high-voltage switch pair driven by the MD1213 to generate ±24V bipolar pulses.

### https://ww1.microchip.com/downloads/aemDocuments/documents/OTH/ProductDocuments/DataSheets/MD0100-Single-and-Dual-ChannelHigh-Voltage-Protection-TR-Switch-Data-Sheet-20005738A.pdf
Microchip/Supertex MD0100 protection T/R switch datasheet. The MD0100N8-G is used on the pic0rick main ADC board to protect the AD8331 amplifier from HV transmit pulses. Key specs: ±100V input tolerance, ±1V trip voltage, 15Ω Ron, 21pF, no power supply required. A passive self-protecting switch.

### https://recom-power.com/pdf/Econoline/R2S_R2D.pdf
RECOM R2D series isolated DC-DC converter datasheet. The R2D-0524_P is used in the pic0rick pulser board to generate ±24V from the USB 5V input. Key specs: 5V in, ±24V out, 2W total, 1kV isolation, ~85% efficiency, 42mA per output rail. Provides galvanic isolation between USB ground and the HV pulser circuit.

### https://www.nordicsemi.com/Products/nRF52832
Nordic Semiconductor nRF52832 product page. The nRF52832 is the BLE SoC used in WULPUS for wireless data transmission. Key specs: 64 MHz Cortex-M4 with FPU, 512KB Flash, 64KB RAM, BLE 5.x with 2M PHY support, +4 dBm TX power, −96 dBm RX sensitivity, 4.6–5.0 mA active current. In WULPUS, it receives ultrasound frames from the MSP430 via SPI and transmits them wirelessly at 320 kbps.

---

## WULPUS Firmware Source Files

### https://raw.githubusercontent.com/pulp-bio/wulpus/main/fw/msp430/wulpus_msp430_firmware/wulpus/us_hv_mux.h
### https://raw.githubusercontent.com/pulp-bio/wulpus/main/fw/msp430/wulpus_msp430_firmware/wulpus/us_hv_mux.c
WULPUS MSP430 firmware source for HV multiplexer control. Reveals the mux is controlled via SPI (8 MHz, MSB first) with a separate latch-enable (LE) pin. Separate functions for TX channel config (hvMuxConfTx) and RX channel config (hvMuxConfRx) write 16-bit shift registers. The LE pin transition latches the configuration into the switch outputs. Used to understand the mux architecture in Phase 1.

### https://raw.githubusercontent.com/pulp-bio/wulpus/main/fw/msp430/wulpus_msp430_firmware/wulpus/wulpus_sys.h
WULPUS MSP430 system header. Reveals: power-gated op-amp (enableOpAmpSupply / enableOpAmp functions), BLE-ready handshake GPIO, memory access macros, IIS2DH IMU presence, and protocol framing (0xFA config packet start byte). Used to understand the acquisition board architecture.

### https://raw.githubusercontent.com/pulp-bio/wulpus/main/fw/nrf52/ble_peripheral/US_probe_nRF52_firmware/us_defines.h
WULPUS nRF52 defines header. Reveals the BLE frame format: 4 SPI transfers × 201 bytes = 804 bytes per ultrasound frame. Maximum buffer: 35 frames. Device name: WULPUS_PROBE_0. Used to calculate actual BLE data throughput in Phase 1.

### https://api.semanticscholar.org/graph/v1/paper/2d6af79dbc506a32b75f0f103162a15714da79c7
Semantic Scholar entry for the WULPUS IEEE EMBC 2022 paper. Provides the abstract and confirms published specifications: <25 mW, 46×25 mm, 13g, BLE, 8-channel. Used as a secondary reference to cross-check specifications.
