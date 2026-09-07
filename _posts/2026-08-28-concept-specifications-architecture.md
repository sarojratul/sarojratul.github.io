---
layout: post
title: "Concept, Specifications and Architecture"
date: 2026-08-28
categories: bike-turn-signals
---

## Late August 2026: Two arrows became one

The first architecture was the obvious one: **two separate arrows**, one on each backpack strap, three segments each, six MOSFETs, six GPIOs.

I killed it for two reasons, one aesthetic and one electrical:

- Two arrows on two shoulder straps looks goofy. It looks like a costume, not a piece of safety equipment.
- One **double-headed arrow** (`◄===►`) mounted horizontally shares its middle. The body segments are used by *both* directions. That is fewer segments, fewer MOSFETs, fewer gate networks, fewer GPIOs, and it looks like something you would actually wear.

**Final layout, left to right:**

| Segment | What it is | LEDs | Strings (2 LEDs each) |
|---|---|---|---|
| **A** | Left arrowhead | 10 | 5 |
| **B** | Body, left of centre | 6 | 3 |
| **C** | Body, right of centre | 6 | 3 |
| **D** | Right arrowhead | 10 | 5 |

**32 LEDs, 16 strings, 4 segments, 4 MOSFETs, 4 GPIOs.**

![Amber LEDs arranged into the arrow shape on a breadboard](/images/arrow_layout_leds.jpg)

*Working out the physical arrow shape in LEDs before committing to a wiring plan. The electrical grouping into 2-LED strings and the visual grouping into an arrow are two different problems, and this is the visual one.*

![Double-headed arrow split into four coloured segments, A B C D](/images/arrow_segments.svg)

*The final layout. Segments B and C sit either side of the centre and are used by both directions, which is the whole reason this shape beats two separate arrows. Currents shown are the simulated and hand-calculated figures from the power budget.*

### The LED count went 7/4/4/7 to 10/6/6/10

Worth recording because the reason is not aesthetic.

Every string is **two LEDs in series**, and that is set by the supply. Four in series needs about 8 V and I only have 4.8 V. So a segment with an *odd* number of strings does not map cleanly onto the topology I had already simulated, and an odd LED count leaves a stray LED with nowhere to go.

Going to **10/6/6/10** meant:

- Every segment is a whole number of identical 2-LED strings, so I could reuse the LTspice results I already had instead of re-simulating.
- Every string uses the same 33 Ω resistor. **No special cases anywhere in the BOM.** One resistor value, sixteen times.
- More LEDs is strictly better for the actual goal, which is a driver seeing me at night.

(For the record: an early draft said 6/4/4/7. That was an AI mis-read of my own 7/4/4/7 note. Caught it on the next pass. It is the kind of error that is invisible unless you check the numbers against your own source.)

---

## The sweep came from a lab I enjoyed

In **EE 232** we designed the sequential taillights from a 1965 Ford Thunderbird, the ones that sweep outward instead of just blinking. I liked that lab more than almost anything else in second year.

So this arrow does not blink. It **sweeps from the centre outward**, and the order depends on the direction:

- **LEFT:** C to B to A (accumulating, so the arrow grows toward the left head)
- **RIGHT:** B to C to D

Same shared body segments, opposite order. The motion itself carries the direction information, which is exactly the trick the Thunderbird used.

`STEP_MS = 120`, `CYCLE_MS = 800`.

---

## How I actually used AI, and why that is a skill rather than a shortcut

I want to be precise about this because it is the part people will assume the worst about.

**Why I used it at all.** I knew CME 331 (microcontrollers) was coming in Fall 2026, and that EE 321 would hand me a design problem with no lab manual holding my hand. Every second-year lab I had done walked me through the design. I wanted to practise *being the one who designs it* before I was graded on it. Working through a real design with an AI, then dragging every one of its claims back to a datasheet, seemed like a good way to build that muscle, and a good way to walk into labs carrying real problems, so that when an instructor solved one efficiently in front of me it would actually stick.

**How the loop worked:**

1. AI drafts a BOM and a sequence of steps.
2. I read the datasheets myself.
3. I find what is wrong, obsolete, or hallucinated.
4. I take it back to the AI with the evidence, and the BOM gets updated.
5. Repeat for the next component.
6. **Only when nothing is left unverified**, order from DigiKey.ca.

Nothing got ordered until it survived that loop. Here is what the loop actually caught:

| What was proposed | What I found | What I ordered |
|---|---|---|
| An older microcontroller with no integrated radio | I needed a peer-to-peer link with no router, on a bike. ESP-NOW needs an ESP chip. | **ESP32-C3** (XIAO / SuperMini). Cheap, Wi-Fi + BLE, ESP-NOW native |
| **IRLB8721** MOSFET | It works, but it is a TO-220 brick and overkill for 124 mA. Reading the AO3400A datasheet: V<sub>GS(th)</sub> = 1.05 V typ, R<sub>DS(on)</sub> = 24 mΩ typ at V<sub>GS</sub> = 2.5 V, fully on at a 3.3 V gate drive with margin | **AO3400A** (SOT-23) |
| The wrong LEDs entirely: wrong package, wrong V<sub>f</sub> for a 2-in-series string on 4.8 V | Went to the DigiKey.ca parametric search myself and filtered on package, colour, and forward voltage | **Lumex SSL-LX5093AD**. T-1¾, 605 nm amber, diffused, V<sub>f</sub> = 2.0 V typ @ 20 mA |
| **LiPo** battery + onboard charge circuit | I already own 2800 mAh AA and 1100 mAh AAA NiMH cells and a charger. Adding a LiPo charger is extra parts, extra failure modes, and a fire risk strapped to my back | **NiMH only.** 4×AA (4.8 V nom) for the panel, 3×AAA for the handlebar unit |
| **REG1117 / REG1117A** LDO regulator | An LDO cannot boost. NiMH sags from 5.35 V fresh to below 3.3 V as it discharges, so an LDO drops out and the logic dies before the batteries are actually empty | **TPS63070 buck-boost** (SparkFun COM-15208, DigiKey 1568-15208-ND) |
| **Hammond 1553WBBK** enclosure | Fixed box, no gasket, and I would still have to machine it. I have access to 3D printing | **3D-printed enclosure** with a gasket groove and heat-set brass inserts |
| **5ET 2-R** fuse | DigiKey.ca listing shows **end-of-life, July 2024** | **Bel Fuse 5HT 2-R** (507-1210-ND) |
| A 3M FP301 heat-shrink kit | Wrong size range and wrong price for what I need | Qualtek Q2-F-QK1-01-6IN-180 |
| "SPST slide switch" | Every common part in that search (C&K 1101, E-Switch EG, NKK SS12, Nidec MFS101) is actually **SPDT with 3 terminals** | Bought them anyway, wired common + one throw |

Two of these are worth calling out as more than parts substitutions:

- **The LDO one is the one that changed how I work.** An LDO instead of a buck-boost is not a typo. It is a proposal that fails silently, halfway through a ride, when the batteries drop below the dropout voltage. It looked completely reasonable in a parts list. The only reason I caught it is that I asked "what happens to this rail at the *end* of the discharge curve," which is an EE 221 question, not an AI question.
- **The SOT-23 problem nobody mentioned.** The AO3400A is a surface-mount part. It does not plug into a breadboard. That fact appears in no BOM; you find it when you look at the package drawing in the datasheet. I ordered **SOT-23-to-DIP adapters** and header pins to go with them. (What that cost me is Part 3.)

![The DigiKey order as it arrived, packing slip on top](/images/digikey_order.jpg)

*The order that came out of that loop, 28 August 2026. Visible on the slip: SOT-23 to DIP adapters, MOSFET N-CH 30 V SOT-23, tactile switches. Every line on that page survived a datasheet check first.*

**What I would tell another student:** an AI is very good at producing a plausible-looking parts list and a plausible-looking sequence of steps. It is not accountable for either. The datasheet is the authority; the AI is a fast way to generate a first draft to argue with. The value is not in the answers it gives. It is in how quickly it gives you something concrete enough to check.

---

## Architecture

| | **Handlebar transmitter** | **Backpack receiver** |
|---|---|---|
| **Input** | 2× tactile switch (left / right) | ESP-NOW packets, 2.4 GHz |
| **Controller** | ESP32-C3 | ESP32-C3, GPIO 2 / 4 / 5 / 6 |
| **Drive stage** | none | 4× (220 Ω series into gate, 10 kΩ gate pulldown), into 4× AO3400A N-MOSFET, low-side |
| **Load** | 1× status LED + 1 kΩ | Segments A(10) B(6) C(6) D(10): 32 LEDs in 16 strings of 2 + 33 Ω |
| **Power** | 3×AAA NiMH 1100 mAh, TPS63070, 3.3 V | 4×AA NiMH 2800 mAh (4.8 V): **raw** to the LED panel, and via TPS63070 to 3.3 V for logic |
| **Role** | Stateless. Sends raw held / not-held levels on a heartbeat. | Holds all state and all behaviour. |

**The link:** handlebar ESP32-C3, to ESP-NOW (2.4 GHz, peer-to-peer, no router), to backpack ESP32-C3, to four GPIOs, to four gate networks, to four MOSFETs, to four LED segments.

![Block diagram: handlebar transmitter, ESP-NOW link, backpack receiver](/images/block_diagram.svg)

*The whole system. Note the asymmetry: the transmitter is a switch, a radio and a regulator, while everything that decides anything lives on the receiver. The 322.9 microamps and the 2.38 millivolts are the LTspice predictions that Part 3 goes on to check against the bench.*

**The transmitter is stateless on purpose.** It sends "left is held / left is not held" on a heartbeat and nothing else. All behaviour (sweep order, timing, trailing cycles, cancellation) lives on the receiver. If the radio link drops, the receiver knows (`linkAlive()`) and can fail safe on its own instead of being stuck in whatever state the transmitter last told it about.

### Activation model

- **While held:** that direction's sweep repeats continuously, however brief the press.
- **On release:** the sweep runs exactly `TRAILING_CYCLES` more full cycles, then goes dark. There is no off button and no timer to forget about.
- **Any press, in any state, cancels immediately** and starts a fresh hold of the newly-pressed direction.

`TRAILING_CYCLES` started at 2. I tuned it to **4** live in simulation because 2 cycles ended before I had finished the turn. It is a named constant precisely so that changing it is one line.

---

## Power budget

The panel runs **directly off the raw 4.8 V NiMH pack**. No boost, no regulator in the LED path. The regulator only feeds logic.

That is a deliberate efficiency choice. A boost converter in series with 400 mA of LEDs is loss and heat for no benefit, because the LEDs do not care about a stable rail: the series resistor sets the current, and a sagging pack just means slightly dimmer LEDs. The microcontroller *does* care, so it gets the TPS63070, which can both step 5.35 V down and step 3.0 V up to 3.3 V as the pack discharges.

**To be clear about the current state of the bench:** the TPS63070 board is bought and waiting, but it does not go in until work-plan step **B10**, the power stage. Everything I am testing today runs on the raw battery pack, with the 3.3 V gate drive coming from the Analog Discovery 2's waveform generator. The regulator gets characterised on its own, with a supply sweep across the full NiMH discharge range, before it is allowed anywhere near the rest of the circuit.

| Case | Current | Power |
|---|---|---|
| One string (2 LEDs + 33 Ω @ 4.8 V) | 24.78 mA | n/a |
| Segment B or C (3 strings) | 74.3 mA | 0.36 W |
| Segment A or D (5 strings) | 123.9 mA | 0.59 W |
| Peak during sweep (3 segments lit) | 272.6 mA | 1.31 W |
| All four lit (bench / fuse sizing) | 396.5 mA | 1.90 W |
| MOSFET dissipation, worst case (A/D) | n/a | ≈ 368 µW |

**Runtime** on 2800 mAh:

- Worst case, continuous, everything on: **≈ 9.12 h**
- Realistic, ~5% duty: **≈ 58.3 h**

The realistic figure still uses a generic 5% duty assumption rather than one derived from the actual hold-and-trailing-cycle behaviour. It is flagged open in my work plan. It clears the requirement by such a margin that refining it is not urgent, but it is *not* a measured number and I am not going to pretend it is.

---

## The documents

Two Word documents, kept in sync, both of which existed before any hardware did:

- **`Jacket_Turn_Signals_Build_Guide.docx`**: spec, architecture, BOM, calculations, phases. The source of truth for *what and why*.
- **`Jacket_Turn_Signals_Work_Plan.docx`**: the execution plan. Stage A (simulation), Stage B (breadboard), Stage C (enclosure and integration), each broken into numbered steps with **explicit pass criteria** and a **gate checklist** at the end of each stage. The source of truth for *order of work*.

The AI's first version of the work plan was structured its own way and I found it hard to follow. I made it restructure the whole thing to follow the **EE 232 lab report format**: objective, procedure, expected result, pass criteria, observations. That was much more efficient, because it is the format I have already been trained to read and write, and because "pass criteria" forces you to decide what success looks like *before* you are emotionally invested in the thing working.

That structure is doing real work now. When the LEDs did not light, I already had a written statement of what "lit" was supposed to mean and what current it was supposed to draw.

---
