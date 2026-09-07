---
layout: post
title: "Physical Bring-Up and the Debugging Log"
date: 2026-09-06
categories: bike-turn-signals
---

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
| 5 | Current in one string: meter in series between the resistor bus and the string | ~25 mA | 0 mA with a correct low V<sub>DS</sub> means an LED backwards, or an open string |

Note that step 1 measures at the **breadboard rail holes**, not the battery terminals. That distinction is the whole point of the ladder: measuring at the source tells you the source is fine, which is not the question.

### Prime suspects

**1. The battery holder leads.** They are bare twisted stranded copper, not tinned, no ferrules, no pins. Stranded wire frays inside a breadboard hole and misses the contact entirely, or splays out and shorts an adjacent row. It looks connected. It is the least clever possible failure and therefore the first one to check.

**2. LED polarity.** The identical failure mode, everything measures right and nothing lights, is exactly the Wokwi bug from a week ago, where all four LEDs were reversed. Flat side = cathode = short leg = toward ground and the MOSFET drain side.

**3. A gate wire on the wrong pin.** Because of the adapter re-routing described earlier, the physical pin arrangement gives no clue about which pin is which. If a gate wire landed on the drain, everything else would still measure exactly as it does.

---

## 7 September 2026: Current plan, tear it down to one segment

Sitting with it overnight, here is where I have landed:

**I am going to remove all the segments and rebuild one.** Debugging densely-packed LEDs and wiring is the problem. 16 strings on one board means 32 LED orientations, 16 resistor placements and four gate networks all in the suspect pool at once, in a space where I cannot easily get a probe in.

One segment is three strings, one MOSFET, one gate network. If one segment lights, the topology is proven on real hardware and everything after that is a wiring fault in a *known-good* design, which is a much smaller search. If one segment does *not* light, I have a circuit small enough to walk the entire ladder on in ten minutes.

This is the same instinct as the Tinkercad decision: **separate "the circuit is wrong" from "a leg is in the wrong hole" before trying to answer either.** The difference is that this time I am doing it on the bench instead of in a simulator.

### The resolution

**Open.** I will update this section when I find it.

I am deliberately not editing this part of the log once I know the answer. Whatever it turns out to be, the answer will look obvious in hindsight (they all do), and the useful part of this page is the ladder I built while I still did not know.

---
