# BLDC Controller v1 — BLDC Controller from Scratch

An ESP32-based three-phase brushless DC (BLDC) motor controller designed from the ground up as a self-learning project: custom 2-layer PCB, discrete MOSFET inverter, bootstrap gate drivers, per-phase current sensing, back-EMF sensing and hand-written commutation firmware.

| | |
|---|---|
| **Supply** | 14 – 25.2 V (4S – 6S Li-ion/LiPo), 30 V abs. max |
| **Power stage** | 6 × IRFS3607 (75 V / 80 A / 7.34 mΩ typ.), 3 × IR2110S gate drivers |
| **Current sensing** | 3 × 1 mΩ shunt + 3 × INA240A1 (bidirectional, ±80 A range) |
| **Sensorless** | Back-EMF comparator network (LMV324) on all three phases |
| **Controller** | ESP32-WROOM-32E-N4 (240 MHz dual-core, Wi-Fi + Bluetooth) |
| **Board** | 79.4 × 88.3 mm, 2 layers, 1.6 mm, 4 × M3 holes |

> **Status:** Board v1 is designed in KiCad and documented in a preliminary datasheet derived from the design files. It has **not been bench-characterised**, and the design review found issues that should be fixed before ordering or powering a build (see [Known Issues](#known-issues-v1)).

---

## Table of Contents

- [Goal](#goal)
- [Hardware Overview](#hardware-overview)
- [Pin Assignment](#pin-assignment)
- [Firmware Notes](#firmware-notes)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Known Issues (v1)](#known-issues-v1)
- [Roadmap](#roadmap)
- [Learning Notes](#learning-notes)
- [Authors](#authors)
- [License](#license)

---

## Goal

The aim is not only to build a working ESC but to understand how BLDC motors and ESCs work at every layer: motor physics, power stage, gate driving, timing, sensing and control. The roadmap below keeps evolving as the project progresses.

## Hardware Overview

```
Battery 4S–6S ──► VBUS ──┬──► 3-phase inverter (Q1–Q6) ──► shunts R1–R3 ──► Motor P1/P2/P3
                         └──► TPS54383 dual buck ──► 12 V (gate drivers) / 3.3 V (logic)

ESP32 ──► HIN/LIN ×3 ──► IR2110S ×3 ──► MOSFETs
ESP32 ◄── INA240 ×3 (phase current) · LMV324 (BEMF zero-cross) · VBUS divider
```

- **Inverter:** six N-channel MOSFETs in D2PAK, one IR2110S half-bridge driver per phase with bootstrap diode and 1 µF bootstrap capacitor, 22 Ω gate resistors.
- **Supplies:** a TPS54383 dual buck derives 12 V (gate drive) and 3.3 V (logic) directly from VBUS. Because 12 V is a step-down, VBUS must be about 14 V or higher.
- **Current sensing:** INA240A1 amplifiers with a mid-supply reference, so zero current reads about 1.65 V; sensitivity is 20 mV/A (roughly 40 mA per ADC LSB).
- **Back-EMF:** each phase is divided down and compared against a virtual neutral; the three comparator outputs go to ESP32 GPIOs for zero-crossing detection.
- **User interface:** UART header (J1), I/O header (J2), BOOT and RESET buttons, 3-position DIP switch, power LED and user LEDs.
- **Power terminals:** large wire pads for B+, B− and phases P1/P2/P3 (solder cables or lugs directly).
- **Not on the board:** bulk electrolytic on VBUS, fuse/pre-charge, reverse-polarity protection, surge clamp. Add these externally.

> Calculated limits only: the shunt power limit is about 55 A continuous, and sustained high current needs 2 oz copper, airflow or a heat-sink on the D2PAK tabs.

## Pin Assignment

| Function | ESP32 GPIO | Notes |
|---|---|---|
| HIN1 / LIN1 (phase A) | 2 / 15 | Strapping pins |
| HIN2 / LIN2 (phase B) | 13 / 12 | GPIO12 is a strapping pin (flash voltage) |
| HIN3 / LIN3 (phase C) | 27 / 26 | |
| Isen1 / Isen2 / Isen3 | 39 / 36 / 34 | ADC1, input-only |
| VBUS sense | 35 | ADC1, input-only |
| BEMF1 / BEMF2 / BEMF3 | 32 / 25 / 33 | GPIO25 is on ADC2: digital only when Wi-Fi is active |
| DIP 1 / 2 / 3 | 17 / 16 / 4 | Closed = GND; enable internal pull-ups |
| LED1 / LED2 / LED3 | 14 / 23 / 22 | See known issue 1 |
| UART0 RX / TX (J1) | 3 / 1 | Also the flashing port |
| J2 I/O | 19 / 21 | General purpose / I²C |
| BOOT / EN | 0 / EN | Hold BOOT, tap RESET to enter bootloader |

Phase mapping: phase 1 = P1 (U10), phase 2 = P2 (U11), phase 3 = P3 (U12). Note that BEMF1 is wired to P3's comparator and BEMF3 to P1's.

## Firmware Notes

- **Target:** ESP32-WROOM-32E, six-step (trapezoidal) sensorless commutation, with FOC as a long-term goal.
- **PWM:** generate six PWM signals on the HIN/LIN GPIOs with complementary outputs. The IR2110 has **no dead-time generator or shoot-through protection**, so firmware must insert dead time and keep all gate inputs low during reset and start-up.
- **Boot safety:** GPIO0/2/12/15 are strapping pins shared with gate-driver inputs. Keep them in a safe state during boot and verify with a scope that HIN/LIN stay low at reset.
- **ADC:** calibrate each current channel at zero current (about 1.65 V) at start-up. Use ADC1 for current and VBUS sensing.
- **Safe testing:** validate commutation logic in simulation before running it on hardware.

> TODO: add toolchain (ESP-IDF / Arduino-ESP32 / PlatformIO), build and flash commands, and a firmware architecture summary.

## Repository Structure

```
BLDC_motor_controller_from_scratch/
├── Firmware/                    # ESP32 source code (PWM, commutation, sensing)
├── Kicad_Project_Files/         # KiCad schematic, PCB and project (bldc_controller_v1)
├── Kicad_Project_Files.pretty/  # Custom footprint library for the project
├── ESC.docx                     # Written design notes / documentation
└── README.md
```

### Suggested structure as the project grows

```
├── Hardware/
│   ├── kicad/                   # Project files (one folder per board revision)
│   ├── footprints.pretty/       # Custom footprints
│   ├── gerbers/                 # Fabrication outputs per revision
│   └── bom/                     # Ordering BOM
├── Firmware/
│   ├── src/ , include/
│   └── README.md                # Build and flash instructions
├── Simulation/                  # Commutation / motor model tests (no hardware needed)
├── Docs/
│   ├── BLDC_Controller_v1_Datasheet.pdf
│   ├── images/                  # Board renders, scope captures, photos
│   └── notes/                   # Learning notes, design decisions
├── LICENSE
└── README.md
```

## Getting Started

### Hardware

1. Open the project in **KiCad** from `Kicad_Project_Files/`.
2. Add `Kicad_Project_Files.pretty` under *Preferences → Manage Footprint Libraries* if footprints are missing.
3. Apply the fixes listed under [Known Issues](#known-issues-v1), then run ERC/DRC.
4. When ordering PCBs, specify outer copper weight (2 oz recommended for the power path) and finish; the design files do not set these. Check that your fabricator supports the 0.2 mm drills in the ESP32 thermal-pad footprint, or enlarge them.

### Bring-up checklist

1. Add external bulk capacitance across B+/B− (for example 470 µF, ≥ 35 V) plus a fuse and battery switch.
2. Connect the motor **without a load on the shaft**, and a current-limited bench supply (start at about 14 – 16 V, a few amps).
3. Check TP1 = 12 V and TP2 = 3.3 V; the PWR LED should light.
4. Connect a 3.3 V USB-UART adapter to J1 (RX/TX crossed). Hold BOOT, tap RESET, flash the firmware, then reset.
5. With a scope, confirm HIN/LIN stay low at reset, then validate dead time and phase-current readings before raising the supply voltage.

> **Warning:** a power stage can fail destructively if dead time or commutation is wrong. High-current battery systems can cause fire and injury if mis-wired or unprotected.

## Known Issues (v1)

These come from the design review in the preliminary datasheet and are **not yet verified on hardware**.

| # | Severity | Issue | Suggested fix |
|---|---|---|---|
| 1 | High | Debug LEDs D9–D11 look reverse-biased (cathode to GPIO, anode via resistor to GND) | Rotate the LEDs 180° or correct the schematic polarity |
| 2 | High | BEMF dividers (33 kΩ / 10 kΩ) give about 5.9 V at 25.2 V, above the LMV324's 3.3 V supply | Change the bottom resistors to about 3.3 kΩ (about 2.3 V at 25.2 V) |
| 3 | High | Bootstrap diodes labelled "uf4007" on a 1206 footprint; BOM uses ES1J (SOD-123FL) | Change the footprint to match |
| 4 | Medium | No local bypass capacitors on INA240 (U2–U4) and LMV324 (U14) | Add 100 nF at each supply pin |
| 5 | Medium | TPS54383 input limit is 30 V; a full 6S pack is 25.2 V, with no surge protection | Add a 30 V-class TVS and/or an input filter near U6 |
| 6 | Low | Missing or mistyped schematic values (C4/C9/C14, R1–R3, R38–R40, C10, R9) | Write BOM values back to the schematic |
| 7 | Low | BEMF2 on ADC2 (unusable for analog reads with Wi-Fi); strapping pins shared with gate-driver inputs | Use as digital input, or move to an ADC1 pin |
| 8 | Low | 1 mΩ shunt gives about 40 mA/LSB; L1 package may not fit the footprint | Consider 2 mΩ if peak current is below 40 A; check the L1 fit |

## Roadmap

- [x] **1. Understanding the basics**: BLDC operation, commutation, rotating fields, inverter structures
- [x] **2. Choosing the ESC architecture**: sensored vs sensorless, open vs closed loop, trapezoidal vs FOC
- [ ] **3. Sensorless open-loop ESC**
  - [x] Dead time
  - [x] Gate driver structure and MOSFET switching
  - [x] Generating a rotating magnetic field
  - [x] PWM generation and commutation logic
  - [x] Basic ESC firmware
  - [x] Timing and code optimisation
  - [ ] Interrupt-based control
- [ ] **4. Sensorless closed-loop ESC**
  - [ ] Floating-phase sampling (comparator / ADC)
  - [ ] Back-EMF detection and zero-crossing
  - [ ] Zero-cross interrupt commutation
  - [ ] Improved startup and synchronisation
- [ ] **5. Bi-directional ESC**
- [ ] **6. Advanced motor control (long term)**
  - [ ] Field-Oriented Control: Clarke/Park transforms, SVPWM, current loops
  - [ ] Position control: encoder feedback, torque and position loops

## Learning Notes

- [Learning document (Google Docs)](https://docs.google.com/document/d/1BpeLRxeEXppZBD9K81JejwA0rQKnB6DB5yYJkA6p348/edit?usp=sharing)
- [`ESC.docx`](ESC.docx): design documentation in the repo

## Authors

Designed by **Harshal Gawande** and **Arnav Patil**.

## License

> TODO: add a `LICENSE` file (MIT is common for hobby firmware; consider CERN-OHL for the hardware files).
