# 555-Based Low-Side Buck Converter

A step-down (buck) DC-DC converter that uses a **555 timer** for PWM and an **N-channel MOSFET on the low side**, so the 555 drives the gate directly. No gate driver or level shifter is needed. Designed in **KiCad 10** as a through-hole, two-layer board for a **12 V input and about 5 V output**.

This README also records how the design arrived at this circuit.

> **Status:** PCB designed. The 555 oscillator is verified on the bench. Power-stage testing is in progress (see [Results](#results)).

---

## Repository contents

| File | Description |
|---|---|
| `Buck_converter.kicad_sch` | Schematic |
| `Buck_converter.kicad_pcb` | PCB layout: 2-layer, 83.5 mm × 48 mm, through-hole |
| `Buck_converter.kicad_pro` | KiCad project file |

<img width="392" height="236" alt="Screenshot 2026-10-23 141431" src="https://github.com/user-attachments/assets/df6825e3-7c2a-4a60-bc57-aaa64b752939" />
<img width="367" height="175" alt="Screenshot 2026-10-23 141903" src="https://github.com/user-attachments/assets/19387f20-4b6d-438a-8ea7-20f20292a022" />

---

## Circuit overview

- **Timer:** NE555 in astable mode.
- **Duty-cycle control:** potentiometer with two steering diodes, so the duty cycle (and output voltage) is adjustable while the frequency stays almost constant.
- **Switch:** IRF740 N-MOSFET on the low side, driven from the 555 output through a 100 Ω gate resistor.
- **Power stage:** 220 µH inductor, 1N5822 Schottky diode, 220 µF output capacitor.
- **Terminals:** `J2` powers the 555 side, `J3` is the power-stage input, `J1` is the output. All share one ground, so a single 12 V supply can feed both inputs.

The output is **not ground-referenced**: `VOUT+` is the input `+` rail, and `VOUT−` floats above ground.

---

## Design journey

**1. Reference circuit.** The starting point was a 555 "buck converter" schematic with a potentiometer and a low-side MOSFET. It only generates PWM and switches the supply. It has no inductor, freewheel diode or output capacitor, so adding that L-D-C stage was the first change.

**2. Keeping it simple.** A 555 boost converter reference showed a simpler idea: the 555 drives a low-side N-MOSFET directly. A standard buck uses a high-side switch, which needs a gate driver or level shifter. That added too much complexity to both the circuit and the PCB, so I looked for a simpler arrangement.

**3. Low-side buck.** The switch can stay on the low side if the inductor and diode are mirrored: the switch connects the inductor to ground, the diode runs from the switch node to the positive rail, and the output capacitor sits between the positive rail and the inductor's far end. This negative-side buck still gives `Vout ≈ D × Vin`, and the 555 drives the gate directly. The trade-off is a floating output, which is acceptable for a self-contained load.

**4. PCB.** The board uses a potentiometer-controlled 555 for adjustable output, with separate input terminals for the 555 side and the power stage.

**5. Bench.** The resistor stock had no 1 kΩ or 10 kΩ values, so the oscillator was first built as a fixed astable with 680 Ω, 27 kΩ and 1.2 nF (plus 100 nF on pin 5). It gives a clean 50% square wave, ready to be connected to the power stage.

---

## PCB vs. bench oscillator

Both are astable 555 circuits driving the same power stage. The bench version is the simplest oscillator, used to prove the 555 side first. The PCB version adds a potentiometer and steering diodes for adjustable duty cycle, and around mid-travel it behaves like the fixed 50% version.

| | PCB (potentiometer) | Bench (fixed astable) |
|---|---|---|
| Timing network | `R2` 1 kΩ, `RV1`, `D2`/`D3`, `C2` 1 nF | 680 Ω, 27 kΩ, 1.2 nF |
| Pin 5 capacitor | 100 nF | 100 nF |
| Duty cycle | Adjustable | 50% |
| Frequency | about 28 kHz (calculated, 50 kΩ pot) | 17.45 kHz (measured) |

The power stage is identical in both, so bench results carry over to the PCB.

---

## Bill of materials

| Ref | Value | Footprint |
|---|---|---|
| U1 | NE555 | DIP-8 |
| Q2 | IRF740 | TO-220 vertical |
| D1 | 1N5822 (Schottky) | DO-201AD |
| D2, D3 | Small-signal diode (e.g. 1N4148) | DO-35 |
| L1 | 220 µH | Radial 12 mm (Fastron 11P) |
| C1 | 220 µF, polarized | Radial 8 mm |
| C2 | 1 nF | Disc 5 mm |
| C3 | 100 nF | Disc 5 mm |
| C4 | 100 µF, polarized | Radial 8 mm |
| R1 | 100 Ω (gate resistor) | Axial DIN0207 |
| R2 | 1 kΩ (pin 7 pull-up) | Axial DIN0207 |
| RV1 | Trimmer potentiometer (50 kΩ assumed) | Bourns 3296W vertical |
| J1, J2, J3 | Screw terminal, 5.08 mm | Phoenix MKDS 1,5/2-5.08 |

Capacitors should be rated 25 V or more. On polarized capacitors, pin 1 is positive.

---

## Netlist

| Net | Connections |
|---|---|
| **VIN+ / VOUT+** | J3-2, D1 cathode, C1 (+), J1-1 |
| **SW** (switch node) | Q2 drain, D1 anode, L1 pin 1 |
| **VOUT−** | L1 pin 2, C1 (−), J1-2 |
| **GND** | Q2 source, J2-1, J3-1, U1 pin 1, C2, C3, C4 (−) |
| **555 supply** | J2-2, U1 pins 4 and 8, R2 pin 1, C4 (+) |
| **Gate** | U1 pin 3 → R1 → Q2 gate |
| **Timing node** | U1 pins 2 and 6, RV1 wiper, C2 |
| **Pin 7 node** | U1 pin 7, R2 pin 2, D2 anode, D3 cathode |

---

## Results

| Test | Result |
|---|---|
| 555 astable oscillator (bench) | 17.45 kHz, 57.24 µs period, 50% duty, mean 5.5 V, RMS 7.82 V |
| Full buck stage at 12 V in | *to be added* |
| Efficiency | *to be added* |
| Output ripple | *to be added* |

---

## Lessons learned

- A PWM stage alone is not a buck converter. It needs the inductor, freewheel diode and output capacitor.
- The switch position sets the drive complexity. A low-side switch removes the need for a gate driver, at the cost of a floating output.
- Check the 555 mode in a diagram: a push-button trigger means monostable, which never gives a continuous square wave.
- Parts on hand can shape the design. Without 1 kΩ or 10 kΩ resistors, 680 Ω and 27 kΩ still gave a clean 50% oscillator.
- Measured frequency (17.45 kHz) differed from the calculated value (about 22 kHz). This is within component tolerance and fine for this converter.

---

## Limitations and next steps

- Open-loop: the output varies with load and input voltage and is set by hand.
- Needs a minimum load for stable output; light loads push the output up.
- `D1` should be a Schottky or fast-recovery diode, not a standard rectifier.
- Next: finish power-stage testing, add test points and a ground pour, and consider a feedback loop.

---

## Safety

- `VOUT−` sits several volts above ground. Do not connect a grounded scope clip or grounded load to `VOUT+` or `VOUT−`.
- Start with a current-limited supply and a resistive load.

---

