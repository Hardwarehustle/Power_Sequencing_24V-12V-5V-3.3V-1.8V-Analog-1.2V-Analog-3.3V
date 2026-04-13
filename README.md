# ⚡ Power Sequencing Design — Multi-Rail DC Power Distribution Board

## 📌 Project Overview

This project presents a **complete multi-rail power sequencing board** that converts **250V AC mains** down to a cascaded hierarchy of DC voltage rails — from 24V down to 1.2V — for powering mixed-signal embedded systems. The board supports **SoC/MCU cores, analog front-ends, relay/driver circuits**, and precision ADC/DAC references, all with controlled power-up sequencing via MCU GPIO enable pins.

The design covers:
- Full **schematic capture** in EasyEDA
- **PCB layout** with proper power and ground plane separation
- **Component selection** rationale with IC datasheets
- **MCU-controlled enable sequencing** via GPIO + P-MOSFET
- **Verification & Validation plan** (LTSpice simulation + physical testing)

---

## 🖼️ Project Visuals

### Schematic
![Schematic](SCH.jpg)

### PCB Layout
![PCB Layout](PCB_Layout.jpg)

### 3D Board Top View
![3D Top View](top_view.jpg)

### 3D Board Bottom View
![3D Bottom View](top_view.jpg)
---

## 🏗️ Power Architecture

The board implements a **top-down cascaded power tree**:

```
250V AC
   └─► 24V DC  (AC-DC SMPS — Mean Well LRS-35-24)
           ├─► 12V DC  (Buck Converter — LM2675-12)
           └─► 5V DC   (Buck Converter — LM7805 / LM2675-5.0)
                   ├─► 3.3V Digital   (LDO — TLV1117-33)
                   ├─► 1.8V Digital   (LDO — LM1117MPX-1.8)
                   ├─► Analog 1.2V    (Precision LDO — TPS7A02-1.2)
                   └─► Analog 3.3V    (Low-Noise LDO — TPS7A4700)
```

**Power-up sequence:**  
`24V → 12V → 5V → 3.3V → 1.8V → Analog 1.2V → Analog 3.3V`

---

## 📊 Load Requirements

| Voltage Rail | Load Current | Purpose |
|:---:|:---:|:---|
| 12V | 300 mA | Relay / Driver Circuits |
| 5V | 5 mA | MCU Supply, LDO Inputs |
| 3.3V (Digital) | 10 mA | MCU I/O, Comm. Interfaces |
| 1.8V | 100 mA | SoC / MCU Core Voltage |
| Analog 1.2V | 150 mA | ADC/DAC References |
| Analog 3.3V | 100 mA | Analog Sensors / Op-Amps |

---

## 🔩 Component Selection

| Voltage Domain | Load | Type | IC | Why Selected |
|:---|:---:|:---:|:---|:---|
| 24V (AC-DC) | 2.2A | SMPS | Mean Well LRS-35-24 | Reliable, compact, 35W rated |
| 12V | 300 mA | Buck Reg | LM2675-12 | High-efficiency, 1A switch, 52 kHz, external EN |
| 5V | 5 mA | Buck Reg | LM7805 / LM2675-5.0 | Simple linear/buck for low current |
| 3.3V Digital | 10 mA | LDO | TLV1117-33 | Compact, stable, low dropout |
| 1.8V Digital | 100 mA | LDO | LM1117MPX-1.8 | Popular, stable for digital rails |
| Analog 1.2V | 150 mA | Low-Noise LDO | TPS7A02-1.2 | Ultralow noise, high PSRR, low IQ |
| Analog 3.3V | 100 mA | Low-Noise LDO | TPS7A4700 | Excellent PSRR, high-performance analog |

---

## 🔋 Role of Each Power Domain

### 24V Rail
- Acts as the **intermediate supply** from AC mains via SMPS
- Feeds the first buck stage (12V converter)

### 12V Rail
- Supplies **high-current peripherals**: relays, solenoids, motor drivers
- Used in external interface circuits requiring higher voltage

### 5V Rail
- Functions as the **logic-level domain** and input to downstream LDO regulators
- Powers LEDs, simple logic blocks, and MCU supply

### 3.3V Digital Rail
- Supplies **MCU I/O**, communication interfaces (UART, I2C, SPI)
- Supports low-power digital peripherals

### 1.8V Digital Rail
- Used as **core supply** for MCUs, SoCs, and memory devices
- Essential for low-voltage digital logic

### Analog 1.2V Rail
- Provides a **low-noise reference voltage** for ADC/DAC references
- Powers RF circuits and sensitive analog sections

### Analog 3.3V Rail
- Delivers a **clean analog power rail** for Op-Amps and precision analog components
- Isolated from the digital 3.3V to prevent noise coupling

---

## 🎛️ MCU-Based Power Control

Power sequencing is controlled by an MCU via **GPIO-driven enable pins** on each regulator stage:

| Regulator / IC | Control Method |
|:---|:---|
| TPSM33625 | EN pin ← MCU GPIO |
| LMR16006 | SHDN# pin ← MCU GPIO |
| LP5907 (U5, U7) | EN pin ← MCU GPIO |
| LP2985 | EN pin ← MCU GPIO |
| LM1117MPX-1.8 | GPIO → Gate of P-MOSFET (SI2301) → LDO VIN |

> **Note:** A **P-MOSFET (Q1 - SI2301)** is used for the LM1117 stage. The MCU GPIO drives the gate, and a pull-up resistor (R8) ensures the rail defaults **OFF** on system reset — providing safe startup behavior.

---

## 📐 PCB Design Highlights

- **Tool:** EasyEDA (Schematic + Layout)
- **Layer stack:** 2-layer PCB (Red = Top copper, Blue = Bottom copper)
- **Connector interface:** JST B2B-XH-A-M style connectors for each output rail
- **24V input:** XT30 male power connector (high-current barrel/plug)
- **Output headers:** Labeled `+12V`, `+5V`, `+3V3`, `+1.8V`, `A3V3`, `A1.2V`
- **Decoupling caps:** Placed close to each IC for EMI suppression
- **Analog/Digital separation:** Analog LDO outputs (TPS7A02, TPS7A4700) routed separately from digital switching rails

---

## 🧪 Verification & Validation Plan

### Phase 1 — LTSpice Simulation

| Simulation Type | Goal |
|:---|:---|
| DC Operating Point | Verify static output voltages at all rails |
| Transient Analysis | Observe power-up/down voltage waveforms |
| Load Transient | Test voltage stability under sudden load steps |
| Ripple & Noise | Measure peak-to-peak and RMS output ripple |
| Efficiency & Stability | Bode plots for phase margin and gain margin |

### Phase 2 — Physical Board Bring-Up

1. **Component Inspection** — Verify part numbers, orientation, polarity of capacitors/diodes
2. **Continuity Check** — Multimeter check on all power/ground nets
3. **No-Load Voltage Measurement** — Confirm all rails at correct voltages
4. **Load Regulation Test** — Apply rated loads (e.g., 300 mA at 12V) and verify regulation
5. **Power Sequencing Verification** — Oscilloscope capture of `24V → 12V → 5V → 3.3V → 1.8V → A1.2V → A3.3V`
6. **Thermal Check** — IR thermometer / thermal camera for hotspot detection
7. **EMI/EMC Screening** — Preliminary conducted/radiated emissions check

---

## 📁 Repository Structure

```
Power-Sequencing-Board/
├── Schematic/
│   └── SCH.jpg                     # Full schematic capture
├── PCB/
│   ├── PCB_Layout.jpg              # 2D layout (Altium/EasyEDA export)
│   └── top_view.jpg                # 3D rendered board view
├── Docs/
│   └── Assignment-upliance.ai-PPT.pptx   # Design presentation
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites
- **EasyEDA** (free, browser-based EDA tool) or compatible Gerber viewer
- **LTSpice** for simulation verification
- **Multimeter + Oscilloscope** for physical validation

### Running the Design
1. Open the `.json` or `.sch` source files in EasyEDA
2. Review schematic for each power stage
3. Export Gerbers from the PCB editor for fabrication
4. Program MCU with GPIO sequencing firmware for controlled power-on

---

## 📄 License

This project is open for educational and personal use.  
© 2025 Janardhan BV — All rights reserved.

---

## 🙋 Author

**Janardhan BV**  
Embedded Hardware Engineer | PCB Design | Power Electronics  
📍 Bengaluru, India

---
*Designed as part of the upliance.ai Hardware Engineering Assignment*
