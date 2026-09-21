---
layout: series-post
title: "Part 5: A Bench Supply and All Four Segments Lit"
date: 2026-09-20 12:00:00 -0600
series: turn-signal
part: 5
covers: "20 September 2026"
excerpt: "A proper bench supply, segments B, C and D brought up, a current shortfall traced to LED forward voltage, all four segments lit together, and what a draining battery does to the arrow."
permalink: /turn-signal/part-5-full-panel/
---

After a two-week gap, the bench came back with a proper adjustable power supply. This entry brings up the other three segments, chases a current shortfall to its actual cause, lights all four segments together for the first time, and sweeps the supply to see what a draining battery will do to the arrow. Segment A's original bring-up is in [Part 4](/turn-signal/part-4-floating-ground/#segment-a-verified).

<details class="toc" markdown="1">
<summary>In this entry</summary>

* TOC
{:toc}

</details>

## A bench supply, and learning to set its current limit

<p class="entry-date">20 September 2026</p>

**Objective.** Get a proper adjustable bench supply into the rail-power role the 4×AA NiMH pack had been filling, and confirm it is set up correctly before trusting it with the panel.

**Method.** The supply is a Jesverty SPS-6005N, 0 to 60 V, 0 to 5 A, single channel, bought for a different project entirely and pressed into service here because it can hold an exact voltage and cap current in a way four AA cells never could. The multimeter's replacement fuse also went in today, a straight clip swap, no iron needed, and the ammeter ranges came back with it.

The SPS-6005N has no dedicated button for setting the current limit. The manual's own procedure is to dial the voltage to target first with the output open, then short the positive and negative terminals together with a spare wire, then turn the current knob until the display shows the desired limit while that short forces the supply into constant-current mode, then remove the short. Voltage first, current second, in that order, because the limit-setting trick only reads correctly once the real target voltage is already dialed in.

**Result.** First attempt: dialed 4.80 V, then set the current knob without paying close enough attention to the short-and-set order, and ended up with the limit set too low. Symptom: with segment A connected and lit, the display read 4.10 V and 0.081 A, current pinned flat while voltage sagged well below the 4.80 V target. That is the textbook constant-current crossover signature from the manual's own CV/CC description: once current hits the set limit, the supply holds current and lets voltage collapse to whatever the load demands.

Redid it properly. Shorted the leads again, turned the current knob up, and confirmed on the display: 0.450 A, 0.07 V, 0.031 W while shorted (0.07 × 0.45 is 0.0315, so the numbers agree with each other, this is not a fault, it is just what a dead short looks like in CC mode). Removed the short, reconnected the segment, and the rail held at 4.80 V under load from then on.

**What it means.** The supply now does what the AA pack did, hold a rail near 4.8 V, but with an actual current ceiling instead of hoping nothing goes wrong, and with a front-panel ammeter as a second, independent way to read current alongside the multimeter. The AD2's ground still has to be tied to the supply's negative terminal, the exact same non-negotiable rule as with the pack. The supply's own grounding terminal (green, separate from the +/- output) was left unconnected, since it is chassis/earth ground and mixing it with the AD2's USB-referenced ground risked a ground loop neither instrument would report cleanly.

---

## Segment D, wired from scratch

<p class="entry-date">20 September 2026</p>

**Objective.** Repeat the 7 September segment A bring-up on segment D, the other five-string arrowhead, this time on the new supply.

**Method.** Segment D's parts had been sitting populated but unwired in the breadboard since early September, the same "leave it in place, don't repopulate" approach used throughout Stage B5. Wired its five string anodes to a bus, the 220 Ω and 10 kΩ gate network, the MOSFET source to the shared GND rail and drain to the string bus, same conventions as every other segment on this board. Supply's black lead to the GND rail, red lead to D's bus. AD2 ground tied to the supply's black terminal. Gate injection at the input side of the 220 Ω, same as before, with the AD2's Wavegen set to DC offset 3.3 V.

One instrument quirk turned up while getting the gate signal working: stopping Wavegen's Run does not reliably drive its output to 0 V. The LEDs stayed lit with Run stopped, as long as the output pin was still physically in the breadboard, which meant the channel was holding its last driven level rather than going to zero or high impedance. Setting the offset itself to 0 V, or just unplugging the lead and letting the 10 kΩ pulldown do its job, both worked. Worth remembering for B and C.

**Result.** Segment D lit. Ten LEDs, five strings, visually no different from segment A.

**What it means.** The topology, gate network and MOSFET assembly for D are physically correct. Whether the currents are actually right is a separate question, and the numbers said something more interesting than a clean pass.

---

## Segment D's current came up short, and why I stopped chasing it

<p class="entry-date">20 September 2026</p>

**Objective.** Confirm segment D is not just lit but correct, to the same standard segment A was held to on 7 September: every string at its design current, the MOSFET fully on.

**Method.** Inline ammeter, now that the fuse is fixed, on the 200 mA range, in series between the string-resistor bus and the MOSFET drain. Cross-checked against the supply's own front-panel current display. Then voltage across each of the five 33 Ω string resistors individually, live, nothing disconnected. Then V<sub>DS</sub> across the MOSFET with the gate held high, on the 200 mV range. Then, once the numbers did not match the design target, voltage across each LED in one string, and a full reseat of that string's connections to rule out a bad contact.

**Result.**

| Measurement | Predicted | Measured | Verdict |
|---|---|---|---|
| Total segment current | 123.9 mA (5 × 24.78 mA) | 80 to 89 mA (inline ammeter and supply display agreed) | short of target |
| Voltage across one 33 Ω string resistor | 0.818 V | 0.59 V, uniform across all five strings | short of target |
| Implied string current | 24.78 mA | 17.9 mA | short of target |
| Rail voltage under load | 4.80 V | 4.80 V, steady | PASS |
| V<sub>DS</sub> with gate high | a few mV (segment A read 3.3 mV) | 2.2 mV | PASS |
| Implied R<sub>DS(on)</sub> | 24 mΩ typical (datasheet) | 2.2 mV / 89 mA, about 25 mΩ | PASS |
| Voltage across one LED | ~1.99 V (design assumption) | 2.08 V, both LEDs in the checked string agreed | higher than assumed |

**My first thought was the breadboard wiring**, since D had sat populated but disconnected for two weeks and a marginal contact seemed like the obvious explanation for a segment drawing less than it should. The ladder ruled that out step by step instead of confirming it. The rail held at a steady 4.80 V under load, so the supply was not the cause. V<sub>DS</sub> came back at 2.2 mV, in the same range as segment A's healthy 3.3 mV reading, so the MOSFET was fully on, not sitting somewhere in its linear region. All five string resistors read the identical 0.59 V, which rules out one bad string dragging the total down and instead points at something affecting every string the same way. Reseating the checked string's LEDs and resistor changed nothing.

The number that closed it: 2.08 V across each LED, both LEDs in the string agreeing with each other, and 2.08 + 2.08 + 0.59 adds up to 4.75 V, which is the 4.80 V rail within the kind of small loss you get from probe and contact resistance. Nothing is unaccounted for. The extra voltage the LEDs are taking, compared to the 1.99 V figure the design calculations assumed, is exactly and only where the missing current went.

Then the real cross-check: segment A, still wired exactly as it was left on 7 September and untouched since, powered up on the same new supply and showed the same shortfall, roughly 80 mA instead of its original 124 mA. Two segments, two completely different wiring histories, the same number. That is stronger evidence than anything on D alone that this is not a D-specific fault.

**What it means.** It proves D's topology, gate drive and MOSFET all work correctly, the same conclusion segment A earned on 7 September, just at a lower operating point than either segment showed back then. It does not prove anything about B or C yet, but it sets the expectation that they will likely land under their 74.3 mA design figure too, and that alone should not be read as a fault when it happens.

It is a genuinely odd result on its own terms. LED forward voltage normally drops a little at lower current, not rises, and 17.9 mA is lower than the 24.78 mA both segments ran at on 7 September. Something changed between then and now, the LEDs themselves, the different supply, two weeks sitting in a breadboard, and I do not have a clean answer for which. I decided not to keep chasing it, since 2.08 V is still comfortably inside the Lumex SSL-LX5093AD's datasheet window of 2.0 V typical to 2.5 V maximum, nothing here is out of spec, and the whole voltage budget for the string is fully accounted for. The original 123.9 mA and 74.3 mA numbers from the design calculations now read as a ceiling from an idealized 1.99 V LED model, not as an exact bench prediction.

![Segment D lit under the new bench supply](/images/segment_d_bench_supply.jpg)

*Segment D lit, powered by the Jesverty SPS-6005N instead of the AA pack for the first time. The display reads 4.82 V, 0.089 A, 0.428 W, matching the sum of the five string currents from the resistor measurements to within rounding. The unlit LEDs and MOSFET adapters elsewhere on the board belong to segments B and C, populated but not yet wired.*
{: .caption}

I suspected the wiring first, mostly because of the two weeks off the bench. Once the numbers came back low but self-consistent rather than erratic, it stopped feeling like a fault to chase, and the brightness looked no different from what I remembered from the last session, which in hindsight is its own small lesson: a 30 percent current difference does not necessarily look like anything to the eye. The meter is the instrument that catches it, not the LED.

**Next:** wire and test segments B and C the same way, on the new supply, expecting the same lower-than-design currents rather than treating it as a surprise a second time. Then the four-segment hand test, and the all-four-lit measurement, against whatever the real total turns out to be rather than the original 396.5 mA figure.

---

## Segments B and C, and the panel together for the first time

<p class="entry-date">20 September 2026</p>

**Objective.** Repeat the bring-up done on segment D earlier today on segments B and C, the two body segments, then run the rest of B5's pass criteria: each of the four segments lighting alone with the others dark, then all four together while watching the rail.

**Method.** Same bench supply, same grounding (black lead to the shared GND rail, AD2 ground tied to the same rail), same voltage-across-the-33 Ω-resistor method used on segment D. Then, one segment at a time, touched the AD2's 3.3 V line to each of the four 220 Ω gate resistors in turn, the hand test from B5 step 8: each segment should light by itself with the rest of the board dark. Then all four gates driven high together, reading the total off the bench supply's own display.

**Result.**

| Segment | Rail | Current | Power |
|---|---|---|---|
| B | 4.81 V | 0.053 A | 0.254 W |
| C | 4.81 V | 0.053 A | resistor and LED voltages matching B exactly |
| All four together | 4.81 V | 0.265 A | 1.274 W |

![All four segments of the arrow lit at once](/images/all_four_segments_lit.jpg)

*All four segments lit together for the first time, B5's final pass criterion. The rainbow ribbon cable on the right belongs to the ESP32-C3 wiring staged for B7, not yet connected to anything live here.*
{: .caption}

![The bench supply reading during the all-four test](/images/bench_supply_all_four_reading.jpg)

*0.265 A total at 4.81 V. Summing each segment's own individually measured current (B and C at 53 mA each, A and D both running close to segment D's already-measured 80 to 89 mA) predicts about 266 mA, close enough to the 265 mA on the display to call it confirmed rather than coincidental.*
{: .caption}

![Probing each gate resistor in turn to confirm the segments switch independently](/images/isolation_test_probe.gif)

*Touching the AD2's 3.3 V line to each of the four 220 Ω gate resistors in turn. Each segment lights on its own with the rest of the board dark, which is the whole point of the test: four independent switches, not one circuit that happens to look right when everything is on together.*
{: .caption}

**What it means.** B and C landing at exactly the same current as each other, down to the resistor and LED voltages, says the LED forward-voltage shift found on segment D earlier today is not specific to that one segment. All four segments now read consistently below the original 74.3 mA and 123.9 mA design figures, and every one of them is internally self-consistent doing it. Both the isolation test and the combined-current test came back clean, which closes B5: the four segments are wired correctly, they switch independently of each other, and the total draw is fully explained by the four segments' own individually measured numbers. None of this points at a fault. It points at a different, better-understood baseline than the LTspice model assumed.

---

## What draining a battery actually looks like

<p class="entry-date">20 September 2026</p>

**Objective.** Segment A had already shown a lower-than-designed current on the new bench supply. Before treating that as just a fixed offset, find out how that current actually changes as the rail sags, since the finished panel runs off four NiMH cells discharging toward empty, not a bench supply holding a fixed 4.8 V.

**Method.** Segment A only, rail dialed by hand to four points: 5.0 V, 4.8 V, 4.5 V and 4.0 V, reading the supply's own current display at each.

**Result.**

| Rail voltage | Measured current |
|---|---|
| 5.0 V | 102 mA |
| 4.8 V | 88 mA |
| 4.5 V | 67 mA |
| 4.0 V | 31 mA |

Every one of those sits well under what the [LTspice sweep from Stage A2](/turn-signal/part-2-simulation/#ltspice-the-analog-half-stage-a2) predicts at the same voltage for a five-string segment, consistent with the higher LED V<sub>F</sub> already found on segment D today. The more interesting part is the shape of the drop, not the offset: 5.0 to 4.8 V is a 14% drop in current for a 4% drop in voltage, 4.8 to 4.5 V is a 24% drop in current for a 6% drop in voltage, and 4.5 to 4.0 V is a 54% drop in current for an 11% drop in voltage. The percentage drop in current accelerates much faster than the percentage drop in voltage.

**What it means.** That is diode-knee behaviour: current through an LED does not fall linearly as forward voltage drops, it falls off a cliff once the voltage gets close to the diode's effective turn-on point. In practice, the panel will not dim evenly as the NiMH pack discharges. It will look close to full brightness for most of the pack's charge, then get dramatically dimmer over the last stretch before the pack is empty, rather than fading the whole way down. Worth knowing before trusting a glance at the panel to judge how much charge is left.

I ran the sweep to emulate the battery pack, because I will not be carrying a DC bench supply on the bike. The shape of the result is exactly the kind of thing a fixed bench voltage would never have shown.

---

## B6, closed by the equipment already on the bench

<p class="entry-date">20 September 2026</p>

B6 exists to prove the panel can run from a power source that will not get damaged and does not need babysitting, before anything more complicated gets wired in. As originally planned that meant a separate test against a USB power bank.

That objective was already satisfied before B6 was ever reached. The bench supply took over the rail-power role days ago specifically because it holds an exact voltage and caps current in a way four AA cells or a power bank cannot, and every segment bring-up done so far, including all of today's, has run on it without an unexpected reset or a brownout. Running a second test against a power bank would prove the same thing a second time. B6 is marked done by the equipment already in use, no separate test recorded.

**Next:** B7, wiring the ESP32-C3 into the gate networks for real. That is [Part 6](/turn-signal/part-6-esp32-drives-the-panel/).
