# EMC Test – Conducted EMI Simulation of an Offline Flyback Converter

LTspice model for checking the **conducted emissions** of a single-switch flyback converter, measured through a standard **LISN** (Line Impedance Stabilization Network) and an input EMI filter.

![Schematic](docs/schematic.png)

## Circuit Overview

The signal path runs left to right: **source → LISN → EMI filter → flyback converter → output**.

### 1. Source
- **V2**: 311 V DC, the peak of a rectified 220 Vac mains.

### 2. LISN 
Sits between the source and the device under test on both lines (L/N):

| Part | Value | Role |
|------|-------|------|
| L8, L9 | 50 µH | Line inductors, block noise from going back to the source |
| C7, C8 + R8, R9 | 8 µF + 5 Ω | Source-side decoupling to the return |
| C9, C10 | 250 nF | Couple the HF noise into the measurement resistors |
| R6, R7 | 50 Ω | Receiver impedance; the noise is read here |
| R10 | 0.1 mΩ | Ties the LISN midpoint to earth |
| R12 | 10 MΩ | Source return → earth (DC reference only, negligible at HF) |

**Measurement nodes:** `V1` (line) and `V2` (neutral). Plot `V(V1)` / `V(V2)` and run an FFT to get the conducted emission spectrum.

### 3. EMI Input Filter
| Part | Value | Role |
|------|-------|------|
| R2, R11 | 0.1 mΩ | Current-sense / probe points |
| L10, L11 (`K1 L10 L11 0.99`) | 10 mH | Common-mode choke |
| C11, C12 | 1 µF | X capacitors (differential mode) |
| C13, C14 | 470 nF | Y capacitors to earth (common mode) |

### 4. Flyback Converter
| Part | Value | Role |
|------|-------|------|
| M1 | STP8NM60 | 600 V primary MOSFET |
| V1 | `PULSE(0 12 0 100p 100p 10u 20u)` | Gate drive: 12 V, **50 kHz, 50 % duty** |
| R4, L7 | 2.2 Ω, 1 pH | Gate resistor and gate-loop inductance |
| L1 | 900 µH | Transformer primary |
| L2 | 50 µH | Leakage inductance (uncoupled) |
| L3 | 100 µH | Transformer secondary (`K L1 L3 0.99`, 3:1 turns ratio) |
| R5, C3, D3 | 4.7 kΩ, 100 pF, RRD20TJ10S | RCD snubber/clamp |
| L5 | 145 nH (Würth 742792114) | Ferrite bead in the clamp path |

**Parasitic coupling paths (the actual sources of common-mode noise):**
| Part | Value | Models |
|------|-------|--------|
| C5 | 5 pF | MOSFET drain → heatsink/earth |
| C6 | 7 pF | Primary → secondary winding capacitance |
| C4 | 1 nF | Y capacitor, secondary return → primary return |
| R13 | 10 MΩ | Bleeder across C4, gives the isolated secondary a DC reference |

### 5. Output Stage
| Part | Value | Role |
|------|-------|------|
| D1 | 1N5819 | Schottky output rectifier |
| C1 | 470 µF (Würth WCAP-ATUL) | Output bulk capacitor |
| L4, C2 | 1 mH, 100 µF (Würth WCAP-ASLI) | Second-stage LC filter |
| R1 | 2 Ω | Load |
| D2 + R3 | EDZV24B, 1 Ω | 24 V zener clamp / overvoltage protection |
