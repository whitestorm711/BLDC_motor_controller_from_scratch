# ESC-from-Scratch

Building a BLDC Electronic Speed Controller (ESC) completely from scratch while learning every concept from the basics.

---

# Goal

The goal of this project is not only to build a working ESC, but also to deeply understand how BLDC motors and ESCs work internally — including the hardware, control systems, and embedded programming involved.

This roadmap will continue evolving as I learn and progress through the project.

---
Document of what i have learned - https://docs.google.com/document/d/1BpeLRxeEXppZBD9K81JejwA0rQKnB6DB5yYJkA6p348/edit?usp=sharing
Will be putting a Check mark on the roadmap goals i have already achieved 

# Roadmap

## 1. Understanding the Basics☑️
- Learn how BLDC motors work
- Learn how ESCs control BLDC motors
- Understand commutation and rotating magnetic fields
- Study basic inverter structures

--

## 2. Choosing the ESC Architecture ☑️
- Decide the type of ESC to build
- Compare:
  - Sensored vs Sensorless
  - Open-loop vs Closed-loop
  - Trapezoidal vs FOC control

--

## 3. Building a Sensorless Open-Loop ESC 
### Learning Goals
- Understanding dead time ☑️
- Gate driver structure and MOSFET switching☑️
- Generating a rotating magnetic field☑️
- PWM generation and commutation logic☑️

### Software Goals
- Writing basic ESC firmware☑️
- Optimizing timing and code execution☑️
- Learning interrupt-based control

### SENSORLESS OPEN-LOOP ESC COMPLETED

--

## 4. Building a Sensorless Closed-Loop ESC
### Learning Goals
- Sampling the floating phase using:
  - Comparator
  - ADC
- Back-EMF detection and zero-crossing

### Control Goals
- Using comparator/ADC readings for commutation
- Triggering interrupts based on zero-cross events
- Improving startup and synchronization


--

## 5. Building a Bi-Directional ESC


--

## 6. Advanced Motor Control (Long-Term Goal)

### Field-Oriented Control (FOC)
- Clarke and Park transforms
- Space Vector PWM (SVPWM)
- Current control loops

### Position Control
- Encoder/sensor feedback
- Torque and position control
- Advanced closed-loop control systems
-

