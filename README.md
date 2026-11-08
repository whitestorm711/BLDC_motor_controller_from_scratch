# BLDC Motor Controller from Scratch

A three-phase brushless DC (BLDC) electronic speed controller (ESC) designed and built from the ground up as a self-learning project: custom PCB, discrete MOSFET power stage, and hand-written commutation firmware. No ESC reference designs or off-the-shelf motor-control libraries.

The goal is not just a working controller, but a clear understanding of every layer: motor physics, power stage, gate driving, timing, and control.

> **Status:** Sensorless open-loop trapezoidal control works and spins a BLDC motor. Closed-loop (back-EMF zero-crossing) commutation is the current focus.

---

## Table of Contents

- [Features](#features)
- [Hardware Overview](#hardware-overview)
- [Firmware Overview](#firmware-overview)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Roadmap](#roadmap)
- [Learning Notes](#learning-notes)
- [Known Issues](#known-issues)
- [License](#license)

---

## Features

- Three half-bridge power stage built from discrete N-channel MOSFETs
- Bootstrap-powered high-side gate drivers
- Trapezoidal (six-step) commutation
- Sensorless operation (back-EMF sensing planned via comparator/ADC)
- Full KiCad schematic and PCB source, including the custom footprint library
- Written documentation of the concepts learned along the way

## Hardware Overview

| Block | Details |
|---|---|
| Power stage | 3 half-bridges, discrete MOSFETs |
| Gate drivers | High/low-side drivers with bootstrap circuits |
| Current sensing | Shunt-based current sense amplifier |
| Power supply | Buck regulators to derive logic rails from the battery bus |
| Design tool | KiCad |

> TODO: fill in exact part numbers, supply voltage range, and max current once the board revision is final.

Key design topics covered: dead time, bootstrap capacitor sizing, MOSFET switching behaviour, and phase-node layout.

## Firmware Overview

- **MCU:** STM32F411 (STM32 HAL / bare-metal, update as appropriate)
- **Control method:** six-step trapezoidal commutation with PWM
- **Current mode:** sensorless open-loop; closed-loop zero-cross detection in progress
- **Timing:** hardware timers generate complementary PWM with dead time; commutation steps advance on timer/interrupt events

> TODO: add the toolchain (STM32CubeIDE / PlatformIO / Makefile) and the pin mapping table (phase PWM outputs, enable pins, ADC/comparator inputs).

## Repository Structure

```
BLDC_motor_controller_from_scratch/
├── Firmware/                    # MCU source code (commutation, PWM, control logic)
├── Kicad_Project_Files/         # KiCad schematic and PCB project
├── Kicad_Project_Files.pretty/  # Custom footprint library used by the project
├── ESC.docx                     # Written notes / design documentation
└── README.md                    # You are here
```

### Suggested structure as the project grows

```
├── Hardware/
│   ├── kicad/                   # Project files
│   ├── footprints.pretty/       # Custom footprints
│   ├── gerbers/                 # Fabrication outputs per revision
│   └── bom/                     # BOM and assembly files
├── Firmware/
│   ├── Core/ (or src/, include/)
│   └── README.md                # Build/flash instructions
├── Simulation/                  # Commutation / motor model tests (no hardware needed)
├── Docs/
│   ├── images/                  # Schematic PDFs, scope captures, board photos
│   └── notes/                   # Learning notes, design decisions
├── LICENSE
└── README.md
```

## Getting Started

### Hardware

1. Open the project in **KiCad** from `Kicad_Project_Files/`.
2. Add the footprint library `Kicad_Project_Files.pretty` under *Preferences → Manage Footprint Libraries* if footprints show as missing.
3. Review the schematic and run ERC/DRC before ordering boards.

### Firmware

1. Open the `Firmware/` project in your STM32 toolchain.
2. Build and flash through an ST-Link (SWD).
3. **Safety:** test first with a current-limited bench supply and no propeller or load attached.

> Warning: Power stages can fail destructively when commutation or dead-time logic is wrong. Validate firmware on a simulator or with strict current limiting before connecting a full battery.

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

## Known Issues

- Open-loop commutation spins the motor but not smoothly; commutation timing is being refined.
- Closed-loop startup (open-loop ramp, then handover to back-EMF) is not implemented yet.

## License

> TODO: add a `LICENSE` file (MIT is a common choice for hobby hardware/firmware; consider CERN-OHL for the hardware files).
