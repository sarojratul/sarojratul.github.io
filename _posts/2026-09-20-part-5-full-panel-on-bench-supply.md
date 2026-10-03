---
layout: series-post
title: "Part 5: A Bench Supply and All Four Segments Lit"
date: 2026-09-20 12:00:00 -0600
series: turn-signal
part: 5
covers: "20 September 2026"
excerpt: "A bench supply with a proper current limit, segments B, C and D brought up, a current shortfall traced to LED forward voltage, all four segments lit together, and a supply sweep that emulates a draining battery."
permalink: /turn-signal/part-5-full-panel/
---

Work resumed after a two-week gap. Segment A's bring-up is in [Part 4](/turn-signal/part-4-floating-ground/#segment-a-verified).

<details class="toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## Bench supply setup

<p class="entry-date">20 September 2026</p>

**Objective.** Move the rail from the 4 × AA pack to an adjustable bench supply, and confirm its current limit is set correctly first.

**Method.** The supply is a **Jesverty SPS-6005N** (0 to 60 V, 0 to 5 A, single channel), bought for another project, used here because it holds an exact voltage and caps current. The multimeter's replacement fuse also went in, restoring the ammeter ranges. The SPS-6005N has no current-set button; per its manual, set the voltage with the output open, short the terminals with a wire, turn the current knob to the desired limit while the short holds the supply in constant-current mode, then remove the short.

**Result.** The first attempt set the limit too low. With segment A lit, the display read **4.10 V and 0.081 A**: current pinned and voltage sagging below the 4.80 V target, the constant-current crossover described in the manual. On the second attempt the shorted display read 0.450 A, 0.07 V, 0.031 W (0.07 × 0.45 = 0.0315, consistent), and with the short removed the rail held 4.80 V under load from then on.

**Notes.** The AD2's ground must still be tied to the supply's negative terminal, the same rule as with the pack. The supply's green earth terminal was left unconnected, since joining chassis earth to the AD2's USB-referenced ground risks a ground loop neither instrument would report clearly.

## Segment D bring-up

<p class="entry-date">20 September 2026</p>

**Objective.** Repeat segment A's bring-up on segment D, the other five-string arrowhead, on the new supply.

**Method.** Segment D's parts had sat populated but unwired since early September. I wired the five string anodes to a bus, the 220 Ω / 10 kΩ gate network, source to the shared ground rail and drain to the string bus. Supply black to the ground rail, red to D's bus, AD2 ground to the supply's black terminal, and gate injection at the input side of the 220 Ω with Wavegen at DC offset 3.3 V.

One instrument quirk: **stopping Wavegen does not reliably drive its output to 0 V.** The LEDs stayed lit with Run stopped while the lead was still in the board. Setting the offset to 0 V, or unplugging the lead so the 10 kΩ pulldown takes over, both work.

**Result.** Segment D lit: ten LEDs, five strings, visually the same as segment A. Whether its currents were correct was a separate question.

## Segment D current below design

<p class="entry-date">20 September 2026</p>

**Objective.** Hold segment D to segment A's standard: every string at design current and the MOSFET fully on.

**Method.** Inline ammeter (200 mA range) between the resistor bus and the MOSFET drain, cross-checked against the supply display; voltage across each of the five 33 Ω resistors, live; V<sub>DS</sub> with the gate high on the 200 mV range; then, once the numbers missed, voltage across each LED in one string and a full reseat of that string.

| Measurement | Predicted | Measured | Verdict |
|---|---|---|---|
| Total segment current | 123.9 mA (5 × 24.78 mA) | 80 to 89 mA (ammeter and supply display agree) | Below target |
| Voltage across one 33 Ω string resistor | 0.818 V | 0.59 V, identical on all five strings | Below target |
| Implied string current | 24.78 mA | 17.9 mA | Below target |
| Rail voltage under load | 4.80 V | 4.80 V, steady | Pass |
| V<sub>DS</sub> with gate high | a few mV (segment A: 3.3 mV) | 2.2 mV | Pass |
| Implied R<sub>DS(on)</sub> | 24 mΩ typical | 2.2 mV / 89 mA ≈ 25 mΩ | Pass |
| Voltage across one LED | ~1.99 V (design assumption) | 2.08 V, both LEDs in the string agree | Higher than assumed |

**Diagnosis.** The first suspect was breadboard contact, after two weeks unwired. The measurements ruled it out one at a time: the rail held 4.80 V, so not the supply; V<sub>DS</sub> of 2.2 mV meant the MOSFET was fully on; identical 0.59 V on all five resistors pointed to something affecting every string equally rather than one bad string; and reseating changed nothing.

The closing number: **2.08 V per LED.** 2.08 + 2.08 + 0.59 = 4.75 V, the 4.80 V rail less small probe and contact losses. Nothing is unaccounted for; the extra forward voltage over the modelled 1.99 V is exactly where the missing current went.

**Cross-check.** Segment A, untouched since 7 September, showed the same shortfall on the new supply: about 80 mA instead of its original 124 mA. Two segments with different wiring histories giving the same number rules out a segment D fault.

**Conclusion.** Segment D's topology, gate drive and MOSFET are correct, at a lower operating point than predicted. B and C should be expected to land below their 74.3 mA design figure too, and that alone is not a fault.

The result is unusual on its own terms: LED forward voltage normally falls slightly at lower current, and 17.9 mA is lower than the 24.78 mA both segments ran at on 7 September. Something changed between sessions (the LEDs, the supply, two weeks in the breadboard) and I could not isolate which. I stopped chasing it because 2.08 V is inside the Lumex SSL-LX5093AD datasheet window (2.0 V typical, 2.5 V maximum), nothing is out of spec, and the string's voltage budget closes. The 123.9 mA and 74.3 mA design figures now read as a ceiling from an idealised 1.99 V model.

![Segment D lit under the new bench supply](/images/segment_d_bench_supply.jpg)

*Segment D on the Jesverty SPS-6005N. The display reads 4.82 V, 0.089 A, 0.428 W, matching the sum of the five string currents within rounding. The unlit LEDs and adapters elsewhere belong to segments B and C, populated but not yet wired.*
{: .caption}

A 30% current difference was not visible by eye; the brightness looked the same as in the earlier session. The meter catches it, not the LED.

## Segments B and C, and all four together

<p class="entry-date">20 September 2026</p>

**Objective.** Bring up the two body segments, then complete B5's pass criteria: each segment lights alone with the others dark, then all four together while watching the rail.

**Method.** Same supply, grounding and resistor-voltage method as segment D. Then the AD2's 3.3 V line touched to each 220 Ω gate resistor in turn (B5 step 8), and finally all four gates driven high, reading the total from the supply display.

| Segment | Rail | Current | Power |
|---|---|---|---|
| B | 4.81 V | 0.053 A | 0.254 W |
| C | 4.81 V | 0.053 A | resistor and LED voltages identical to B |
| All four together | 4.81 V | 0.265 A | 1.274 W |

![All four segments of the arrow lit at once](/images/all_four_segments_lit.jpg)

*All four segments lit together for the first time, B5's final pass criterion. The ribbon cable on the right is the ESP32-C3 wiring staged for B7, not yet connected.*
{: .caption}

![The bench supply reading during the all-four test](/images/bench_supply_all_four_reading.jpg)

*0.265 A at 4.81 V. The individually measured segments (B and C at 53 mA each, A and D near segment D's 80 to 89 mA) sum to about 266 mA, which confirms the reading.*
{: .caption}

![Probing each gate resistor in turn to confirm the segments switch independently](/images/isolation_test_probe.gif)

*The AD2's 3.3 V line touched to each gate resistor in turn. Each segment lights alone with the rest dark: four independent switches, not one circuit that only looks right with everything on.*
{: .caption}

**Conclusion.** B and C match each other exactly, so the forward-voltage shift is not specific to segment D. All four segments sit consistently below the 74.3 mA and 123.9 mA design figures and are each internally consistent. The isolation test and the combined-current test both passed, which **closes B5**: correct wiring, independent switching, and a total fully explained by the individual measurements.

## Supply sweep: emulating a draining battery

<p class="entry-date">20 September 2026</p>

**Objective.** The finished panel runs from discharging NiMH cells, not a fixed 4.8 V. Measure how segment A's current changes as the rail sags.

**Method.** Segment A only, rail set by hand to four points, current read from the supply display.

| Rail voltage | Measured current |
|---|---|
| 5.0 V | 102 mA |
| 4.8 V | 88 mA |
| 4.5 V | 67 mA |
| 4.0 V | 31 mA |

Every point sits below the [LTspice sweep](/turn-signal/part-2-simulation/#ltspice-led-strings-and-mosfet-switch-stage-a2) for a five-string segment, consistent with the higher measured V<sub>f</sub>. The shape matters more than the offset:

- 5.0 to 4.8 V: current falls 14% for a 4% voltage drop
- 4.8 to 4.5 V: current falls 24% for a 6% voltage drop
- 4.5 to 4.0 V: current falls 54% for an 11% voltage drop

**Conclusion.** This is diode-knee behaviour: current collapses as the supply approaches the LEDs' effective turn-on voltage. The panel will look close to full brightness for most of the pack's charge, then dim sharply near the end, so its brightness is a poor fuel gauge. A fixed bench voltage would never have shown this.

## Step B6 closed

<p class="entry-date">20 September 2026</p>

B6 exists to prove the panel can run from a source that will not be damaged and needs no babysitting, originally by testing against a USB power bank. The bench supply already fills that role: it holds an exact voltage, limits current, and has run every bring-up so far without a reset or brownout. B6 is marked done by the equipment in use, with no separate test.

**Next:** B7, the ESP32-C3 driving the gate networks. See [Part 6](/turn-signal/part-6-esp32-drives-the-panel/).
