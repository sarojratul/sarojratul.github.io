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

With all four segments proven under injected gate signals in [Part 5](/turn-signal/part-5-full-panel/), the microcontroller takes over.

<details class="toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## GPIO 2 is a strapping pin

<p class="entry-date">20 September 2026</p>

**Objective.** B7 step 1: check the locked GPIO 2/4/5/6 assignment against the documentation before wiring.

**Method.** Compared each pin with Espressif's ESP32-C3 hardware design guidelines and the Seeed XIAO ESP32-C3 pinout.

**Result.** GPIO 2 is one of the ESP32-C3's three strapping pins (with GPIO 8 and GPIO 9), and Espressif recommends pulling it **up** "due to glitches". The design put a permanent 10 kΩ **pulldown** on it through segment A's gate network.

**Action.** That risked unreliable boot, a fault that would only appear once the panel was wired. Segment A moved to **GPIO 3**, adjacent, unused and not a strapping pin. **GPIO 3, 4, 5, 6** is the final pin set for segments A to D, and the firmware and build guide were updated before any wire touched the board.

## Headers, flashing and a ground loop

<p class="entry-date">20 September 2026</p>

**Objective.** Fit headers to both ESP32-C3 boards (the panel receiver, and the handlebar transmitter for B8) and flash the receiver.

**Method.** Soldered all 14 header pins on each board, with female sockets on hand so the boards can later sit on perfboard without resoldering.

![Both ESP32-C3 boards with all 14 header pins soldered](/images/esp32c3_boards_soldered.jpg)

*Both Seeed XIAO ESP32-C3 boards, one for the panel receiver and one for the handlebar transmitter. A few joints are imperfect but adequate for prototyping, a clear improvement on the [SOT-23 adapters](/turn-signal/part-3-soldering-and-panel-build/#learning-sot-23-soldering).*
{: .caption}

**Flashing faults, in order:**

1. **Greyed-out "esp32" boards in Arduino IDE.** The ESP32 board package was not installed; fixed by installing "esp32 by Espressif Systems" in Boards Manager.
2. **A COM port that appeared for a few seconds, then vanished,** on every cable and port tried. With cables and ports eliminated one at a time, the remaining variable was the ground wire from the ESP32-C3 to the panel. Disconnecting it, letting the port enumerate and selecting it, then reconnecting ground, worked reliably.

**Cause.** A ground loop: the USB-grounded laptop and the mains-earthed bench supply were each referenced to their own outlet and also joined through the panel's ground rail, which disrupted USB enumeration at plug-in. It is a bench artifact and disappears once the panel runs from its own battery with nothing else on its ground.

## B7: firmware-driven sweep in both directions

<p class="entry-date">20 September 2026</p>

**Objective.** With GPIO 3/4/5/6 wired to the gate networks, confirm the firmware sweeps in the correct direction, draws the expected current, and boots cleanly.

**Method.** Flashed a build with the radio stubbed to a fixed direction (B7 step 3), so the sweep runs without the transmitter or an ESP-NOW link.

**Result.** LEFT: C, then C+B, then C+B+A, toward the left arrowhead.

![The panel sweeping through its lit sequence, wired to the ESP32-C3 for the first time](/images/direction_sweep_flash.gif)

*The receiver's own GPIOs driving the gate networks, with no injected gate signal. The smaller breadboard below carries the ESP32-C3 boards and their wiring to the panel.*
{: .caption}

RIGHT: B, then B+C, then B+C+D, mirrored correctly. The supply read a peak of **0.187 A** during the three-segment hold against about **186 mA** predicted from the individual segment currents, so B7's separate per-string current check was not repeated. A power cycle with the panel connected lit no stray segment at boot.

**Conclusion.** B7 passes: correct topology, direction logic both ways, current accounted for, clean boot. The state machine and timing were drawn up once the timing stopped changing:

![The receiver's activation state machine: OFF, HELD and TRAILING, with the any-press-cancels and link-timeout failsafe paths](/images/state_machine_diagram.png)

*Three states: OFF (dark), HELD (sweeping, direction latched while a button is down) and TRAILING (counting down the four cycles after release). Any press from either state restarts HELD in the pressed direction. The dashed amber path is the link-timeout failsafe: if ESP-NOW is quiet for more than 1000 ms, the panel goes dark rather than freezing lit.*
{: .caption}

![Timing diagram of a LEFT sweep over two 1688 ms cycles, showing segments A, B and C joining centre-out and holding, with no blank tail](/images/sweep_timing_diagram.png)

*Segments join centre-out and accumulate (C, C+B, C+B+A) and stay lit until the cycle wraps straight back to C with no blank gap. The shaded band is the 788 ms HOLD_MS phase. Values are those confirmed on the scope below.*
{: .caption}

## Sweep timing: tuned by eye, verified on a scope

<p class="entry-date">20 September 2026</p>

**Objective.** Settle `HOLD_MS`, open since the Wokwi build, by watching the real panel.

**Method.** Flash a value, watch, adjust, reflash, aiming only for an arrow readable at a glance.

| Round | STEP_MS | HOLD_MS | CYCLE_MS | Observation |
|---|---|---|---|---|
| 1 | 120 | none | 800 | Original values. Too fast to follow. |
| 2 | 300 | 350 | 1000 | Added a distinct hold. Still quick. |
| 3 | 600 | 700 | 2000 | Everything 2× slower. Overshot. |
| 4 | 450 | 525 | 1425 | 1.5× instead of 2×; the blank tail removed so the sweep restarts straight from the first segment. |
| 5 | 450 | **788** | **1688** | Hold extended another 1.5×, cycle following. Final. |

Timing judged by eye is not a measurement, so the final values went on the AD2 scope: two channels on adjacent segment nodes, cursors on the edges.

| Quantity | Configured | Measured on the scope |
|---|---|---|
| STEP_MS | 450 ms | 446.9 ms |
| HOLD_MS | 788 ms | about 781 ms (derived from a 1228 ms segment on-time) |
| CYCLE_MS | 1688 ms | 1689 ms |

**Conclusion.** All three are within about 1% of the configured values, so the `millis()`-based timing does what the firmware asks. The speed may still change after a few days of fresh eyes.

**Next:** B8, the second ESP32-C3 as handlebar transmitter, and the ESP-NOW link. The plan beyond that is in [Part 7](/turn-signal/part-7-whats-next/).
