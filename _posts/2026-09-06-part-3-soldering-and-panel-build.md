---
layout: series-post
title: "Part 3: Soldering, Panel Build and First Tests"
date: 2026-09-06 12:00:00 -0600
series: turn-signal
part: 3
covers: "Early September to 6 September 2026"
excerpt: "Nine MOSFETs lost learning SOT-23 soldering, a diode-mode check that beats the silkscreen, 32 LEDs on a breadboard, and a gate test that passed while proving less than it seemed to."
permalink: /turn-signal/part-3-soldering-and-panel-build/
redirect_from:
  - /2026/09/20/physical-bring-up-and-debugging.html
---

With the simulations done ([Part 2](/turn-signal/part-2-simulation/)), the parts went onto a real board.

<details class="toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## Learning SOT-23 soldering

<p class="entry-date">Early September 2026</p>

Four AO3400As had to be soldered onto SOT-23-to-DIP adapters, with header pins in each adapter. It was my first surface-mount work and close to my first soldering of any kind, and it went badly: three small burns, a mark on the table, and then an iron that **stopped taking solder at all**. The causes, all at once:

- The solder supplied with the iron was poor quality.
- I was not using flux.
- I did not know tip tinner existed.
- The cleaning sponge was being used **dry**, which destroys a tip quickly.

The tip oxidised until it would not transfer heat. By the time I recognised a fault rather than inexperience, **nine MOSFETs and their adapters were ruined.**

![The retinned soldering tip beside a brass wool ball and spare amber LEDs](/images/retinned_tip.jpg)

*The tip after restoration (there is no photo of it at its worst). Hours of flux and fresh solder brought it back to wetting properly. The brass wool beside it is what should have been used from the start; the loose LEDs were practice parts.*
{: .caption}

What fixed it, in order of importance:

1. **Wet sponge, or better, brass wool,** which cleans without shock-cooling the tip.
2. **Keep the tip tinned.** An untinned hot tip oxidises in air and stops transferring heat.
3. **Flux is required on small joints;** it lets solder wet the pad instead of balling up.
4. **A tip can be restored** with flux and fresh solder alone, given patience.

After re-tinning, I practised on the nine ruined parts, then **soldered five MOSFETs, of which four worked**, the number needed.

![An AO3400A soldered onto a SOT-23 to DIP adapter](/images/mosfet_adapter_soldered.jpg)

*The first one soldered. The source pad joint is not properly wetted, and testing later showed the MOSFET had been cooked by too much heat for too long. It is one of the nine casualties: "looks soldered" and "is soldered" are different claims, and only the diode test settles which.*
{: .caption}

<!-- Photo wanted: all four surviving adapters side by side with their header pins in. -->

### Verifying the MOSFETs with diode mode

The adapter's silkscreen cannot be trusted for pinout: the adapter presents two pins on one side and one on the other, while the SOT-23 has three in a row, so the routing hides which pin is which. Diode mode on the assembled part identifies it empirically:

- **The body diode exists only from Source (anode) to Drain (cathode).** Red on Source and black on Drain reads a diode drop; reversed reads OL.
- **A reading in both directions** means a solder bridge or a dead part.
- **Gate to either other pin must read OL both ways.** Any continuity means a bridge or a blown gate.

The test was repeated at the header pins after they were fitted; any difference from the bare adapter means a cold joint or a broken trace. All four survivors passed.

## Building the panel

<p class="entry-date">6 September 2026</p>

![LED rows seated in the breadboard, seen from a low angle](/images/led_rows_angled.jpg)

*Each string is two LEDs in series, each pair straddling two columns. Nothing is wired yet, so a reversed LED here stays invisible until the panel refuses to light.*
{: .caption}

![Resistors and the four MOSFET adapters going in, partway through the build](/images/breadboard_full.jpg)

*Partway through, with the 33 Ω resistors and the four AO3400A adapters placed. The rails are not powered and nothing has been tested.*
{: .caption}

On the board:

- 16 strings, each two amber LEDs in series with a 33 Ω resistor
- Grouped 5 / 3 / 3 / 5 onto four segment buses (A, B, C, D, left to right)
- Each bus to its own AO3400A drain; all sources to a shared ground rail
- Per segment: 220 Ω from the GPIO to the gate, 10 kΩ from gate to ground
- 470 µF electrolytic and 0.1 µF ceramic decoupling

Breadboard rules worked out during the build:

- Each 5-hole column is one node. A part's two legs go in **different** columns; two parts are in series by **sharing** one.
- The centre channel only matters for DIP packages; a discrete LED string does not need to straddle it.
- A segment bus is two adjacent columns bridged by a jumper (ten holes, one node), holding the five resistor legs and the wire to the MOSFET drain.
- Leave an empty column between strings so a stray leg cannot short two of them.
- **Power rails are sometimes split mid-board with no marking.** Continuity-check before relying on them.
- The 470 µF is polarised: long leg to +4.8 V, striped leg to ground, placed where the supply enters the rail. The 0.1 µF goes across the microcontroller's 3.3 V and ground, at the chip.
- **LED cathode = flat side = short leg.** Anode toward +. Only the MOSFET *source* goes to ground; the *drain* goes to the segment's resistor bus.

On 4.8 V, strings stay at two LEDs however the LEDs are arranged visually; four in series would need about 8 V.

## Test 1: gate networks pass

<p class="entry-date">6 September 2026</p>

**Objective.** Confirm each gate network is intact and correctly placed.

**Predicted.** 3.3 V ÷ (220 Ω + 10,000 Ω) = 3.3 / 10,220 = **322.9 µA**.

**Method.** Multimeter in series, 2000 µA range.

| Segment | Gate current |
|---|---|
| A | 316 µA |
| B | 318 µA |
| C | 319.5 µA |
| D | 319.5 µA |

**Result: all four pass,** within about 2% of prediction, a spread explained by resistor tolerance. Both the 220 Ω and the 10 kΩ are present and correctly landed on every segment.

### Analog Discovery 2 notes

I first tried to read this current from the AD2's Supplies tool. **It has no per-channel current readout.** The row showing USB voltage, USB current, AUX voltage, AUX current and temperature is the **System Monitor**: the AD2's own draw from USB, about 340 mA idle.

![WaveForms Supplies tab next to the Analog Discovery 2 and the breadboard](/images/ad2_waveforms.jpg)

*Positive Supply at 3.3 V and on, with the row beneath reading USB Voltage 4.820 V, USB Current 343.0 mA. That is the AD2's own consumption, not the circuit's.*
{: .caption}

- In **Wavegen DC mode, Offset is the output level.** Amplitude is greyed out by design. Set Offset = 3.3 V, then press that channel's own **Run**; the Supplies Master Enable does not start Wavegen.
- **V− is not a second positive rail.** It is negative relative to the AD2's ground and cannot be stacked with V+ to make 4.8 V. The AD2 has one positive supply.

The multimeter in series answered the question in about a minute. The AD2 is the right tool for timing and for finding *where* a signal stops; for a 300 µA DC reading, the simpler instrument is better.

## A fault the gate test cannot detect

<p class="entry-date">6 September 2026</p>

Walking each MOSFET source back to the ground rail with the continuity test found **one source with no ground connection.**

> **The 322.9 µA gate check does not detect a missing source ground.** Gate current flows through the 220 Ω and 10 kΩ to ground regardless of the source connection, so a segment can pass the gate test with a floating source that will never conduct. A passing test proves only what it measured.

The same day, with every gate network correct, the panel still would not light. That is [Part 4](/turn-signal/part-4-floating-ground/).
