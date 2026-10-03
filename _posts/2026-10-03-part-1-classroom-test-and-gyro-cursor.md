---
layout: series-post
title: "Part 1: Classroom PC Test and a First Gyro Cursor"
date: 2026-10-03 12:00:00 -0600
series: comet-air-mouse
part: 1
covers: "3 October 2026"
excerpt: "The Bluetooth test repeated on a classroom PC, the MPU-6050 wired to the XIAO, and four rounds of firmware turning gyro rate into cursor motion, ending with gravity-compensated pointing and an open drift problem."
permalink: /comet-air-mouse/part-1-classroom-test-and-gyro-cursor/
---

This entry continues from [Part 0](/comet-air-mouse/part-0-presenter-design-and-ble-test/), where the COMET-Test sketch passed on my own laptop.

<details class="toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## Bluetooth test on a classroom PC

<p class="entry-date">3 October 2026</p>

**Objective.** Repeat the Part 0 test on the PC that drives a classroom projector, where I have a student account and no administrator rights. The open questions were whether its Bluetooth adapter supports LE and whether its policy allows a new keyboard and mouse to pair. My earbuds already paired with it, which proved Bluetooth Classic and pairing in general, but neither of those.

**Method.** Same COMET-Test firmware and XIAO ESP32-C3 as Part 0, powered over USB-C, with the laptop's and phone's Bluetooth switched off so neither could claim the bonded device first. Pairing through Settings, Bluetooth & devices, Add device. Serial output read on the laptop at 115200 baud.

**Result.**

| Step | What happened |
|---|---|
| Device Manager check for "Microsoft Bluetooth LE Enumerator" | Device Manager would not open from the student account. Check skipped |
| First pairing attempts | COMET-Test appeared in the list, so the radio could see an LE advertiser, but connecting failed with "Try connecting your device again" |
| TP-Link USB Bluetooth adapter plugged in | Still failed. After unplugging it, the Bluetooth option disappeared from Settings until the PC was restarted |
| After the restart, built-in radio | Connected after a couple of tries; `pairing done: encrypted=1 bonded=1` |
| Square, centre jump, Page Down | All three worked as on the laptop. Pass |
| Connection stability | Dropped once with `reason 520`, readvertised and reconnected by itself from the stored bond. After a further PC restart it stayed connected |

Serial output around the drop:

```text
3/3 Page Down (keyboard)
disconnected, reason 520; advertising again
connected, demo starts in 3 s
pairing done: encrypted=1 bonded=1
1/3 square (relative mouse)
```

NimBLE reports controller errors as 0x200 plus the HCI error code, so 520 (0x208) is HCI error 0x08, Connection Timeout: the link went quiet for longer than the supervision timeout and both sides dropped it. That points to signal quality rather than a refusal by the PC.

> **Lesson:** Windows uses one Bluetooth radio at a time. Plugging in a second adapter on a PC without administrator rights made it try to switch radios and left it with none until a restart. On a PC whose built-in radio works, leave the adapter out.

**What it shows.** The classroom PC's built-in radio handles Bluetooth LE, and its policy allows a Bluetooth keyboard and mouse to pair and bond. That was the main risk to the design; the ESP32-S3 USB dongle fallback from Part 0 is not needed for this PC.

**What it does not show.** Range across the room, which was not walked; the cause of the single timeout; and whether the bond survives the PC's own restarts on later days, since lab PCs often reset their settings.

## Wiring the MPU-6050

<p class="entry-date">3 October 2026</p>

The gyro went on the breadboard on its own, without the four buttons, to get cursor motion working first.

![Adafruit MPU-6050 breakout with all nine header pins soldered, seated in a breadboard](/images/comet_mpu6050_header.jpg)

*The Adafruit 3886 MPU-6050 breakout after soldering all nine header pins, not just the four I²C and power pins, so it sits level in the breadboard and INT and AD0 are available later. The silkscreen marks the sensor's X and Y axes, which matter in the firmware rounds below.*
{: .caption}

| MPU-6050 pin | XIAO pin | GPIO |
|---|---|---|
| VIN | 3V3 | |
| GND | GND | |
| SCL | D2 | 4 |
| SDA | D3 | 5 |

3Vo, XCL, XDA, AD0 and INT are left unconnected; AD0 floating keeps the I²C address at 0x68, and the breakout carries its own I²C pull-ups. These are the same I²C pins as upstream and as the [draft pin map](/comet-air-mouse/#pin-map-draft).

![MPU-6050 near the top of a breadboard wired to a XIAO ESP32-C3 near the bottom, with a USB-C cable plugged into the XIAO](/images/comet_gyro_wiring.jpg)

*The gyro test wiring: the MPU-6050 (top) and the lower XIAO ESP32-C3 (bottom), powered over USB-C while the gyro firmware was flashed. The u.FL antenna socket is still empty here; the antenna was fitted afterwards. The resistors at the top and the rail headers at the bottom are left over from earlier work and are not connected to this circuit.*
{: .caption}

**Firmware setup.** The MPU-6050 is configured directly over I²C at 400 kHz: ±500 °/s full scale (65.5 LSB per °/s), digital low-pass filter at about 44 Hz, 125 Hz sample rate to match the report rate, read every 8 ms by a non-blocking loop. At power-up the board lies still for 2 s while 250 samples are averaged into a per-axis bias.

| Check | Result |
|---|---|
| Build | 592,455 bytes of flash (45%), 23,364 bytes of RAM (7%), ESP32 core 3.3.12 |
| WHO_AM_I | 0x68 |
| Bias measured at power-up | gx −4.28, gy −3.03, gz −2.43 °/s |
| Rest noise after bias removal | Within ±0.1 °/s on all three axes (serial printout every 250 ms) |
| Pairing as COMET-Presenter on the classroom PC | Bonded on the second try |

> **Lesson:** The Arduino IDE compiles every `.ino` file in a sketch folder as one program. The new sketch saved beside `comet_test.ino` failed with duplicate `setup()`, `loop()` and class definitions. Each sketch needs its own folder named like the file.

## From gyro rate to cursor

<p class="entry-date">3 October 2026</p>

**Objective.** Turn the gyro's rotation rate into cursor motion that behaves like a laser pointer: swing right and the cursor goes right, swing down and it goes down, however the hand is rolled, with no motion while the hand is still. No buttons yet, so the cursor follows the gyro whenever a PC is connected.

**Method.** One change at a time on the classroom PC, judged by watching the projector. Every round sends 8-bit relative mouse reports at 125 Hz and carries the fractional pixel left over from each report into the next, so slow motion is not rounded away.

| Round | Mapping and response | Observed |
|---|---|---|
| v0 | Board frame, as upstream: gyro Z to cursor X, gyro Y to cursor Y. Linear, 0.10 px per °/s per report (12.5 px per degree), 3 °/s deadzone | Jitter while still. Slow moves barely moved the cursor; fast ones threw it across the screen. Tilting up and down did nothing; rolling the wrist moved the cursor vertically |
| v0.1 | Cursor Y from gyro X instead. Exponential smoothing (α 0.25), 4 °/s deadzone, power-law speed curve (exponent 0.6) | Correct with the board face up. With the board upside down, swinging down moved the cursor up |
| v0.2 | Gravity compensation (below), same speed curve | Correct directions at any roll, including upside down, but far too slow: close to a full turn of the hand to cross the screen |
| v0.3 | Linear, set in real-world terms: 35° of swing per 1920 px screen width. 1€ filter, 1.5 °/s deadzone | Directions correct and sensitive, but the cursor drifts by itself while the board is held still. See [the open problem](#open-problem-drift-at-rest) |

**Why v0 and v0.1 failed.** A gyro measures rotation about the board's own axes. Mapping fixed board axes to screen axes only works for one way of holding the board. Held as in this test, upstream's gyro Y axis was wrist roll, and turning the board upside down reverses every fixed mapping.

**Gravity compensation (v0.2).** The accelerometer in the same MPU-6050 measures gravity. Low-pass filtered (α 0.05, about 160 ms), it gives the "up" direction <b>û</b> in board coordinates. With **n** the board's pointing axis (its −Y end), the firmware splits the gyro vector **ω** into:

- yaw rate = **ω** · <b>û</b>, rotation about vertical, which drives cursor X
- pitch rate = **ω** · (**n** × <b>û</b>) / \|**n** × <b>û</b>\|, rotation about the horizontal axis across the pointing direction, which drives cursor Y

Rolling the wrist about **n** contributes to neither, so the mapping holds at any roll. Pointing straight up or down leaves **n** × <b>û</b> undefined, so the firmware keeps the last valid axis there.

**Why v0.2 was slow.** The speed curve and deadzone, calculated from the firmware constants:

| Swing speed | Cursor speed | Rotation to cross 1920 px |
|---|---|---|
| 30 °/s | 335 px/s | about 170° |
| 100 °/s | 735 px/s | about 260° |

The curve gave less distance per degree the faster the swing, the opposite of what a pointer needs.

**Before v0.3: upstream and published practice.** Upstream's v1.0.0 sketch maps gyro Z and Y in the board frame, scales linearly by 0.20, truncates each report to whole pixels (dropping slow motion), applies a 1 px deadzone after scaling and boosts diagonals by 1.12. Its click-or-hold-to-scroll button handling (180 ms threshold, a step every 120 ms) is worth reusing once buttons are wired; its motion code had no fix for the problems above. Two published sources set the direction for v0.3:

- Gyro aiming guidance from game development ([Game Developer](https://gamedeveloper.com/design/the-absolute-basics-of-good-gyro-controls)): keep the response linear and state sensitivity in real-world terms, provide a button that turns the gyro off while the hand is repositioned, and calibrate manually while the device is set down. The planned Point button is that clutch.
- The [1€ filter](https://gery.casiez.net/1euro/) (Casiez et al.), a low-pass filter whose cutoff rises with speed: heavy smoothing when the hand is nearly still, to remove tremor, and light smoothing during fast moves, to avoid lag.

v0.3 therefore uses 1920 / 35 = 54.9 px per degree on both axes (a 1080 px screen height is about 20°), with the 1€ filter at a 1 Hz minimum cutoff and a slope of 0.05 Hz per °/s on the yaw and pitch rates.

![The breadboard held like a pointer in front of a classroom projector screen, swept left, right, up and down](/images/comet_v03_projector.gif)

*v0.3 on the classroom projector, the breadboard powered from a USB power bank held behind it. The cursor is a few pixels wide at this size; the clip shows the pointing motion and setup rather than the cursor path.*
{: .caption}

## Open problem: drift at rest

<p class="entry-date">3 October 2026</p>

With v0.3, the cursor moves by itself while the board is held still:

| Board orientation | Drift direction |
|---|---|
| Face up | Up and to the left |
| Upside down | Down and to the right |

Drift speed was not measured. The reversal with orientation is the useful clue. A leftover bias in the board's own axes would be projected onto yaw and pitch through <b>û</b>, which flips sign when the board is turned over, so it would reverse in exactly this way. In v0.1 and v0.2 the 4 °/s deadzone hid any small residual; v0.3 cut the deadzone to 1.5 °/s and raised the gain about four times. This is a hypothesis, not yet tested.

**What this section shows.** Gravity compensation gives correct pointing directions at any roll, and real-world sensitivity makes the response usable.

**What it does not show.** The source of the drift; whether the 2 s boot calibration is long enough or the bias changes as the sensor warms; and how the 1€ filter settings feel once the drift is gone.

**Next:** measure the residual rate at rest for a few minutes after calibration, then update the bias automatically whenever the board is still (low gyro variance and an accelerometer magnitude near 1 g). After that, wire the four buttons on the [draft pin map](/comet-air-mouse/#pin-map-draft) so the cursor moves only while Point is held.
