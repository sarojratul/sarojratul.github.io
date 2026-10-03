---
layout: series-post
title: "Part 2: Simulation and Virtual Prototyping"
date: 2026-09-05 12:00:00 -0600
series: turn-signal
part: 2
covers: "29 August to 5 September 2026"
excerpt: "LTspice for the analog circuit, Wokwi for the state machine, Tinkercad for the breadboard layout, and a mid-stream redesign that reused the work already done."
permalink: /turn-signal/part-2-simulation/
redirect_from:
  - /2026/08/29/simulation-and-virtual-prototyping.html
---

Before any breadboard work, each part of the design went through a simulator: LTspice for the electronics, Wokwi for the firmware and Tinkercad for the physical layout. Each one proves something different, and each has a blind spot that later parts of the log run into.

<details class="toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## LTspice: LED strings and MOSFET switch (Stage A2)

<p class="entry-date">29 August 2026</p>

**Objective.** Prove the LED strings, the 33 Ω resistor value and the MOSFET switch before buying anything.

![LTspice A2 schematic and operating point for one 2-LED string](/images/ltspice_op_string.jpg)

*One string: two AMBER diodes and 33 Ω on a 4.8 V source. The operating point reports I(D1) = I(D2) = I(R1) = 0.0247755 A, the 24.78 mA the design is built on.*
{: .caption}

**Step 1, LED model.** The Lumex part has no SPICE model, so I fitted one to the datasheet's V<sub>f</sub> curve:

```spice
.model AMBER D(Is=2e-18 N=2 Rs=3 Cjo=30p)
```

It gives **V<sub>f</sub> = 1.99 V** per diode at the operating point, against 2.0 V typical at 20 mA in the datasheet. Nominal string current at 4.8 V through 33 Ω: **24.78 mA**.

**Step 2, DC sweep from 4.0 V to 5.0 V**, to see what a discharging pack does:

| Supply | String current |
|---|---|
| 4.00 V | 7.45 mA |
| 4.50 V | 17.94 mA |
| 4.80 V | 24.78 mA |
| 5.00 V | 29.45 mA |

![DC sweep of the supply from 4.0 V to 5.0 V](/images/ltspice_dc_sweep.jpg)

*`.dc V1 4.0 5.0 0.05`, plotting the voltage across the 33 Ω resistor. Divide by 33 for string current.*
{: .caption}

![Exported sweep data, V1 against I(R1)](/images/ltspice_sweep_data.jpg)

*The same run exported as text, the source of the exact values in the table. 4.80 V gives 2.477554e-02 A.*
{: .caption}

The curve is monotonic and stays inside the LED rating on a fresh pack. It is also steep: LED current is exponential in supply voltage, so the arrow will dim visibly as the pack drains. That is acceptable for the LEDs and not for the logic, which is the case for the buck-boost.

**Step 3, a full segment with the real MOSFET:** three strings, an AO3400A low-side switch (a `.SUBCKT` model), 3.3 V on the gate. Operating point:

- Segment current **74.16 mA** (24.72 mA per string)
- **V<sub>DS</sub> = 2.38 mV**: the MOSFET is effectively a wire
- MOSFET dissipation **≈ 176 µW**
- Gate sweep: drain current flat at about 74 mA from about **1.4 V** up to 3.3 V
- Transient 10–90% fall time **≈ 100 ns**, irrelevant at a 120 ms step

![Three-string segment with the AO3400A and its gate network, operating point](/images/ltspice_segment_op.jpg)

*Segment B/C in full: three strings, three 33 Ω resistors, the AO3400A, and the 220 Ω / 10 kΩ gate network. I(V1) = 0.0741613 A is the segment current, V(n002) = 0.00238018 V is V<sub>DS</sub>, and I(R4) = I(R5) = 0.000322896 A is the 322.9 µA gate current measured on the breadboard nine days later ([Part 3](/turn-signal/part-3-soldering-and-panel-build/#test-1-gate-networks-pass)).*
{: .caption}

![Gate voltage swept from 0 to 3.3 V against drain current](/images/ltspice_gate_sweep.jpg)

*Drain current is flat from about 1.4 V to 3.3 V, so the ESP32-C3's 3.3 V output sits well inside the fully-on region. This is the argument for a logic-level MOSFET.*
{: .caption}

> **A debugging constant.** With the gate high, V<sub>DS</sub> should be millivolts (2.38 mV simulated). Near the full rail means the MOSFET is off. One probe answers the question.

**Not simulated:** segments A and D (five strings) are hand-calculated at 123.9 mA and 0.59 W. Same topology with two more strings in parallel. Segments B and C are exactly the simulated circuit.

### LTspice notes

- A bare `.SUBCKT` with no symbol does not appear in the component browser. Place a generic NMOS, open MOSFET Properties and type the subcircuit name; change the instance prefix from `M` to `X`.
- **Near-zero MOSFET current in `.op` with everything else normal means drain or source is not actually connected.** Check for the junction dot.
- In a `.dc` sweep, hover a wire for `Ix(<refdes>:<pinname>)` to plot current through one pin.
- The form-based PULSE editor is less error-prone than typing the string.
- For precise timing, export `.tran` data as text and interpolate at the threshold rather than reading a zoomed plot.

## Mid-stream redesign

<p class="entry-date">31 August 2026</p>

The two-arrow layout was replaced by the shared double-headed arrow ([reasons in Part 1](/turn-signal/part-1-design/#display-layout-one-double-headed-arrow)) after the old design had been simulated and documented. **Because the new LED counts are whole multiples of the simulated string, the simulation still applied:** segment B/C is exactly the circuit run two days earlier.

The documents needed the real work. Segment counts, MOSFET counts, "six output-capable pins", a removed auto-cancel behaviour and a shared BOM table were spread across both files. Stage B, the gate checklists and the firmware section were rewritten. One line still escaped: B5 step 4 said "wire all 8 string anodes" instead of 16, found a week later.

> **Lesson:** before a rewrite pass, search the document for every stale term and number (segment counts, pin counts, old behaviour names). Stale text hides in parts lists and pass criteria, not in headings.

## Wokwi: state machine (Stage A3)

<p class="entry-date">1 to 3 September 2026</p>

**Objective.** Prove the firmware logic. Wokwi does not model current, so each segment is a single LED.

![Wokwi simulation with the ESP32-C3, four LEDs and two buttons](/images/wokwi_sim.jpg)

*The A3 rig: one LED per segment, two buttons, and the serial log showing a LEFT hold, release with 4 trailing cycles counting down, OFF, then a hold in the other direction.*
{: .caption}

Design points:

1. **A radio seam.** `commandedState()` and `linkAlive()` are the only functions that know where commands come from. In Wokwi they read buttons; on hardware they read `g_state` and `g_lastRx` from the ESP-NOW callback. Swapping simulation for radio costs two one-line function bodies.
2. **Direction-aware sweep** from a 4-bit segment mask, `segmentMaskFor(dir, msIntoCycle)`.
3. **Hold plus trailing cycles,** with `cycleStartMs` latched on each new press.
4. `isNewPress = (cmd != activeDir) || (!wasHeld)` makes any press cancel any state.

### Bug 1: free-running cycle phase

The first version derived animation phase directly from `millis()`, so a press could land mid-sweep and the trailing count was off by a fraction of a cycle. Latching `cycleStartMs` on the press fixed both.

### Bug 2: LEDs wired backwards

Code, buttons and serial output were all correct and nothing lit. Four LEDs failing together is implausible, so they were reversed, and they were.

> **Lesson:** when nothing works, use what *does* work to narrow the fault. Working serial output, buttons and timing each rule out a category of cause, leaving the only unverified part. This applied again a week later ([Part 4](/turn-signal/part-4-floating-ground/)).

### Open items

- **`HOLD_MS`:** the full arrow is lit for only 120 ms of each 800 ms cycle. Whether that reads as urgent or flickery to a driver 30 m back cannot be settled at a desk.

  > **Update, 20 September:** partly settled. Tuned on the real panel to a 788 ms hold in a 1688 ms cycle ([Part 6](/turn-signal/part-6-esp32-drives-the-panel/#sweep-timing-tuned-by-eye-verified-on-a-scope)). The outdoor night check is still to come.

- **`millis()` rollover** at 49.7 days is accepted; the light is powered off between rides.
- **Pins:** Wokwi used GPIO 2/3/4/5; the hardware pin set was locked as GPIO 2/4/5/6 and recorded in three places.

  > **Update, 20 September:** GPIO 2 is a strapping pin, so the final set is **GPIO 3/4/5/6** ([Part 6](/turn-signal/part-6-esp32-drives-the-panel/#gpio-2-is-a-strapping-pin)).

## Tinkercad: breadboard layout rehearsal

<p class="entry-date">5 September 2026</p>

The next step was the breadboard: 32 LEDs, 16 resistors and 4 MOSFET adapters on one board. If it failed, I would not be able to tell a wrong circuit from a leg in the wrong hole. Wokwi has no current model or MOSFET, and LTspice says nothing about layout, so I rehearsed the build in **Tinkercad Circuits**, which simulates a physical breadboard.

- Tinkercad **does** have NMOSFET parts. I initially rebuilt with BJTs, which wasted an hour because the 220 Ω / 10 kΩ gate network does not transfer to a BJT.
- Fixed batteries are only 9 V, 3 V and 1.5 V, but the adjustable **custom power supply** works: 4.8 V at 1 A for the panel and 3.3 V for gate injection, negatives on one ground rail.
- The test signal was injected at the **input side of the 220 Ω**, where the GPIO will attach, so both resistors are exercised. An unflashed microcontroller on that node is high-impedance, so the same method works on the real board.

**Result: all four segments passed.** Each lights alone with 3.3 V on its gate and is dark otherwise.

![Tinkercad simulation of the full four-segment panel](/images/tinkercad_full.jpg)

*The full rehearsal: 16 two-LED strings, 16 resistors, four MOSFETs, the 470 µF bulk capacitor, a 4.8 V panel supply and a separate gate-injection supply.*
{: .caption}

One number does **not** transfer: Tinkercad shows 91.6 mA for a five-string arrowhead (18.3 mA per string) because it models a red LED at V<sub>f</sub> ≈ 2.1 V. With amber at 1.99 V the expected figure is about 123.9 mA. I recorded that before going to the bench so it would not be chased later.

> **Scope of the result:** Tinkercad proves topology and wiring plan. It proves nothing about real components, real currents, thermals or whether a stranded wire makes contact. Parts 3 and 4 measure how much that leaves out.

**Next:** [Part 3](/turn-signal/part-3-soldering-and-panel-build/), real parts on a real board.
