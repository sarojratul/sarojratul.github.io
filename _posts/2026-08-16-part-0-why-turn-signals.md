---
layout: series-post
title: "Part 0: Why Build Bicycle Turn Signals"
date: 2026-08-16 12:00:00 -0600
series: turn-signal
part: 0
covers: "Mid August 2026"
excerpt: "The problem the project solves, why it became a backpack panel instead of a jacket, and the bench it was built on."
permalink: /turn-signal/part-0-why/
redirect_from:
  - /2026/08/16/why-i-built-bicycle-turn-signals.html
---

## The problem with hand signals

<p class="entry-date">Mid August 2026</p>

Hand signals on a bicycle fail in three situations I ride into regularly:

1. **An injured right wrist.** Signalling a left turn leaves only the weaker hand on the bar, on rough Saskatoon streets.
2. **Night.** Drivers see lights, not an arm.
3. **Aggressive traffic.** When a driver behind honks or cuts close, both hands belong on the bar, at exactly the moment a signal is needed.

The design brief follows directly: **signal a turn without taking a hand off the bar, in a way a driver can read at night.**

## From a jacket to a backpack panel

<p class="entry-date">Mid August 2026</p>

The first concept was a jacket with LEDs sewn into the back, which is why the project folder and documents still say "jacket". It was dropped after one ride, for three reasons:

- **Nobody wears a jacket in July.** Almost every rider carries a backpack.
- **Textiles are a materials problem.** Flexible wiring, fabric mounting and washability were out of scope for a first build.
- **Durability.** A jacket gets stuffed into a bag; a rigid enclosure on a backpack does not.

The design became a **rigid, backpack-mounted panel**. The jacket stays in the documents as an out-of-scope fallback, marked superseded rather than deleted, a convention the rest of this log follows.

## The workstation

<p class="entry-date">Mid August 2026</p>

The bench was set up in mid August 2026, after completing EE 221 (analog electronics) and EE 232, and funded by bursaries and awards from my second year. It includes:

- Soldering station, solder, fume extractor, heat gun and glue gun
- **Digilent Analog Discovery 2**: USB oscilloscope, logic analyzer, waveform generator and supply. It has been the most useful purchase, because it localises a fault rather than only detecting one.
- Multimeters, breadboards, jumper wire, perfboard and helping hands
- Ceramic capacitor, electrolytic capacitor and resistor assortments
- The project's bill of materials, ordered from DigiKey.ca

![The workstation: lamp, multimeter, helping hands, soldering mat, breadboard, parts drawers](/images/workstation.jpg)

*The complete bench: a desk in a bedroom with a lamp, silicone mat, helping hands with a spool holder, multimeter, magnifier, fume extractor and two parts drawers. Every step in this log so far was run on it.*
{: .caption}

![24-value, 480-piece ceramic capacitor assortment](/images/capacitor_kit.jpg)

*480 ceramic capacitors in 24 values, 10 pF to 10 µF, 50 V. The 100 nF row supplies the decoupling at the microcontroller.*
{: .caption}

![Through-hole resistor kit with the colour code chart on the lid](/images/resistor_kit.jpg)

*The resistor kit, colour-code chart on the lid. The 33 Ω value is used sixteen times, once per LED string.*
{: .caption}

![Electrolytic capacitor assortment in a 24-compartment tray](/images/electrolytic_kit.jpg)

*Electrolytics sorted by value. The 470 µF bulk capacitor across the panel's supply rail came from this tray.*
{: .caption}

**Next:** the design itself, in [Part 1](/turn-signal/part-1-design/).
