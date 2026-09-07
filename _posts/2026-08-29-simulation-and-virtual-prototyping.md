---
layout: post
title: "Simulation and Virtual Prototyping"
date: 2026-08-29
categories: bike-turn-signals
---

## 29 August 2026: LTspice, the analog half (Stage A2)

**Objective:** prove the LED strings and the MOSFET switch before spending anything, and specifically prove the resistor value.

**Why LTspice at all:** we used it in university labs. Combined with EE 221, which is literally analog electronics, and where I learned MOSFET physics, how to actually measure resistance with a multimeter without fighting it, and the habit of spending hours on one bug instead of swapping parts randomly, this is the part of the project that felt like home.

*Screenshot wanted: the A2 LTspice schematic, and the DC sweep and transient plots.*

**Part 1, model the LED.** The Lumex part has no SPICE model, so I built one from the datasheet's V<sub>f</sub> curve:

```spice
.model AMBER D(Is=2e-18 N=2 Rs=3 Cjo=30p)
```

Result: **V<sub>f</sub> = 1.99 V per diode** at the operating point. Datasheet says 2.0 V typical at 20 mA. Close enough to trust.

Nominal string current at 4.8 V through 33 Ω: **24.78 mA.**

**Part 2, DC sweep, 4.0 V to 5.0 V.** This answers "what happens as the batteries die?"

| Supply | String current |
|---|---|
| 4.00 V | 7.45 mA |
| 4.50 V | 17.94 mA |
| 4.80 V | 24.78 mA |
| 5.00 V | 29.45 mA |

Monotonic, no surprises, and nothing exceeds the LED's rating even fresh off the charger. The steepness is worth noting: LED current is exponential in supply voltage, so the arrow will visibly dim as the pack drains. That is acceptable. It is not acceptable for the logic, which is the argument for the buck-boost all over again.

**Part 3, a full segment with the real MOSFET.** Three strings, an AO3400A low-side switch, 3.3 V on the gate. The AO3400A ships as a `.SUBCKT`, which took some fighting (see the gotchas below).

`.op` results:

- Total segment current: **74.16 mA** (24.72 mA per string)
- **V<sub>DS</sub> = 2.38 mV**, so the MOSFET is essentially a wire
- MOSFET dissipation: **≈ 176 µW**
- Gate sweep: current is flat at ~74 mA from about **1.4 V** all the way to 3.3 V

That gate sweep is the whole argument for choosing a logic-level MOSFET. It is fully on well below the 3.3 V the ESP32-C3 can supply, so there is no marginal-turn-on region to worry about.

Transient 10–90% fall time: **≈ 100 ns.** Irrelevant for a 120 ms step, but nice to know the switch is not the limiting factor.

> **The 2.38 mV number became my most useful debugging constant.** If the gate is high and V<sub>DS</sub> is millivolts, the MOSFET is on. If V<sub>DS</sub> is sitting at nearly the full rail, it is not. That is a one-probe answer to a question that would otherwise be guesswork.

**Not simulated:** segments A and D (5 strings) are hand-calculated only, at 123.9 mA and 0.59 W. They are the same topology with two more strings in parallel and I am comfortable with the arithmetic. Segments B and C are *exactly* this circuit, so those two are simulated, not estimated.

### LTspice gotchas I lost time to

- The component browser will not find a bare `.SUBCKT` with no symbol. Place a generic NMOS, right-click into MOSFET Properties, and type the subcircuit name in the **MOSFET** field.
- Also change the instance prefix from `M` to `X` (right-click into the attribute editor). A subcircuit instance is an `X`.
- **Near-zero MOSFET current in `.op` while everything else looks normal means the drain/source pins are not actually wired.** Look for the junction dot. This cost me a while.
- To read current through a specific pin in a `.dc` sweep, hover the wire for `Ix(<refdes>:<pinname>)`.
- The form-based PULSE editor is much less error-prone than typing the raw string.
- For precise timing, export the `.tran` result as text and interpolate around the threshold crossing. Do not eyeball a zoomed screenshot.

---

## 31 August 2026: Redesign day, mid-stream

This is the day the two-arrow design died and the shared double-headed arrow replaced it (Part 1). It happened *after* I had already simulated and documented the old one.

Throwing out finished work is supposed to feel bad. It mostly did not, for a specific reason: **the LED count landed on even numbers, which meant the simulation results I already had still applied.** Segment B/C is byte-for-byte the circuit I had run in LTspice two days earlier. The work was not wasted; it was reused.

What *was* painful was the documents. The old design was scattered across both files: segment counts, MOSFET counts, "six output-capable pins," an auto-cancel behaviour that no longer existed, an old BOM table shared with the out-of-scope jacket fallback. I rewrote all of Stage B, updated the gate checklists, and updated the firmware section.

**And I still missed one line.** Step 4 of B5 says "wire all 8 string anodes to the +4.8 V rail." It should say 16. That is a leftover from the two-arrow design that my rewrite pass walked straight past, and I found it a week later.

> **The lesson, which I am now applying to everything:** before a rewrite pass, *grep the whole document for the stale terms and numbers* (segment counts, pin counts, old behaviour names) instead of trusting your memory of the structure. Stale text hides in bulleted "you will need" lists and pass criteria, not in the headings you are looking at.

---

## 1–3 September 2026: Wokwi, the firmware half (Stage A3)

**Objective:** prove the state machine, not the electronics. Wokwi does not model current, so this was four LEDs, one per segment, and pure logic.

*Screenshot or GIF wanted: the Wokwi sweep running, both directions.*

The state machine:

1. **A radio-stub seam.** `commandedState()` and `linkAlive()` are the only two functions that know where the command comes from. In Wokwi they read local buttons; on real hardware they read `g_state` and `g_lastRx` set by the ESP-NOW receive callback. **The cost of swapping simulation for radio is two one-line function bodies.** Designing that seam in on day one is the single best decision in the firmware.
2. **Direction-aware sweep** via a 4-bit segment mask: `segmentMaskFor(dir, msIntoCycle)`.
3. **Hold plus trailing cycles**, with `cycleStartMs` latched on every new press.
4. `isNewPress = (cmd != activeDir) || (!wasHeld)`, which is what makes any press cancel any state.

### Bug 1: the cycle phase was free-running

The first version computed the animation phase from `millis()` directly. That meant pressing the button dropped you into whatever point of the 800 ms cycle happened to be current, so sometimes the arrow started mid-sweep. Latching `cycleStartMs` on the press fixed it, and it also fixed the trailing-cycle count, which had been off by a fraction of a cycle for the same reason.

### Bug 2: all four LEDs were wired backwards

Code correct. Buttons correct. Serial output correct. Nothing lit.

I want to be honest about how long this took me: **not long.** From EE 221 I know an LED is a diode, and a diode that will not conduct is either dead or backwards. Four of them all dead at once is not plausible. So they were backwards, and they were.

But the *general* lesson is the one that mattered, and I wrote it down at the time:

> **When nothing works, use what DID work to narrow down what did not.**

The serial output worked. The buttons worked. The timing worked. Every one of those rules out a whole category of cause. The failure was in the only part I had not verified. Guessing at causes is slower than eliminating them, even when guessing feels faster.

I did not know then how directly I would need that a week later.

### Still open

`HOLD_MS`. The full arrow is only lit for 120 ms out of every 800 ms cycle. I do not yet know whether that reads as "urgent and attention-grabbing" or "flickery and hard to parse" to a driver 30 m back, and I do not think I can answer it at a desk. It is flagged as an open question in the work plan and it gets settled outdoors, on the real panel, at night.

Also accepted and not fixed: `millis()` rolls over at 49.7 days. On a bike light that is powered off between rides, that is cosmetic.

**Pin note:** Wokwi used GPIO 2/3/4/5. Real hardware is **GPIO 2/4/5/6**, the ESP-NOW-ready receiver's pin set. That is locked and documented in three places, because a pin table that disagrees with itself is exactly the kind of thing that eats an evening.

---

## 5 September 2026: Tinkercad, which nobody suggested

At this point the plan said: go build it on a breadboard.

I did not want to. I had 32 LEDs, 16 resistors, 4 MOSFETs on adapters and a battery pack about to go onto one board, and if it did not work I would have no way to tell whether the *circuit* was wrong or a *leg was in the wrong hole*. Those are completely different problems and I did not want to debug both at once.

Wokwi could not help, because it does not model current and it has no MOSFET. LTspice proves the electronics but says nothing about physical layout. So I went looking myself and found **Tinkercad Circuits**, which simulates an actual breadboard with actual holes.

Two things I had to discover by poking at it:

- It **does** have NMOSFET parts. I missed them at first and started rebuilding with BJTs, which was a waste of an hour, because the 220 Ω / 10 kΩ gate network does not transfer to a BJT at all.
- The fixed batteries are only 9 V / 3 V / 1.5 V, but there is an adjustable **"custom power supply"**. I used two: 4.8 V @ 1 A for the panel rail, and 3.3 V for gate injection, with all negatives on one ground rail.

I injected the gate test signal at the **input side of the 220 Ω**, where the GPIO would attach, so that both the series resistor and the pulldown are actually exercised. (An unflashed microcontroller on that node is high-impedance and will not fight the injection, which is why the same trick works on the real board.)

**Result: all four segments passed.** Each lights alone under 3.3 V on its gate; all dark otherwise.

![Tinkercad simulation of the full four-segment panel](/images/tinkercad_full.jpg)

*The full rehearsal in Tinkercad Circuits: 16 strings of two LEDs, 16 resistors, four MOSFETs, the 470 uF bulk capacitor, a 4.8 V supply for the panel rail and a second supply for gate injection. This is what let me separate a wrong circuit from a leg in the wrong hole before touching real parts.*

One number that does **not** transfer: the sim showed 91.6 mA for a lit 5-string arrowhead (18.3 mA per string) because Tinkercad models a red LED at V<sub>f</sub> ≈ 2.1 V. Real amber is 1.99 V, so the real figure is ~123.9 mA. I wrote that discrepancy down *before* going to the bench specifically so I would not chase it later.

> **What Tinkercad proves and does not prove:** it proves the topology and the wiring plan. It proves nothing about real components, real currents, thermals, or whether a stranded wire is making contact. I knew that when I wrote it down. Part 3 is about finding out how much it does not prove.

---
