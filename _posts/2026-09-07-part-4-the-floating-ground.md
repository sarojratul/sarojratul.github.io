---
layout: series-post
title: "Part 4: Troubleshooting a Panel That Would Not Light"
date: 2026-09-07 12:00:00 -0600
series: turn-signal
part: 4
covers: "6 to 7 September 2026"
excerpt: "The simulation worked and the panel did not. A troubleshooting ladder, a teardown to one segment, a floating ground no voltmeter could see, a blown meter fuse, and segment A verified."
permalink: /turn-signal/part-4-floating-ground/
---

Continues from [Part 3](/turn-signal/part-3-soldering-and-panel-build/), where every gate network measured correctly.

<details class="toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## Symptom: nothing lights

<p class="entry-date">6 September 2026</p>

Rail connected, 3.3 V on a gate, every gate network correct, and no segment lights. The pack measured **5.35 V** open-circuit, about 1.34 V per cell, which is a normal fully charged NiMH pack that sags toward 4.8 V under load. It was noted and ruled out.

<!-- Photo wanted: the finished panel, powered, with nothing lit. -->

### Troubleshooting ladder

Rather than guessing at causes, I wrote ordered measurements working **outward from the source**, each with an expected value and the meaning of a failure:

| # | Measure | Expected | A failure means |
|---|---|---|---|
| 1 | Rail voltage at the **breadboard's own** + and − holes, not the battery terminals | ~5.3–5.4 V | The battery leads are not making contact |
| 2 | Rail to each segment's string-anode bus | ~5.3–5.4 V | A broken jumper from rail to bus |
| 3 | MOSFET **gate pin** to ground with 3.3 V driven | ~3.3 V | Open 220 Ω path, or the source is not enabled |
| 4 | **V<sub>DS</sub>** with the gate high | a few mV (LTspice: **2.38 mV**) | MOSFET not on: gate wire on the wrong pin, or source-to-ground missing |
| 5 | One string's current, meter in series between resistor bus and string | ~25 mA | 0 mA with correct V<sub>DS</sub> means a reversed LED or an open string |

Step 1 deliberately measures at the breadboard holes: measuring at the battery only proves the battery is fine.

### Initial suspects

1. **Battery holder leads.** Bare, untinned stranded copper. Stranded wire can fray inside a breadboard hole and miss the contact, or splay and short the next row, while looking connected.
2. **LED polarity.** The same symptom as the [Wokwi reversed-LED bug](/turn-signal/part-2-simulation/#bug-2-leds-wired-backwards).
3. **A gate wire on the wrong pin.** The adapter routing ([Part 3](/turn-signal/part-3-soldering-and-panel-build/#verifying-the-mosfets-with-diode-mode)) gives no physical clue to pin identity; a gate wire on the drain would leave every other measurement unchanged.

## Reducing the problem to one segment

<p class="entry-date">Night of 6 September 2026</p>

The decision overnight: **strip the panel to one segment.** Sixteen strings put 32 LED orientations, 16 resistor placements and four gate networks in the suspect pool at once, on a board too dense to probe easily. One segment is three to five strings, one MOSFET and one gate network. If it lights, the topology is proven on hardware and any remaining fault is wiring in a known-good design; if not, the whole ladder can be walked in ten minutes. It is the same principle as the Tinkercad rehearsal: separate "the circuit is wrong" from "a leg is in the wrong hole".

## Root cause: a floating ground

<p class="entry-date">7 September 2026</p>

**Method.** I removed the wiring from every segment except **segment A** (ten LEDs, five strings of two), leaving the other LEDs physically in the board, and replaced long flying jumpers with short wires so every connection was visible. The rail was verified three ways:

| Measured at | Instrument | Reading |
|---|---|---|
| Battery pack leads | Multimeter | 5.44 V |
| Far end of the breadboard + rail | Multimeter | 5.43 V |
| Breadboard + rail | Analog Discovery 2 | 5.42 V |

Three instruments, three points, 10 mV of spread. With 3.3 V on the gate resistor, still nothing.

**Result.** The AD2's ground was not tied to the battery pack's negative rail. One wire between them and the segment lit immediately.

**Explanation.** The panel rail came from the AA pack and the gate drive from the AD2, with no shared reference. The "3.3 V" on the gate was relative to the AD2's internal ground, not the MOSFET source, so V<sub>GS</sub> was undefined and the MOSFET never turned on. **Every voltage measurement still read correctly,** because each instrument measured against its own reference. A ladder built only from voltage measurements could not catch this.

> **Rule added: rung 0.** Before any voltage measurement, check continuity between the grounds of every instrument and supply in the setup. The original ladder stays as written; that it missed this fault is the useful result.

Simulation cannot show this class of fault either. LTspice, Wokwi and Tinkercad all give the schematic one implicit ground, so unconnected supplies never appear unless modelled deliberately.

![Segment A lit on the breadboard](/images/segment_a_lit.jpg)

*Segment A lit: ten LEDs in five strings, each carrying 24.5 to 25.8 mA. The dark cluster beside it is segment D, populated but unwired. The AD2 supplies the 3.3 V gate drive, and the fix is the wire tying its ground to the pack's negative rail.*
{: .caption}

It is not certain whether that ground was connected the night before, while all four segments were still wired; the fault could have been elsewhere then. The record does not show it either way.

## Blown multimeter fuse

<p class="entry-date">7 September 2026</p>

**Objective.** Measure each string's current directly against the 24.78 mA design value.

**Method.** Break the circuit at one string's anode and insert the multimeter in series on the 200 mA range.

**Result.** No reading, and the string went dark. Continuity tested fine and the leads were in the correct jack. The string lit through a plain jumper and went dark with the meter inserted, so the path through the meter was open.

Inside the meter are two 5×20 mm ceramic fuses in clips: **F500mAH/600V** and **F10AH/600V**. The 500 mA fuse read OL out of circuit. It protects the 2000 µA, 20 mA and 200 mA ranges through the shared VΩmA jack, so all three were dead; the 10 A range, on its own jack and fuse, still worked.

**Cause.** An ammeter is a near short and must be **in series**, inserted into a break. Touching the probes across two points of an intact circuit on a current range shorts whatever lies between them through the shunt. That most likely happened during the gate-current check on 6 September. The range selector on a current setting is not only about display digits: it chooses which shunt and which fuse carry the current.

![The two fuses inside the multimeter](/images/multimeter_fuses.jpg)

*Inside the meter: F10AH/600V at top right and F500mAH/600V at centre, both 5×20 mm ceramic in clips. The 500 mA fuse, which reads OL, sits behind both the 322.9 µA gate check and the 25 mA string check.*
{: .caption}

**Replacement:** a 500 mA, 600 V, 5×20 mm fast-acting **ceramic** fuse, not a 250 V glass one that happens to fit, because the ceramic body is what quenches the arc when a fuse clears at high voltage. Bought locally, since DigiKey shipping to Saskatoon turns a five-dollar fuse into twenty. Lesson for the next instrument: read its manual before use, not as problems arise.

## Measuring current across the string resistors

<p class="entry-date">7 September 2026</p>

With every low current range dead, the 33 Ω resistor already in each string served as a current shunt. Measure the voltage across it on a live, fully lit circuit and apply I = V<sub>R</sub> / R. At the design current:

24.78 mA × 33 Ω = **0.818 V**

This works on a live circuit, breaks nothing, cannot blow a fuse, and covers all five strings in about a minute. The gate check could have been done the same way, reading across the 10 kΩ pulldown and dividing by 10,000. (In EE 221 the equivalent was the oscilloscope's math channel, dividing a measured voltage by a known resistance.)

## Segment A verified

<p class="entry-date">7 September 2026</p>

**Objective.** Confirm segment A is correct, not just lit: each string at design current and the MOSFET fully on.

**Method.** Voltage across each 33 Ω resistor, live, on the 2 V and 20 V ranges; then V<sub>DS</sub> with the gate at 3.3 V on the 200 mV range.

| Measurement | Predicted | Measured | Verdict |
|---|---|---|---|
| Voltage across one 33 Ω string resistor | 0.818 V | 0.81 to 0.85 V | Pass |
| Implied string current | 24.78 mA | 24.5 to 25.8 mA | Pass |
| V<sub>DS</sub> with gate high | 2.38 mV (LTspice, 3-string segment) | 3.3 mV | Pass |
| Implied R<sub>DS(on)</sub> | 24 mΩ typical (datasheet) | 3.3 mV / 124 mA ≈ 27 mΩ | Pass |

**The spread across strings has a cause.** The string nearest the supply reads 0.85 V and the farthest 0.81 V: IR drop along the breadboard rail, as each string draws current out of it. About 5% end to end, harmless here, and expected to tighten on perfboard with soldered copper rails. On the breadboard each string also got its own rail jumper rather than a per-segment anode bus; electrically identical, but sixteen jumpers with all segments in. The perfboard layout should use four buses.

**What this proves:** segment A's topology, components, gate drive and MOSFET work at real currents, and the hand calculation for a five-string arrowhead (never simulated) holds. The 27 mΩ result near the 24 mΩ typical shows the MOSFET fully enhanced, not in its linear region. **What it does not prove:** anything about the other three segments, thermals over long duty, or behaviour with all four segments drawing 396.5 mA.

![One of the four surviving MOSFET adapters](/images/mosfet_adapter_working.jpg)

*The adapter switching segment A, one of four survivors after nine were destroyed. Flux smudges and cotton fibres from alcohol cleaning are visible; it measured about 27 mΩ R<sub>DS(on)</sub> against 24 mΩ typical, which is good enough for breadboard prototyping.*
{: .caption}

**Next:** rewire segments B, C and D, run the hand test (each segment lights alone with the others dark), then all four together. That resumed two weeks later in [Part 5](/turn-signal/part-5-full-panel/).
