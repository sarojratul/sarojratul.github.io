---
layout: series-post
title: "Part 4: Nothing Lights, and the Wire That Was Not There"
date: 2026-09-07 12:00:00 -0600
series: turn-signal
part: 4
covers: "6 to 7 September 2026"
excerpt: "The simulation worked and the panel did not. A troubleshooting ladder, a teardown to one segment, a floating ground no voltmeter could see, a blown meter fuse, and segment A finally verified."
permalink: /turn-signal/part-4-floating-ground/
---

At the end of [Part 3](/turn-signal/part-3-soldering-and-panel-build/) every gate network measured correctly. This entry is what happened when the panel still would not light: a troubleshooting ladder, a teardown, the actual cause (which no voltage measurement could have shown), a blown meter fuse along the way, and finally a segment verified to the numbers.

<details class="toc" markdown="1">
<summary>In this entry</summary>

* TOC
{:toc}

</details>

## The wall: nothing lights

<p class="entry-date">6 September 2026</p>

**The simulation worked. The physical LEDs are not turning on.**

Rail connected. 3.3 V driven onto a gate. Every gate network measuring correctly. Nothing lights, on any segment.

One thing I checked immediately and ruled out: the battery pack measures **5.35 V** open-circuit. That is a normal fully-charged NiMH pack, about 1.34 V per cell, not a fault. It sags toward 4.8 V under load. I noted it so that I would not treat a healthy battery as a suspect later.

<!-- Photo wanted: the finished panel, powered, with nothing lit. -->

### The troubleshooting ladder

Rather than listing possible causes and guessing, I wrote an ordered set of measurements working **outward from the source**, each with an expected value and the meaning of a failure. This is the EE 232 lab-report instinct applied to a real bench, and it is the part of this project I am most confident is correct even though the circuit is not working yet.

| # | Measure | Expected | A failure here means |
|---|---|---|---|
| 1 | Rail voltage at the **breadboard's own** + and − rail holes (not the battery terminals) | ~5.3–5.4 V | The battery leads are not making contact |
| 2 | Rail to each segment's string-anode bus | same ~5.3–5.4 V | A broken jumper from the rail to that bus |
| 3 | MOSFET **gate pin** to ground, with 3.3 V driven | ~3.3 V | Open 220 Ω path, or the source is not actually enabled |
| 4 | **V<sub>DS</sub>** (drain to source) with the gate high | a few mV (LTspice: **2.38 mV**) | MOSFET not turning on: gate wire on the wrong pin, or source-to-ground missing |
| 5 | Current in one string: meter in series between the resistor bus and the string | ~25 mA | 0 mA with a correct low V<sub>DS</sub> means an LED backwards, or an open string |

Note that step 1 measures at the **breadboard rail holes**, not the battery terminals. That distinction is the whole point of the ladder: measuring at the source tells you the source is fine, which is not the question.

### Prime suspects

**1. The battery holder leads.** They are bare twisted stranded copper, not tinned, no ferrules, no pins. Stranded wire frays inside a breadboard hole and misses the contact entirely, or splays out and shorts an adjacent row. It looks connected. It is the least clever possible failure and therefore the first one to check.

**2. LED polarity.** The identical failure mode, everything measures right and nothing lights, is exactly the [Wokwi bug from a week ago](/turn-signal/part-2-simulation/#bug-2-all-four-leds-were-wired-backwards), where all four LEDs were reversed. Flat side = cathode = short leg = toward ground and the MOSFET drain side.

**3. A gate wire on the wrong pin.** Because of the adapter re-routing described in [Part 3](/turn-signal/part-3-soldering-and-panel-build/#how-i-verified-them-the-multimeter-test-that-beats-the-silkscreen), the physical pin arrangement gives no clue about which pin is which. If a gate wire landed on the drain, everything else would still measure exactly as it does.

---

## Tearing it down to one segment

<p class="entry-date">Night of 6 September 2026</p>

Sitting with it overnight, here is where I have landed:

**I am going to remove all the segments and rebuild one.** Debugging densely-packed LEDs and wiring is the problem. 16 strings on one board means 32 LED orientations, 16 resistor placements and four gate networks all in the suspect pool at once, in a space where I cannot easily get a probe in.

One segment is three strings, one MOSFET, one gate network. If one segment lights, the topology is proven on real hardware and everything after that is a wiring fault in a *known-good* design, which is a much smaller search. If one segment does *not* light, I have a circuit small enough to walk the entire ladder on in ten minutes.

This is the same instinct as the Tinkercad decision: **separate "the circuit is wrong" from "a leg is in the wrong hole" before trying to answer either.** The difference is that this time I am doing it on the bench instead of in a simulator.

---

## One segment, and the wire that was not there

<p class="entry-date">7 September 2026</p>

**Objective.** Execute the teardown decided the previous night: strip the panel to a single segment, and either light it or walk the whole ladder on a circuit small enough to see.

**Method.** I removed the wiring from every segment except one arrowhead, **segment A**, ten LEDs in five parallel strings of two. I left the LEDs themselves in the board. Rewiring ten LEDs into an arrow shape on a breadboard is slow and fiddly, and I did not want to do it twice. I also replaced the long flying jumpers with short wire pieces so I could actually see what was connected to what.

Then I verified the rail three separate ways before touching anything downstream:

| Measured at | Instrument | Reading |
|---|---|---|
| Battery pack leads | Multimeter | 5.44 V |
| Far end of the breadboard + rail | Multimeter | 5.43 V |
| Breadboard + rail | Analog Discovery 2 | 5.42 V |

Three instruments, three points, 10 mV of spread. The rail was not the problem.

I applied 3.3 V to the gate resistor. Nothing.

**Result.** Then it occurred to me that the AD2 needs a ground after all. Its ground was not tied to the battery pack's negative rail. I put one wire between them and the segment lit immediately.

**What it means.** The two supplies were floating with respect to each other. The panel rail came from the 4×AA pack. The 3.3 V gate drive came from the AD2. With no shared ground, "3.3 V" on the gate was 3.3 V relative to the AD2's own internal reference, not relative to the MOSFET's source. V<sub>GS</sub> was undefined, so the MOSFET never turned on.

The part worth writing down: **every voltage measurement still read correctly.** Probe the gate, 3.3 V. Probe the rail, 5.4 V. Each instrument measured against its own reference and reported a perfectly sensible number. A floating-ground fault is invisible to voltage probing, which is exactly why a ladder built entirely out of voltage measurements was never going to catch it.

**The ladder needed a rung 0: continuity between the grounds of every instrument in the setup, checked before any voltage measurement at all.** That is the rule I am taking out of this. The ladder itself stays exactly as I wrote it the night before. It did not find the fault, and that is the useful part.

It is also a fault that simulation structurally cannot show you. LTspice, Wokwi and Tinkercad all give the whole schematic one implicit shared ground. There is no way for "these two supplies are not tied together" to appear in a simulator unless you deliberately set out to model it. The Tinkercad rehearsal that proved the topology could never have predicted this one.

![Segment A lit on the breadboard](/images/segment_a_lit.jpg)

*Segment A lit: ten LEDs in five parallel strings of two, each string carrying 24.5 to 25.8 mA. The dark cluster beside it is segment D, still fully populated but with its wiring removed, left in the board because rebuilding ten LEDs into an arrow shape is slow work. The Analog Discovery 2 beside the breadboard supplies the 3.3 V gate drive, and the wire that made all of this work is the one tying its ground to the battery pack's negative rail.*
{: .caption}

What I actually thought, standing there: I could not remember whether I had connected that ground the night before or not. Maybe I did, and the fault was somewhere else while all four segments were still wired in. Maybe I never did. I do not know, and I am not going to pretend otherwise in my own log.

The other thing I noticed is how the idea arrived. Stopping work the previous night is what gave me the tear-it-down-to-one-segment idea, and it turned up while I was doing something else entirely. Grounding the AD2 arrived the same way, a day later. I do not think I would have reached it at 1 a.m. with the board still full. A night off did more than another hour of staring.

---

## I blew the multimeter's fuse, and found out what the dial actually does

<p class="entry-date">7 September 2026</p>

**Objective.** Measure the current in each string directly and confirm the segment draws what the design predicts, 24.78 mA per string.

**Method.** Break the circuit at one string's anode, insert the multimeter in series, 200 mA range.

**Result.** No reading, and the string went dark.

The elimination took a while. Continuity tested fine. The leads were in the correct jack and had never been anywhere else. The string lit normally through a plain jumper and went dark the moment the meter was inserted. That combination means one thing: the path through the meter is open.

I opened the meter. Two ceramic 5×20 mm fuses, both in clips rather than soldered: **F500mAH/600V** and **F10AH/600V**. Out of the meter, probes on the end caps, the 500 mA one reads OL. Blown. It feeds the 2000 µA, 20 mA and 200 mA ranges through the shared VOhmmA jack, which is why all three died together. The 10 A range, on its own jack and its own fuse, still works.

**What it means.** An ammeter is a near short circuit, and it has to be **in series**, inserted into a break in the circuit. Touching two probes to two points of an intact circuit with a current range selected shorts whatever sits between those two points through the meter's shunt. I am fairly confident that is what killed it, during the gate-current check on 6 September.

My mind and the multimeter's fuse were blown by the same simple fact.

I had assumed the dial's many ranges existed only because the display cannot show enough digits otherwise. That is roughly true for the voltage ranges. It is not the whole story for the current ranges, where the selection also decides which shunt and which fuse the current runs through. I had measured voltage and resistance plenty of times before this project and assumed the rest would be easy learning along the way. It was learning along the way. It was not easy.

![The two fuses inside the multimeter](/images/multimeter_fuses.jpg)

*Inside the meter. Both fuses sit in clips: F10AH/600V at the top right of the board, F500mAH/600V at the centre, both 5×20 mm ceramic. The 500 mA one reads OL out of circuit. It is the fuse behind the 322.9 µA gate-network check from 6 September and behind tonight's 25 mA string check, which is why losing it took out both.*
{: .caption}

**What I would do differently:** read the manual properly before starting, instead of picking it up as the need arises. That goes double for the next meter I buy.

Replacement is a 500 mA, 600 V, 5×20 mm fast-acting ceramic fuse. Not a 250 V glass one, even though it fits the clips: the 600 V ceramic body is what quenches the arc when a fuse clears at high voltage, and this fuse lives in the instrument I point at things I have not characterised yet. DigiKey's shipping to Saskatoon turns a five dollar fuse into a twenty dollar one, so this is a local purchase.

---

## Measuring current without an ammeter

<p class="entry-date">7 September 2026</p>

With every current range dead, I still needed the string currents. Every string already has a 33 Ω resistor in it, and a known resistor carrying an unknown current is a current shunt.

Ohm's law, run backwards:

I = V<sub>R</sub> / R

Probe the two ends of the 33 Ω resistor with the circuit fully powered and lit, nothing disconnected, meter on DC volts. At the design current:

24.78 mA x 33 Ω = **0.818 V**

This is better than the inline ammeter in nearly every way that matters here. It works on a live circuit. It does not require breaking anything. It cannot blow a fuse. And it can be done on all five strings in about a minute instead of five rewires. The gate-network check on 6 September could have been done exactly the same way, reading across the 10 kΩ pulldown and dividing by 10,000.

It was new to me. In EE 221 we used the oscilloscope's math channel, dividing a measured voltage by a known resistance to plot current against time. If we ever did the multimeter-in-series version, it was rare enough that I have no memory of it. Making the series connection made complete sense afterwards, once the fuse had explained it.

---

## Segment A verified

<p class="entry-date">7 September 2026</p>

**Objective.** Confirm the working segment is not merely lit but correct: every string at its design current, and the MOSFET fully on rather than partly on.

**Method.** Voltage across each 33 Ω string resistor, live, on the 2 V and 20 V ranges. Then V<sub>DS</sub> across the MOSFET with the gate held at 3.3 V, on the 200 mV range.

**Result.**

| Measurement | Predicted | Measured | Verdict |
|---|---|---|---|
| Voltage across one 33 Ω string resistor | 0.818 V | 0.81 to 0.85 V | PASS |
| Implied string current | 24.78 mA | 24.5 to 25.8 mA | PASS |
| V<sub>DS</sub> with gate high | 2.38 mV (LTspice, 3-string segment) | 3.3 mV | PASS |
| Implied R<sub>DS(on)</sub> | 24 mΩ typical (datasheet) | 3.3 mV / 124 mA, about 27 mΩ | PASS |

**The spread across the five strings is real and it has a cause.** The string nearest the supply reads 0.85 V, the one farthest reads 0.81 V. That is IR drop along the breadboard's own power rail. Each string taps current out of the rail as you move along it, so strings farther from the supply see slightly less voltage and draw slightly less current. About 5% end to end. Harmless here, and it should tighten considerably on the perfboard build, where the rails are soldered copper rather than a row of spring contacts.

A related note for that build: on the breadboard I ran a separate rail jumper to each string's first anode instead of building one anode bus per segment. Electrically identical, fine for a bench test, but it means sixteen rail jumpers once all four segments are in. The perfboard layout should collapse those back into four buses.

**What this does and does not prove.** It proves segment A's topology, components, gate drive and MOSFET all work on real hardware at real currents, and it validates the hand calculation for a five-string arrowhead, which was never built in LTspice. The 27 mΩ figure landing close to the AO3400A's 24 mΩ typical is good evidence the part is fully enhanced and not sitting somewhere in its linear region. It proves nothing about the other three segments, nothing about thermals over a long duty cycle, and nothing about what happens when all four segments pull 396.5 mA at once.

![One of the four surviving MOSFET adapters](/images/mosfet_adapter_working.jpg)

*One of the four surviving AO3400A adapters, the one switching segment A. Nine MOSFETs and adapters were destroyed learning to solder SOT-23 before this one came out working. The smudges are flux, and the fibres are cotton from cleaning it with alcohol. It measured about 27 mΩ of R<sub>DS(on)</sub> tonight against the datasheet's 24 mΩ typical.*
{: .caption}

Far from professional looking. Imperfect soldering, flux and alcohol smudges, cotton fibres. But if it works, I am happy using it for breadboard prototyping.

**Next:** rewire segments B, C and D, then run the hand test. Each segment must light alone under 3.3 V on its gate with all the others dark. Then all four together, watching the pack sag under 396.5 mA. (That picked up two weeks later, in [Part 5](/turn-signal/part-5-full-panel/).)
