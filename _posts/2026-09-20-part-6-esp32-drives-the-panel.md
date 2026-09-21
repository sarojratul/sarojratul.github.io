---
layout: series-post
title: "Part 6: The ESP32-C3 Drives the Arrow"
date: 2026-09-20 18:00:00 -0600
series: turn-signal
part: 6
covers: "20 September 2026"
excerpt: "A strapping pin caught in the datasheet, a ground loop that looked like a dead board, the arrow sweeping both ways under firmware control, and the timing tuned by eye then checked on a scope."
permalink: /turn-signal/part-6-esp32-drives-the-panel/
---

With all four segments proven under bench-injected gate signals in [Part 5](/turn-signal/part-5-full-panel/), the next step was to let the microcontroller drive them. This entry covers the pin check that changed a "locked" decision, getting the boards soldered and flashing, the first firmware-driven sweep, and settling the animation timing.

<details class="toc" markdown="1">
<summary>In this entry</summary>

* TOC
{:toc}

</details>

## A GPIO pin the datasheet caught before I did

<p class="entry-date">20 September 2026</p>

**Objective.** Before wiring the ESP32-C3 into the panel for the first time, B7 step 1 calls for checking the locked GPIO 2/4/5/6 pin assignment against the board's actual datasheet, rather than trusting a pin set that had been decided weeks earlier.

**Method.** Checked GPIO 2, 4, 5 and 6 against Espressif's own ESP32-C3 hardware design guidelines and the Seeed XIAO ESP32-C3 pinout diagram.

**Result.** GPIO2 is one of the ESP32-C3's three strapping pins, along with GPIO8 and GPIO9, and Espressif's documentation explicitly recommends pulling it up, not down, "due to glitches." The panel's design put a permanent 10 kΩ pulldown on GPIO2 for segment A's gate network, exactly the opposite of that guidance.

**What it means.** A board that might not boot reliably, in a failure mode that would only show up once the panel was actually wired in, since none of the gate testing done up to this point had the ESP32-C3 anywhere near that pin. Moved segment A off GPIO2 onto GPIO3, the adjacent pin, unused and not a strapping pin. GPIO 3, 4, 5, 6 is now the final, locked pin set for segments A, B, C, D, and the firmware and build guide were both updated before any wire touched the board. Same instinct that found the Wokwi LEDs wired backwards and identified the MOSFET pinout empirically instead of trusting the adapter's silkscreen: check a part's own documentation before trusting an assumption, however locked that assumption was supposed to be.

---

## Headers, and a ground loop that looked like a dead board

<p class="entry-date">20 September 2026</p>

**Objective.** Get header pins onto both ESP32-C3 boards, the panel receiver and the handlebar transmitter built ahead for B8, and get the receiver flashing the real firmware for the first time.

**Method.** Soldered all 14 header pins on each board, using female header sockets already on hand so the boards can sit on perfboard later without resoldering.

**Result.**

![Both ESP32-C3 boards with all 14 header pins soldered](/images/esp32c3_boards_soldered.jpg)

*Both Seeed XIAO ESP32-C3 boards done, one for the panel receiver, one held for the handlebar transmitter.*
{: .caption}

Soldering went much better than the MOSFET adapters did. A few pins still are not very well soldered, but they are good enough for prototyping, and after [nine ruined SOT-23 adapters](/turn-signal/part-3-soldering-and-panel-build/#the-soldering-disaster) earlier in the month, that is a fair standard to hold a 0.1 inch through-hole header to.

Flashing came with its own ladder. Arduino IDE's board search showed nothing but greyed-out entries for "esp32," because the ESP32 board package had never been installed in Boards Manager, fixed by installing "esp32 by Espressif Systems" from there directly. The next problem was different: plugging the board in showed a USB COM port for a few seconds, then it vanished, every time, boards list still greyed out. Different cables, different ports, same result.

The usual suggestions for a port that appears and vanishes are a broken cable, an overloaded port and so on. I did not think that was it, but I worked through them one at a time anyway: various ports and cable combinations, same result. If it was an electrical fault and the ports and cables were fine, the only thing left was the ground wire connecting the ESP32-C3 to the panel. I tested that, and it worked.

Disconnecting the ESP32-C3's ground wire from the panel's shared GND rail, plugging in, letting the port enumerate and selecting it, then reconnecting ground afterward, worked cleanly and stayed working.

**What it means.** Not a short and not damaged hardware. A ground loop: the USB-grounded laptop and the separately earthed bench supply were both referenced to their own wall outlets while also tied together through the panel's GND rail, and that loop was enough to disrupt enumeration on plug-in. It is a bench-setup artifact specifically, caused by a mains-grounded instrument and a USB-grounded computer sharing one rail, and it will not exist once the panel runs off its own battery pack with nothing else sharing its ground.

---

## B7: the arrow sweeps in both directions

<p class="entry-date">20 September 2026</p>

**Objective.** With GPIO 3, 4, 5, 6 wired to the four gate networks for the first time, confirm the receiver firmware actually drives the panel the way it is supposed to: sweeping the correct direction, current where predicted, a clean boot.

**Method.** Flashed a build with the radio stubbed to a hard-coded direction, per B7 step 3, so the sweep runs continuously without needing the transmitter board or a live ESP-NOW link yet.

**Result.** With the direction set to LEFT: segment C alone, then C with B, then C with B with A, sweeping toward the left arrowhead as designed.

![The panel sweeping through its lit sequence, wired to the ESP32-C3 for the first time](/images/direction_sweep_flash.gif)

*The receiver's own GPIOs driving the gate networks directly, no bench-injected gate signal this time. The second, smaller breadboard below carries the ESP32-C3 boards and their wiring to the panel.*
{: .caption}

Reflashed with the direction set to RIGHT: B alone, then B with C, then B with C with D, mirrored correctly toward the right arrowhead. The bench supply's display read a peak of 0.187 A during the three-segment hold, against roughly 186 mA predicted from the individually measured segment currents, close enough that B7's per-string current check was skipped rather than repeated with an inline meter on top of that cross-check. Power-cycled the board with the panel still connected: no stray segment lit at boot.

**What it means.** B7 passes in full: correct topology, correct direction logic both ways, current accounted for, clean boot. The activation state machine (OFF, HELD, TRAILING) and the segment timing it produces got written up as a diagram once this closed out, since the timing itself kept changing for the rest of the day and a diagram that goes stale is worse than no diagram at all.

![The receiver's activation state machine: OFF, HELD and TRAILING, with the any-press-cancels and link-timeout failsafe paths](/images/state_machine_diagram.png)

*Three states cover the whole activation model: OFF (panel dark), HELD (sweeping, direction latched while a button is down), and TRAILING (counting down the four cycles played after release). Any press, from either state, cancels immediately and restarts HELD in the newly pressed direction. The dashed amber path is the link-timeout failsafe: if ESP-NOW goes quiet for more than 1000 ms, the panel forces itself dark rather than freezing lit.*
{: .caption}

![Timing diagram of a LEFT sweep over two 1688 ms cycles, showing segments A, B and C joining center-out and holding, with no blank tail](/images/sweep_timing_diagram.png)

*Segments join center-out and accumulate, C alone, then C+B, then C+B+A, and stay lit together until the cycle wraps straight back to C with no blank gap. The shaded band is the 788 ms HOLD_MS phase where the full arrow is lit. Drawn from the same STEP_MS/HOLD_MS/CYCLE_MS values confirmed on the AD2 scope in the next section.*
{: .caption}

---

## Tuning the sweep by eye, then checking it with a scope

<p class="entry-date">20 September 2026</p>

**Objective.** HOLD_MS, how long the full three-segment arrow stays lit before the cycle repeats, had been an open question since the Wokwi build in early September. With the real hardware finally flashing, settle it by watching the actual panel instead of guessing at a number in advance.

**Method.** Live and iterative: flash a guess, watch it, change the constant, reflash. No target number going in beyond wanting it readable at a glance.

**Result.** Five rounds, each one a reflash:

| Round | STEP_MS | HOLD_MS | CYCLE_MS | What I saw |
|---|---|---|---|---|
| 1 | 120 | none | 800 | The original values. Too fast to follow. |
| 2 | 300 | 350 | 1000 | Added a distinct hold phase. Still felt quick. |
| 3 | 600 | 700 | 2000 | Everything 2× slower. Overshot. |
| 4 | 450 | 525 | 1425 | 1.5× instead of 2×, and the blank tail at the end of the cycle cut, so the sweep restarts straight from the first segment instead of going dark for a beat. |
| 5 | 450 | **788** | **1688** | The full-arrow hold needed to last longer: HOLD_MS up another 1.5×, CYCLE_MS following it. Final. |

I wanted it to be noticeable, and the first configuration was too fast for that. I might still change the speed if, after a few days, it looks wrong to a fresh pair of eyes.

The work plan's own B7 instructions warn that judging timing by eye is not a measurement you can put in a table, so before calling it settled, the final numbers went on the Analog Discovery 2's scope: two channels on adjacent segment nodes, cursors dropped directly on the edges, reading the time delta straight off the display. STEP_MS measured 446.9 ms against a configured 450 ms. A second cursor pair across one segment's full on-time, from the start of the hold phase to the next cycle's restart, measured 1228 ms, which combined with the STEP_MS reading gives a derived HOLD_MS of about 781 ms against the configured 788 ms. A third pair across one full cycle, rise to rise, measured 1689 ms against a configured 1688 ms.

| Quantity | Configured | Measured on the scope |
|---|---|---|
| STEP_MS | 450 ms | 446.9 ms |
| HOLD_MS | 788 ms | about 781 ms (derived from a 1228 ms on-time) |
| CYCLE_MS | 1688 ms | 1689 ms |

**What it means.** Every one of those landed within about 1% of what the firmware was actually asked to do. The by-eye tuning and the microcontroller's own millis()-based timing agree with each other, not just with the sound of "close enough."

**Next:** B8, the second ESP32-C3 as the handlebar transmitter, and the ESP-NOW link between them. What comes after that is in [Part 7](/turn-signal/part-7-whats-next/).
