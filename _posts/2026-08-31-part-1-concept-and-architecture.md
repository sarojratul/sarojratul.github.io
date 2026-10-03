---
layout: series-post
title: "Part 1: Concept, Specifications and Architecture"
date: 2026-08-31 12:00:00 -0600
series: turn-signal
part: 1
covers: "Late August 2026"
excerpt: "One double-headed arrow instead of two, a centre-out sweep, the system architecture, the power budget, and how each part was checked against its datasheet before ordering."
permalink: /turn-signal/part-1-design/
redirect_from:
  - /2026/08/28/concept-specifications-architecture.html
---

Everything in this part was settled on paper before any hardware existed. Some numbers changed once real parts were on the bench; the original figures stay, and each change is marked **Update** with a link to where it happened.

<details class="toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## Display layout: one double-headed arrow

<p class="entry-date">31 August 2026</p>

The first architecture used **two separate arrows**, one per backpack strap: three segments each, six MOSFETs, six GPIOs. I replaced it on 31 August, after the first simulations had already run ([effect on the simulation work](/turn-signal/part-2-simulation/#mid-stream-redesign)):

- Two strap-mounted arrows read as a costume rather than safety equipment.
- A single horizontal **double-headed arrow** (`◄===►`) shares its body between both directions, which means fewer segments, MOSFETs, gate networks and GPIOs.

| Segment | Position | LEDs | 2-LED strings |
|---|---|---|---|
| **A** | Left arrowhead | 10 | 5 |
| **B** | Body, left of centre | 6 | 3 |
| **C** | Body, right of centre | 6 | 3 |
| **D** | Right arrowhead | 10 | 5 |

**Totals: 32 LEDs, 16 strings, 4 segments, 4 MOSFETs, 4 GPIOs.**

![Amber LEDs arranged into the arrow shape on a breadboard](/images/arrow_layout_leds.jpg)

*Working out the physical arrow shape before the wiring plan. The visual grouping (an arrow) and the electrical grouping (2-LED strings) are separate problems; this photo is the visual one.*
{: .caption}

![Double-headed arrow split into four coloured segments, A B C D](/images/arrow_segments.svg)

*The final layout. B and C sit either side of centre and serve both directions. Currents are the simulated and hand-calculated figures from the power budget below.*
{: .caption}

### Why the LED count went from 7/4/4/7 to 10/6/6/10

Every string is **two LEDs in series**, fixed by the supply: four in series needs about 8 V and the pack provides 4.8 V. An odd LED count leaves a stray LED, and segments that are not whole multiples of the simulated string do not reuse the simulation. Moving to 10/6/6/10:

- makes every segment a whole number of identical strings, so the existing LTspice results still apply;
- uses one resistor value, 33 Ω, sixteen times, with no special cases in the BOM;
- adds LEDs, which only helps visibility.

## Sweep pattern

The arrow does not blink. It **sweeps from the centre outward**, after the 1965 Ford Thunderbird sequential taillights designed in EE 232:

- **LEFT:** C, then C+B, then C+B+A
- **RIGHT:** B, then B+C, then B+C+D

The motion carries the direction. Initial timing: `STEP_MS = 120`, `CYCLE_MS = 800`.

> **Update, 20 September:** too fast on the real panel. After tuning by eye and checking on a scope the final values are `STEP_MS = 450`, `HOLD_MS = 788`, `CYCLE_MS = 1688`. See [Part 6](/turn-signal/part-6-esp32-drives-the-panel/#sweep-timing-tuned-by-eye-verified-on-a-scope).

## Architecture

| | Handlebar transmitter | Backpack receiver |
|---|---|---|
| **Input** | 2 × tactile switch (left / right) | ESP-NOW packets, 2.4 GHz |
| **Controller** | ESP32-C3 | ESP32-C3, GPIO 3 / 4 / 5 / 6 (originally 2 / 4 / 5 / 6, see update below) |
| **Drive stage** | none | 4 × (220 Ω series gate resistor, 10 kΩ pulldown) into 4 × AO3400A N-MOSFET, low side |
| **Load** | 1 status LED + 1 kΩ | Segments A(10) B(6) C(6) D(10): 32 LEDs in 16 strings of 2 + 33 Ω |
| **Power** | 3 × AAA NiMH 1100 mAh, TPS63070, 3.3 V | 4 × AA NiMH 2800 mAh (4.8 V): raw to the LEDs, TPS63070 to 3.3 V for logic |
| **Role** | Stateless: sends held / not-held levels on a heartbeat | Holds all state and behaviour |

![Block diagram: handlebar transmitter, ESP-NOW link, backpack receiver](/images/block_diagram.svg)

*The system. The transmitter is a switch, a radio and a regulator; every decision is made on the receiver. The 322.9 µA and 2.38 mV figures are LTspice predictions that Parts 3 and 4 check on the bench.*
{: .caption}

**The transmitter is stateless by design.** It reports "left held / not held" on a heartbeat and nothing else. Sweep order, timing, trailing cycles and cancellation all live on the receiver, which also tracks link health (`linkAlive()`) and can fail safe on its own if packets stop.

> **Update, 20 September:** GPIO 2 is an ESP32-C3 strapping pin that Espressif recommends pulling *up*, while segment A's gate network pulls it down through 10 kΩ. Segment A moved to GPIO 3 before any wiring. See [Part 6](/turn-signal/part-6-esp32-drives-the-panel/#gpio-2-is-a-strapping-pin). The block diagram shows the corrected pins.

### Activation model

- **While a button is held,** that direction's sweep repeats.
- **On release,** the sweep runs `TRAILING_CYCLES` more cycles, then goes dark. No off button, no timer to forget.
- **Any press in any state cancels immediately** and starts a fresh hold in the pressed direction.

`TRAILING_CYCLES` started at 2 and was tuned to **4** in simulation, because two cycles ended before a turn was complete.

## Power budget

The LED panel runs **directly from the 4.8 V NiMH pack**, with no regulator in the LED path. Each string's series resistor sets its current, so a sagging pack only dims the LEDs, and a converter in series with about 400 mA of LEDs would only add loss. The logic does need a stable rail, so it gets a TPS63070 buck-boost, which can step a fresh 5.35 V pack down and a depleted 3.0 V pack up to 3.3 V.

At the time of writing the TPS63070 was purchased but reserved for work-plan step **B10**, where it is characterised alone across the full discharge range before joining the circuit. Bench testing used the raw pack, with 3.3 V gate drive from the Analog Discovery 2's waveform generator. (**Update:** from 20 September the rail came from a bench supply, see [Part 5](/turn-signal/part-5-full-panel/#bench-supply-setup).)

| Case | Current | Power |
|---|---|---|
| One string (2 LEDs + 33 Ω at 4.8 V) | 24.78 mA | n/a |
| Segment B or C (3 strings) | 74.3 mA | 0.36 W |
| Segment A or D (5 strings) | 123.9 mA | 0.59 W |
| Peak during sweep (3 segments lit) | 272.6 mA | 1.31 W |
| All four lit (bench and fuse sizing) | 396.5 mA | 1.90 W |
| MOSFET dissipation, worst case (A or D) | n/a | ≈ 368 µW |

> **Update, 20 September:** on the bench each LED dropped about 2.08 V rather than the modelled 1.99 V, so every segment draws less than this table. Measured at 4.8 V: A or D about 80 to 89 mA, B or C 53 mA, all four 265 mA. The table stands as the ceiling from the idealised model. See [Part 5](/turn-signal/part-5-full-panel/).

**Runtime on 2800 mAh:** about **9.12 h** continuous with everything lit, and about **58.3 h** at a generic 5% duty cycle. The duty figure is an assumption, not derived from the hold-and-trailing behaviour, and is flagged open in the work plan; it clears the requirement by a wide margin.

## Component selection

<p class="entry-date">Up to 28 August 2026</p>

I used an AI assistant to draft the bill of materials and the build sequence, then treated every line as a claim to verify against the datasheet. The aim was practice at owning a design ahead of CME 331 and EE 321, which are less guided than second-year labs. The loop:

1. AI drafts a BOM and a sequence of steps.
2. I read the datasheets.
3. I mark what is wrong, obsolete or invented.
4. The BOM is revised with that evidence, and the loop repeats per component.
5. **Nothing is ordered until every line is verified.**

| Proposed | Finding | Ordered |
|---|---|---|
| A microcontroller with no integrated radio | A router-free peer-to-peer link on a bike needs ESP-NOW, which needs an ESP chip | **ESP32-C3** (XIAO / SuperMini) |
| **IRLB8721** MOSFET | Works, but a TO-220 part is oversized for 124 mA. AO3400A: V<sub>GS(th)</sub> = 1.05 V typ, R<sub>DS(on)</sub> = 24 mΩ typ at V<sub>GS</sub> = 2.5 V, fully on from a 3.3 V gate | **AO3400A** (SOT-23) |
| LEDs with the wrong package and V<sub>f</sub> for 2-in-series on 4.8 V | Filtered DigiKey.ca's parametric search on package, colour and forward voltage | **Lumex SSL-LX5093AD**: T-1¾, 605 nm amber, diffused, V<sub>f</sub> = 2.0 V typ at 20 mA |
| LiPo with an onboard charger | NiMH cells and a charger already owned; a LiPo adds parts, failure modes and a fire risk on my back | **NiMH only**: 4 × AA for the panel, 3 × AAA for the handlebar |
| **REG1117 / REG1117A** LDO | An LDO cannot boost; NiMH sags from 5.35 V to below 3.3 V, so the logic would drop out before the cells are empty | **TPS63070 buck-boost** (SparkFun COM-15208, DigiKey 1568-15208-ND) |
| **Hammond 1553WBBK** enclosure | No gasket and still needs machining; 3D printing is available | **3D-printed enclosure** with gasket groove and heat-set inserts |
| **5ET 2-R** fuse | Listed end-of-life, July 2024 | **Bel Fuse 5HT 2-R** (507-1210-ND) |
| 3M FP301 heat-shrink kit | Wrong size range and price | Qualtek Q2-F-QK1-01-6IN-180 |
| "SPST slide switch" | Every common part found (C&K 1101, E-Switch EG, NKK SS12, Nidec MFS101) is SPDT with 3 terminals | Bought; wired common plus one throw |

Two findings mattered beyond the part swap:

- **The LDO.** It would have failed silently partway through a ride, once the pack fell below dropout. It was caught by asking what the rail does at the *end* of the discharge curve.
- **SOT-23 on a breadboard.** The AO3400A is surface-mount and does not fit a breadboard, a fact visible only on the package drawing. SOT-23-to-DIP adapters and header pins were added to the order ([what that cost](/turn-signal/part-3-soldering-and-panel-build/#learning-sot-23-soldering)).

![The DigiKey order as it arrived, packing slip on top](/images/digikey_order.jpg)

*The resulting order, 28 August 2026. Visible on the slip: SOT-23 to DIP adapters, 30 V N-channel SOT-23 MOSFETs and tactile switches.*
{: .caption}

## Build guide and work plan

Two documents existed before any hardware and are kept in sync:

- **Build guide** (`Jacket_Turn_Signals_Build_Guide.docx`): specification, architecture, BOM, calculations. The source of truth for *what and why*.
- **Work plan** (`Jacket_Turn_Signals_Work_Plan.docx`): Stage A (simulation), Stage B (breadboard) and Stage C (enclosure and integration), each split into numbered steps with **explicit pass criteria** and a **gate checklist** at the end of each stage.

The work plan follows the EE 232 lab-report structure: objective, procedure, expected result, pass criteria, observations. Writing pass criteria first defines success before the test is run, which is what made the later debugging tractable: when the LEDs did not light, there was already a written statement of what "lit" meant and what current it should draw.

**Next:** [Part 2](/turn-signal/part-2-simulation/), proving the design in simulation.
