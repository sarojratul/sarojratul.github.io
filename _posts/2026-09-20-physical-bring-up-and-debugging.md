---
layout: post
title: "Physical Bring-Up and the Debugging Log"
date: 2026-09-20
---

# Part 3: Physical Bring-Up and the Debugging Log

## Early September 2026: The soldering disaster

Before any of the breadboard work could happen, four AO3400As had to get onto SOT-23-to-DIP adapters, and header pins had to go into those adapters.

**First time doing surface mount.** First time doing much soldering at all, really.

It went badly.

The pads are tiny and the pitch demands a precision I did not have. I burned my hands in three places and left a mark on the wooden table. And then things got worse in a way I did not understand at the time: **the iron stopped taking solder at all.**

What I did not know, all at once:

- The solder that came with the iron was bad solder.
- I did not know what flux was for.
- I did not know what tip tinner was.
- I did not know the cleaning sponge is supposed to be **wet**. I had been using it dry, which is the fastest way there is to destroy a tip.

The tip oxidised heavily and became completely unusable. By the time I understood that something was actually wrong rather than that I was just bad at this, I had **ruined nine MOSFETs and their adapters.**

That is when panic set in, and then research mode.

![The retinned soldering tip beside a brass wool ball and spare amber LEDs](/images/retinned_tip.jpg)

*I have no photo of the tip at its worst, which I regret, because the difference would tell the story on its own. This is after. Hours of flux and fresh solder brought that tip back to something that wets properly, and the brass wool sitting next to it is the thing I should have had from the start. Loose amber LEDs in frame because those were the practice targets.*

**What I learned, in the order it mattered:**

1. **Wet sponge, always.** Brass wool balls are better than a sponge, because they do not shock-cool the tip.
2. **Oxidation is the enemy.** A tip that is not tinned oxidises the moment it is hot and in air, and once oxidised it will not transfer heat.
3. **Flux** is not optional on small joints. It is what lets the solder wet the pad instead of balling up.
4. **Tip tinner exists**, but you can restore a tip with flux and fresh solder alone if you are patient, which is what I did rather than waiting on another order.

I spent hours re-tinning that heavily oxidised tip. Then, once the brass wool and flux arrived, I practised on the ruined MOSFETs and LEDs. The nine dead parts turned out to be the best thing I could have had, because I could not make them any worse.

**Then I soldered five. Four came out working.**

I needed four.

![An AO3400A soldered onto a SOT-23 to DIP adapter](/images/mosfet_adapter_soldered.jpg)

*The very first one I soldered, and I was proud of it. Look at the source pad: the joint is not properly wetted. Testing later showed I had cooked the MOSFET with too much heat and too long on the pad. Task failed successfully.*

*This is casualty number one of nine. The lesson underneath it is that "it looks soldered" and "it is soldered" are different claims, and only the diode test settles which one you have.*

*Photo wanted: all four surviving adapters side by side with their header pins in.*

### How I verified them: the multimeter test that beats the silkscreen

You cannot trust the adapter board's silkscreen. My adapters present as **two pins on one side and one on the other**, while the SOT-23 part itself is three pins in a row. The adapter re-routes them, so the physical arrangement tells you nothing about which pin is which.

Diode mode, on the assembled part:

- The **body diode exists only between Source (anode) and Drain (cathode).** Red on Source, black on Drain gives a diode drop. Reverse the leads and you get OL.
- **A reading in both directions means a solder bridge or a dead part.**
- **Gate to either other pin should read OL both ways.** Any continuity at all means a gate bridge or a blown gate.

That identifies the pinout empirically, which is the only trustworthy way. I re-ran it at the header pins after soldering those in: the results must be identical, and a difference means a cold joint or a broken adapter trace.

This is straight out of EE 221's "read the datasheet, then verify the part in front of you" habit, and it is the reason I know the four survivors are actually good.

---

## 6 September 2026: Building the panel

![LED rows seated in the breadboard, seen from a low angle](/images/led_rows_angled.jpg)

*Every string is two LEDs in series, and every pair has to straddle two different columns. At this stage nothing is wired yet, so a single LED in backwards here is invisible until the whole panel refuses to light.*

![Resistors and the four MOSFET adapters going in, partway through the build](/images/breadboard_full.jpg)

*Partway through, with the 33 ohm resistors and the four AO3400A adapters placed. The rails are not powered in this shot. Nothing here has been tested yet.*

**What went on the board:**

- 16 strings, each 2 amber LEDs in series with a 33 Ω resistor
- Grouped 5 / 3 / 3 / 5 onto four segment buses (A, B, C, D, left to right)
- Each bus to its own AO3400A drain; all four sources to a shared ground rail
- Per segment: 220 Ω from the GPIO to the gate, 10 kΩ from gate to ground
- 470 µF electrolytic + 0.1 µF ceramic decoupling

Breadboard mechanics I had to work out and wrote down so I would not re-derive them:

- Each 5-hole column is one node. A component's two legs must land in **different** columns; two parts are in series by **sharing** a column.
- The centre divider only isolates the two halves of a row, for DIP chips. A discrete LED string does not need to straddle it.
- To join five string outputs into one segment bus: bridge two adjacent columns with a jumper so ten holes act as one node, then land the five resistor legs plus the wire to the MOSFET drain among them. One bus per segment.
- Leave at least one empty column between strings so a stray leg cannot short two of them.
- **Rails are sometimes split into disconnected halves mid-board with no marking.** Continuity-check them before assuming.
- The 470 µF is polarised: long leg to +4.8 V, striped short leg to ground, placed right where the supply leads enter the rail. The 0.1 µF is not polarised and goes across the microcontroller's own 3.3 V and ground, right at the chip.
- **LED polarity: the flat side of the plastic body is the cathode, and it is the short leg.** Long leg (anode) goes toward +. Only the MOSFET's *source* goes to the ground rail; the *drain* goes to that segment's resistor bus.

**Note for anyone reading this to copy it:** on 4.8 V you cannot put four LEDs in series. That needs about 8 V. Strings stay at two LEDs regardless of how the LEDs are physically arranged into the arrow shape. The electrical grouping and the visual grouping are two different things.

---

## 6 September 2026: Test 1, the gate networks. PASS.

**Objective:** confirm every gate network is intact and correctly landed before trusting any of them.

**Predicted:** 3.3 V ÷ (220 Ω + 10,000 Ω) = 3.3 / 10,220 = **322.9 µA**

**Method:** multimeter in series on the 2000 µA range. (It reads `000` with the leads open and jumps when inserted, so you know immediately whether you are actually in the circuit.)

**Measured:**

| Segment | Gate current |
|---|---|
| A | 316 µA |
| B | 318 µA |
| C | 319.5 µA |
| D | 319.5 µA |

**All four PASS.** Every reading is within about 2% of predicted, and resistor tolerance fully explains the spread. Both the 220 Ω series resistor and the 10 kΩ pulldown are present and correctly landed on all four segments.

### The AD2 detour, and why the boring instrument won

I went to the Analog Discovery 2 for this measurement first, looking for a per-channel output current readout on the Supplies tool.

**There isn't one.** The row showing `USB Voltage / USB Current / AUX Voltage / AUX Current / Temperature` is the **System Monitor**, which is the AD2's own draw from the USB port (about 340 mA idle). It tells you nothing about your circuit. I spent a real amount of time hunting for a display that does not exist, partly because I was told it did.

![WaveForms Supplies tab next to the Analog Discovery 2 and the breadboard](/images/ad2_waveforms.jpg)

*The screen that cost me the time. Positive Supply set to 3.3 V and on, and the row underneath reading USB Voltage 4.820 V, USB Current 343.0 mA. That is the System Monitor reporting the AD2's own consumption from the USB port. It is not my circuit's current, and there is no per-channel readout anywhere on this tab.*

Other WaveForms things I learned the hard way, recorded so I do not repeat them:

- In **Wavegen DC mode, the Offset field is the output level.** Amplitude is greyed out and that is correct, not a fault. Set Offset = 3.3 V, then hit that channel's own **Run**. The Master Enable on the Supplies tab does not start Wavegen.
- **V− is not a second positive rail.** It is negative with respect to the AD2's own ground and cannot be stacked with V+ to make 4.8 V. (I went down this path trying to power the whole thing from the AD2. The AD2 has exactly one positive supply. The batteries were charged the whole time.)

**The multimeter in series answered the question in about a minute.** I have written "prefer the simple instrument for DC measurements" into my project notes in bold. The AD2 is a brilliant tool for *timing* and for finding *where* in a circuit a signal dies, which is what I bought it for and what EE 221 taught me to use a scope for, and a poor tool for reading 300 µA.

---

## 6 September 2026: The bug the gate test could not catch

While working through the segments I found **a missing ground connection on one MOSFET's source**, caught with the **continuity test** on the multimeter, walking each source pin back to the ground rail.

This one is important and I flagged it in my notes as a warning to myself:

> **The 322.9 µA gate check does NOT catch a missing source ground.** Gate current flows through the 220 Ω and the 10 kΩ to ground; it is completely independent of whether the source has a path to ground. A segment can pass the gate test perfectly and still be a MOSFET with a floating source, which will never conduct.

A test that passes tells you exactly what it tested, and nothing more. Writing down what a test does *not* cover is as useful as writing down what it does.

---

## 6 September 2026: The wall

**The simulation worked. The physical LEDs are not turning on.**

Rail connected. 3.3 V driven onto a gate. Every gate network measuring correctly. Nothing lights, on any segment.

One thing I checked immediately and ruled out: the battery pack measures **5.35 V** open-circuit. That is a normal fully-charged NiMH pack, about 1.34 V per cell, not a fault. It sags toward 4.8 V under load. I noted it so that I would not treat a healthy battery as a suspect later.

*Photo wanted: the finished panel, powered, with nothing lit. The shot below is from partway through the build, not from the failure.*

### The troubleshooting ladder

Rather than listing possible causes and guessing, I wrote an ordered set of measurements working **outward from the source**, each with an expected value and the meaning of a failure. This is the EE 232 lab-report instinct applied to a real bench, and it is the part of this project I am most confident is correct even though the circuit is not working yet.

| # | Measure | Expected | A failure here means |
|---|---|---|---|
| 1 | Rail voltage at the **breadboard's own** + and − rail holes (not the battery terminals) | ~5.3–5.4 V | The battery leads are not making contact |
| 2 | Rail to each segment's string-anode bus | same ~5.3–5.4 V | A broken jumper from the rail to that bus |
| 3 | MOSFET **gate pin** to ground, with 3.3 V driven | ~3.3 V | Open 220 Ω path, or the source is not actually enabled |
| 4 | **V<sub>DS</sub>** (drain to source) with the gate high | a few mV (LTspice: **2.38 mV**) | MOSFET not turning on: gate wire on the wrong pin, or source-to-ground missing |
| 5 | Current in one string: meter in series between the resistor bus and the string | <sub>25 mA | 0 mA with a correct low V</sub>DS~ means an LED backwards, or an open string |

Note that step 1 measures at the **breadboard rail holes**, not the battery terminals. That distinction is the whole point of the ladder: measuring at the source tells you the source is fine, which is not the question.

### Prime suspects

**1. The battery holder leads.** They are bare twisted stranded copper, not tinned, no ferrules, no pins. Stranded wire frays inside a breadboard hole and misses the contact entirely, or splays out and shorts an adjacent row. It looks connected. It is the least clever possible failure and therefore the first one to check.

**2. LED polarity.** The identical failure mode, everything measures right and nothing lights, is exactly the Wokwi bug from a week ago, where all four LEDs were reversed. Flat side = cathode = short leg = toward ground and the MOSFET drain side.

**3. A gate wire on the wrong pin.** Because of the adapter re-routing described earlier, the physical pin arrangement gives no clue about which pin is which. If a gate wire landed on the drain, everything else would still measure exactly as it does.

---

## 6 September 2026: Current plan, tear it down to one segment

Sitting with it overnight, here is where I have landed:

**I am going to remove all the segments and rebuild one.** Debugging densely-packed LEDs and wiring is the problem. 16 strings on one board means 32 LED orientations, 16 resistor placements and four gate networks all in the suspect pool at once, in a space where I cannot easily get a probe in.

One segment is three strings, one MOSFET, one gate network. If one segment lights, the topology is proven on real hardware and everything after that is a wiring fault in a *known-good* design, which is a much smaller search. If one segment does *not* light, I have a circuit small enough to walk the entire ladder on in ten minutes.

This is the same instinct as the Tinkercad decision: **separate "the circuit is wrong" from "a leg is in the wrong hole" before trying to answer either.** The difference is that this time I am doing it on the bench instead of in a simulator.

### The resolution

**The Analog Discovery 2 had no ground connection to the battery pack.** Found on 7 September, written up in full below, because the fix belongs to the day I found it and not to the day I wrote the ladder.

The ladder above stays exactly as it was. It did not find the fault. That is the useful part.

---

## 7 September 2026: One segment, and the wire that was not there

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

**What it means.** The two supplies were floating with respect to each other. The panel rail came from the 4xAA pack. The 3.3 V gate drive came from the AD2. With no shared ground, "3.3 V" on the gate was 3.3 V relative to the AD2's own internal reference, not relative to the MOSFET's source. V<sub>GS</sub> was undefined, so the MOSFET never turned on.

The part worth writing down: **every voltage measurement still read correctly.** Probe the gate, 3.3 V. Probe the rail, 5.4 V. Each instrument measured against its own reference and reported a perfectly sensible number. A floating-ground fault is invisible to voltage probing, which is exactly why a ladder built entirely out of voltage measurements was never going to catch it.

**The ladder needed a rung 0: continuity between the grounds of every instrument in the setup, checked before any voltage measurement at all.** That is the rule I am taking out of this.

It is also a fault that simulation structurally cannot show you. LTspice, Wokwi and Tinkercad all give the whole schematic one implicit shared ground. There is no way for "these two supplies are not tied together" to appear in a simulator unless you deliberately set out to model it. The Tinkercad rehearsal that proved the topology could never have predicted this one.

![Segment A lit on the breadboard](/images/segment_a_lit.jpg)

*Segment A lit: ten LEDs in five parallel strings of two, each string carrying 24.5 to 25.8 mA. The dark cluster beside it is segment D, still fully populated but with its wiring removed, left in the board because rebuilding ten LEDs into an arrow shape is slow work. The Analog Discovery 2 beside the breadboard supplies the 3.3 V gate drive, and the wire that made all of this work is the one tying its ground to the battery pack's negative rail.*

What I actually thought, standing there: I could not remember whether I had connected that ground the night before or not. Maybe I did, and the fault was somewhere else while all four segments were still wired in. Maybe I never did. I do not know, and I am not going to pretend otherwise in my own log.

The other thing I noticed is how the idea arrived. Stopping work the previous night is what gave me the tear-it-down-to-one-segment idea, and it turned up while I was doing something else entirely. Grounding the AD2 arrived the same way, a day later. I do not think I would have reached it at 1 a.m. with the board still full. A night off did more than another hour of staring.

---

## 7 September 2026: I blew the multimeter's fuse, and found out what the dial actually does

**Objective.** Measure the current in each string directly and confirm the segment draws what the design predicts, 24.78 mA per string.

**Method.** Break the circuit at one string's anode, insert the multimeter in series, 200 mA range.

**Result.** No reading, and the string went dark.

The elimination took a while. Continuity tested fine. The leads were in the correct jack and had never been anywhere else. The string lit normally through a plain jumper and went dark the moment the meter was inserted. That combination means one thing: the path through the meter is open.

I opened the meter. Two ceramic 5x20 mm fuses, both in clips rather than soldered: **F500mAH/600V** and **F10AH/600V**. Out of the meter, probes on the end caps, the 500 mA one reads OL. Blown. It feeds the 2000 uA, 20 mA and 200 mA ranges through the shared VOhmmA jack, which is why all three died together. The 10 A range, on its own jack and its own fuse, still works.

**What it means.** An ammeter is a near short circuit, and it has to be **in series**, inserted into a break in the circuit. Touching two probes to two points of an intact circuit with a current range selected shorts whatever sits between those two points through the meter's shunt. I am fairly confident that is what killed it, during the gate-current check on 6 September.

My mind and the multimeter's fuse were blown by the same simple fact.

I had assumed the dial's many ranges existed only because the display cannot show enough digits otherwise. That is roughly true for the voltage ranges. It is not the whole story for the current ranges, where the selection also decides which shunt and which fuse the current runs through. I had measured voltage and resistance plenty of times before this project and assumed the rest would be easy learning along the way. It was learning along the way. It was not easy.

![The two fuses inside the multimeter](/images/multimeter_fuses.jpg)

*Inside the meter. Both fuses sit in clips: F10AH/600V at the top right of the board, F500mAH/600V at the centre, both 5x20 mm ceramic. The 500 mA one reads OL out of circuit. It is the fuse behind the 322.9 uA gate-network check from 6 September and behind tonight's 25 mA string check, which is why losing it took out both.*

**What I would do differently:** read the manual properly before starting, instead of picking it up as the need arises. That goes double for the next meter I buy.

Replacement is a 500 mA, 600 V, 5x20 mm fast-acting ceramic fuse. Not a 250 V glass one, even though it fits the clips: the 600 V ceramic body is what quenches the arc when a fuse clears at high voltage, and this fuse lives in the instrument I point at things I have not characterised yet. DigiKey's shipping to Saskatoon turns a five dollar fuse into a twenty dollar one, so this is a local purchase.

---

## 7 September 2026: Measuring current without an ammeter

With every current range dead, I still needed the string currents. Every string already has a 33 Ohm resistor in it, and a known resistor carrying an unknown current is a current shunt.

Ohm's law, run backwards:

I = V<sub>R</sub> / R

Probe the two ends of the 33 Ohm resistor with the circuit fully powered and lit, nothing disconnected, meter on DC volts. At the design current:

24.78 mA x 33 Ohm = **0.818 V**

This is better than the inline ammeter in nearly every way that matters here. It works on a live circuit. It does not require breaking anything. It cannot blow a fuse. And it can be done on all five strings in about a minute instead of five rewires. The gate-network check on 6 September could have been done exactly the same way, reading across the 10 kOhm pulldown and dividing by 10,000.

It was new to me. In EE 221 we used the oscilloscope's math channel, dividing a measured voltage by a known resistance to plot current against time. If we ever did the multimeter-in-series version, it was rare enough that I have no memory of it. Making the series connection made complete sense afterwards, once the fuse had explained it.

---

## 7 September 2026: Segment A verified

**Objective.** Confirm the working segment is not merely lit but correct: every string at its design current, and the MOSFET fully on rather than partly on.

**Method.** Voltage across each 33 Ohm string resistor, live, on the 2 V and 20 V ranges. Then V<sub>DS</sub> across the MOSFET with the gate held at 3.3 V, on the 200 mV range.

**Result.**

| Measurement | Predicted | Measured | Verdict |
|---|---|---|---|
| Voltage across one 33 Ohm string resistor | 0.818 V | 0.81 to 0.85 V | PASS |
| Implied string current | 24.78 mA | 24.5 to 25.8 mA | PASS |
| V<sub>DS</sub> with gate high | 2.38 mV (LTspice, 3-string segment) | 3.3 mV | PASS |
| Implied R<sub>DS(on)</sub> | 24 mOhm typical (datasheet) | 3.3 mV / 124 mA, about 27 mOhm | PASS |

**The spread across the five strings is real and it has a cause.** The string nearest the supply reads 0.85 V, the one farthest reads 0.81 V. That is IR drop along the breadboard's own power rail. Each string taps current out of the rail as you move along it, so strings farther from the supply see slightly less voltage and draw slightly less current. About 5% end to end. Harmless here, and it should tighten considerably on the perfboard build, where the rails are soldered copper rather than a row of spring contacts.

A related note for that build: on the breadboard I ran a separate rail jumper to each string's first anode instead of building one anode bus per segment. Electrically identical, fine for a bench test, but it means sixteen rail jumpers once all four segments are in. The perfboard layout should collapse those back into four buses.

**What this does and does not prove.** It proves segment A's topology, components, gate drive and MOSFET all work on real hardware at real currents, and it validates the hand calculation for a five-string arrowhead, which was never built in LTspice. The 27 mOhm figure landing close to the AO3400A's 24 mOhm typical is good evidence the part is fully enhanced and not sitting somewhere in its linear region. It proves nothing about the other three segments, nothing about thermals over a long duty cycle, and nothing about what happens when all four segments pull 396.5 mA at once.

![One of the four surviving MOSFET adapters](/images/mosfet_adapter_working.jpg)

*One of the four surviving AO3400A adapters, the one switching segment A. Nine MOSFETs and adapters were destroyed learning to solder SOT-23 before this one came out working. The smudges are flux, and the fibres are cotton from cleaning it with alcohol. It measured about 27 mOhm of R<sub>DS(on)</sub> tonight against the datasheet's 24 mOhm typical.*

Far from professional looking. Imperfect soldering, flux and alcohol smudges, cotton fibres. But if it works, I am happy using it for breadboard prototyping.

**Next:** rewire segments B, C and D, then run the hand test. Each segment must light alone under 3.3 V on its gate with all the others dark. Then all four together, watching the pack sag under 396.5 mA.

---

## 20 September 2026: A bench supply, and learning to set its current limit

**Objective.** Get a proper adjustable bench supply into the rail-power role the 4xAA NiMH pack had been filling, and confirm it is set up correctly before trusting it with the panel.

**Method.** The supply is a Jesverty SPS-6005N, 0 to 60 V, 0 to 5 A, single channel, bought for a different project entirely and pressed into service here because it can hold an exact voltage and cap current in a way four AA cells never could. The multimeter's replacement fuse also went in today, a straight clip swap, no iron needed, and the ammeter ranges came back with it.

The SPS-6005N has no dedicated button for setting the current limit. The manual's own procedure is to dial the voltage to target first with the output open, then short the positive and negative terminals together with a spare wire, then turn the current knob until the display shows the desired limit while that short forces the supply into constant-current mode, then remove the short. Voltage first, current second, in that order, because the limit-setting trick only reads correctly once the real target voltage is already dialed in.

**Result.** First attempt: dialed 4.80 V, then set the current knob without paying close enough attention to the short-and-set order, and ended up with the limit set too low. Symptom: with segment A connected and lit, the display read 4.10 V and 0.081 A, current pinned flat while voltage sagged well below the 4.80 V target. That is the textbook constant-current crossover signature from the manual's own CV/CC description: once current hits the set limit, the supply holds current and lets voltage collapse to whatever the load demands.

Redid it properly. Shorted the leads again, turned the current knob up, and confirmed on the display: 0.450 A, 0.07 V, 0.031 W while shorted (0.07 x 0.45 is 0.0315, so the numbers agree with each other, this is not a fault, it is just what a dead short looks like in CC mode). Removed the short, reconnected the segment, and the rail held at 4.80 V under load from then on.

**What it means.** The supply now does what the AA pack did, hold a rail near 4.8 V, but with an actual current ceiling instead of hoping nothing goes wrong, and with a front-panel ammeter as a second, independent way to read current alongside the multimeter. The AD2's ground still has to be tied to the supply's negative terminal, the exact same non-negotiable rule as with the pack. The supply's own grounding terminal (green, separate from the +/- output) was left unconnected, since it is chassis/earth ground and mixing it with the AD2's USB-referenced ground risked a ground loop neither instrument would report cleanly.

---

## 20 September 2026: Segment D, wired from scratch

**Objective.** Repeat the 7 September segment A bring-up on segment D, the other five-string arrowhead, this time on the new supply.

**Method.** Segment D's parts had been sitting populated but unwired in the breadboard since early September, the same "leave it in place, don't repopulate" approach used throughout Stage B5. Wired its five string anodes to a bus, the 220 Ohm and 10 kOhm gate network, the MOSFET source to the shared GND rail and drain to the string bus, same conventions as every other segment on this board. Supply's black lead to the GND rail, red lead to D's bus. AD2 ground tied to the supply's black terminal. Gate injection at the input side of the 220 Ohm, same as before, with the AD2's Wavegen set to DC offset 3.3 V.

One instrument quirk turned up while getting the gate signal working: stopping Wavegen's Run does not reliably drive its output to 0 V. The LEDs stayed lit with Run stopped, as long as the output pin was still physically in the breadboard, which meant the channel was holding its last driven level rather than going to zero or high impedance. Setting the offset itself to 0 V, or just unplugging the lead and letting the 10 kOhm pulldown do its job, both worked. Worth remembering for B and C.

**Result.** Segment D lit. Ten LEDs, five strings, visually no different from segment A.

**What it means.** The topology, gate network and MOSFET assembly for D are physically correct. Whether the currents are actually right is a separate question, and the numbers said something more interesting than a clean pass.

---

## 20 September 2026: Segment D's current came up short, and why I stopped chasing it

**Objective.** Confirm segment D is not just lit but correct, to the same standard segment A was held to on 7 September: every string at its design current, the MOSFET fully on.

**Method.** Inline ammeter, now that the fuse is fixed, on the 200 mA range, in series between the string-resistor bus and the MOSFET drain. Cross-checked against the supply's own front-panel current display. Then voltage across each of the five 33 Ohm string resistors individually, live, nothing disconnected. Then V<sub>DS</sub> across the MOSFET with the gate held high, on the 200 mV range. Then, once the numbers did not match the design target, voltage across each LED in one string, and a full reseat of that string's connections to rule out a bad contact.

**Result.**

| Measurement | Predicted | Measured | Verdict |
|---|---|---|---|
| Total segment current | 123.9 mA (5 x 24.78 mA) | 80 to 89 mA (inline ammeter and supply display agreed) | short of target |
| Voltage across one 33 Ohm string resistor | 0.818 V | 0.59 V, uniform across all five strings | short of target |
| Implied string current | 24.78 mA | 17.9 mA | short of target |
| Rail voltage under load | 4.80 V | 4.80 V, steady | PASS |
| V<sub>DS</sub> with gate high | a few mV (segment A read 3.3 mV) | 2.2 mV | PASS |
| Implied R<sub>DS(on)</sub> | 24 mOhm typical (datasheet) | 2.2 mV / 89 mA, about 25 mOhm | PASS |
| Voltage across one LED | ~1.99 V (design assumption) | 2.08 V, both LEDs in the checked string agreed | higher than assumed |

**My first thought was the breadboard wiring**, since D had sat populated but disconnected for two weeks and a marginal contact seemed like the obvious explanation for a segment drawing less than it should. The ladder ruled that out step by step instead of confirming it. The rail held at a steady 4.80 V under load, so the supply was not the cause. V<sub>DS</sub> came back at 2.2 mV, in the same range as segment A's healthy 3.3 mV reading, so the MOSFET was fully on, not sitting somewhere in its linear region. All five string resistors read the identical 0.59 V, which rules out one bad string dragging the total down and instead points at something affecting every string the same way. Reseating the checked string's LEDs and resistor changed nothing.

The number that closed it: 2.08 V across each LED, both LEDs in the string agreeing with each other, and 2.08 + 2.08 + 0.59 adds up to 4.75 V, which is the 4.80 V rail within the kind of small loss you get from probe and contact resistance. Nothing is unaccounted for. The extra voltage the LEDs are taking, compared to the 1.99 V figure the design calculations assumed, is exactly and only where the missing current went.

Then the real cross-check: segment A, still wired exactly as it was left on 7 September and untouched since, powered up on the same new supply and showed the same shortfall, roughly 80 mA instead of its original 124 mA. Two segments, two completely different wiring histories, the same number. That is stronger evidence than anything on D alone that this is not a D-specific fault.

**What it means.** It proves D's topology, gate drive and MOSFET all work correctly, the same conclusion segment A earned on 7 September, just at a lower operating point than either segment showed back then. It does not prove anything about B or C yet, but it sets the expectation that they will likely land under their 74.3 mA design figure too, and that alone should not be read as a fault when it happens.

It is a genuinely odd result on its own terms. LED forward voltage normally drops a little at lower current, not rises, and 17.9 mA is lower than the 24.78 mA both segments ran at on 7 September. Something changed between then and now, the LEDs themselves, the different supply, two weeks sitting in a breadboard, and I do not have a clean answer for which. I decided not to keep chasing it, since 2.08 V is still comfortably inside the Lumex SSL-LX5093AD's datasheet window of 2.0 V typical to 2.5 V maximum, nothing here is out of spec, and the whole voltage budget for the string is fully accounted for. The original 123.9 mA and 74.3 mA numbers from the design calculations now read as a ceiling from an idealized 1.99 V LED model, not as an exact bench prediction.

![Segment D lit under the new bench supply](/images/segment_d_bench_supply.jpg)

*Segment D lit, powered by the Jesverty SPS-6005N instead of the AA pack for the first time. The display reads 4.82 V, 0.089 A, 0.428 W, matching the sum of the five string currents from the resistor measurements to within rounding. The unlit LEDs and MOSFET adapters elsewhere on the board belong to segments B and C, populated but not yet wired.*

I suspected the wiring first, mostly because of the two weeks off the bench. Once the numbers came back low but self-consistent rather than erratic, it stopped feeling like a fault to chase, and the brightness looked no different from what I remembered from the last session, which in hindsight is its own small lesson: a 30 percent current difference does not necessarily look like anything to the eye. The meter is the instrument that catches it, not the LED.

**Next:** wire and test segments B and C the same way, on the new supply, expecting the same lower-than-design currents rather than treating it as a surprise a second time. Then the four-segment hand test, and the all-four-lit measurement, against whatever the real total turns out to be rather than the original 396.5 mA figure.

---
