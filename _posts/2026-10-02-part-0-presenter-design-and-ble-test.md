---
layout: series-post
title: "Part 0: Presenter Design and a First Bluetooth HID Test"
date: 2026-10-02 21:00:00 -0600
series: comet-air-mouse
part: 0
covers: "21 September to 2 October 2026"
excerpt: "The project's starting point, the changes needed to turn an air mouse into a presenter remote, and a test sketch proving that Windows accepts one Bluetooth LE device acting as keyboard, mouse and absolute pointer."
permalink: /comet-air-mouse/part-0-presenter-design-and-ble-test/
---

<details class="toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## Starting point

<p class="entry-date">21 to 30 September 2026</p>

The project started on 21 September 2026, when the [bicycle turn signal](/turn-signal/) was paused for the winter. It is based on CoreComet Industries' open-source [COMET Air Mouse v1.0.0](https://github.com/CoreCometIndustries/COMET-AIR-MOUSE/releases/tag/v1.0.0): an ESP32-C3 reads an MPU-6050 gyroscope and reports to the computer as a Bluetooth LE HID mouse. Upstream features:

- Native Bluetooth LE HID with "Just Works" pairing (no PIN) and a 125 Hz report rate
- Gyro Z mapped to cursor X, gyro Y to cursor Y, with smoothing, sensitivity and a motion deadzone
- Automatic gyro calibration
- Left and right click, with hold to scroll
- Works with Windows and Android, per the release notes

Upstream lists only the ESP32-C3, the MPU-6050 and two buttons, so the rest of the bill of materials was worked out separately. The parts were ordered from DigiKey.ca on 26 September and arrived by 30 September. A 1S LiPo is still to be bought.

## Design for classroom presenting

<p class="entry-date">2 October 2026</p>

The intended use is giving presentations: standing beside the screen, walking from side to side, pointing the device like a laser pointer and pressing buttons to change slides. Latency is not the concern; a BLE HID connection interval of 7.5 to 15 ms with 125 Hz reports is comparable to a wireless mouse. Three properties of a gyro mouse matter more:

- **It is relative, not absolute.** The cursor moves by the amount the hand rotates, from wherever it already is. It does not go where the device points, and it can end up pinned against a screen edge.
- **The gyro drifts.** Position is integrated from rotation rate, so small errors accumulate and the cursor creeps while the hand is still.
- **Walking adds motion.** Steps and arm swing feed into the signal.

| Problem | Design decision |
|---|---|
| Drift, and the cursor wandering while talking with my hands | **Hold to move:** the cursor follows the gyro only while Point is held |
| Slide changes by mouse click depend on application settings | **Keyboard report:** Next and Prev send Page Down and Page Up |
| No visible laser dot | Point sends **Ctrl+L** on press and **Ctrl+A** on release, toggling PowerPoint's laser pointer |
| Cursor lost or pinned at an edge | **Double tap Point to recenter,** using an absolute pointer report (0 to 32767 on each axis), which is independent of screen resolution and Windows pointer acceleration |
| Upstream's right button on GPIO 2, a strapping pin | Buttons on **D1, D4, D5 and D10**; nothing on GPIO 2, 8 or 9 |
| Deep sleep can only wake on GPIO 0 to 5 | **Point on D1 (GPIO 3)**, the only free wake-capable pad |
| Pairing a new PC, and others pairing to it | Planned: hold **Prev + Next for 5 s** to accept a new host for 60 s; otherwise accept bonded hosts only. This needs a status LED, since the XIAO ESP32-C3 has no user LED |

Two limits of the absolute recenter: if someone moves the PC's own mouse, the firmware's idea of the cursor position is stale until the next recenter, and with an extended desktop "centre" is the centre of the main display.

The absolute pointer means a custom HID report map, which the common Arduino BLE mouse libraries do not support. Before writing the presenter firmware or wiring any hardware, the first question was whether Windows would accept one device that is a keyboard, a relative mouse and an absolute pointer at once.

## Test setup

<p class="entry-date">2 October 2026</p>

**Objective.** Confirm that Windows pairs with a single Bluetooth LE HID device carrying three report types, and that each report works, on my own laptop before the classroom PC.

**Hardware.** A bare Seeed XIAO ESP32-C3 on USB-C power. No gyro, no buttons, and the u.FL antenna not fitted, so the laptop was kept close.

**Software.** Arduino IDE with the ESP32 Arduino core 3.3.11 and **NimBLE-Arduino 2.x** (h2zero), which accepts a custom HID report map and builds on core 3.x. Board setting XIAO_ESP32C3 with USB CDC On Boot enabled.

**Firmware.** A test sketch, COMET-Test, advertises one device with three input reports: keyboard (ID 1), relative mouse (ID 2) and absolute pointer (ID 3). It pairs with Just Works security and bonds, so a known PC reconnects without re-pairing. Once connected it waits 3 s for Windows to finish setting up the HID service, then repeats a demo cycle: a 100 px square, a jump to screen centre, then a Page Down. The loop is non-blocking, and the Bluetooth callbacks only set flags for the main loop, following the interrupt rules from CME 331.

![Flowchart of the COMET-Test firmware: setup, a non-blocking main loop with three tasks, and BLE callbacks that only set flags](/images/comet_test_flowchart.png)

*Firmware structure. The main loop reads the time once per pass and runs three short tasks; the Bluetooth stack's callbacks only set volatile flags.*
{: .caption}

![Timing diagram of one COMET-Test demo cycle: 80 relative reports, one absolute report, a 30 ms Page Down](/images/comet_test_timing.png)

*One demo cycle, about 10.7 s: 80 relative reports at 8 ms intervals, one absolute report two seconds later, then Page Down held for 30 ms.*
{: .caption}

![Byte layout of the keyboard, relative mouse and absolute pointer reports](/images/hid_report_layout.png)

*The three reports as declared in the report map, with the bytes the sketch actually sends.*
{: .caption}

**Host check before flashing.** The sketch was compiled against stubbed Arduino and NimBLE headers with `-Wall -Wextra` (zero warnings) and run in a 40 s simulation. It produced 80 square steps per cycle at 8 ms spacing, the centre report `00 00 40 00 40`, Page Down released 30 ms after press, and a clean restart of the demo after a simulated disconnect and reconnect.

## Results

<p class="entry-date">2 October 2026</p>

| Check | Expected | Result |
|---|---|---|
| Compile and upload | Builds for XIAO_ESP32C3 | 565,675 bytes of flash (43%), 22,652 bytes of RAM (6%); written and hash-verified |
| Advertising | COMET-Test listed in Windows "Add a device" | Listed with a mouse icon. Pass |
| Pairing | Just Works, bonded | `pairing done: encrypted=1 bonded=1`. Pass |
| Relative mouse (ID 2) | Cursor traces a 100 px square | Traced. Pass |
| Absolute pointer (ID 3) | Cursor jumps to screen centre | Jumped to centre. Pass |
| Keyboard (ID 1) | Page Down advances an open slide deck or PDF | Advanced. Pass |
| Reconnect after power cycle | Reconnects without re-pairing | Reconnected and the cycle resumed. Pass |
| PowerPoint laser pointer, by hand | Ctrl+L shows the red dot, Ctrl+A restores the arrow | Both worked in PowerPoint version 2609 (build 20430.20118). Pass |

![PowerPoint slideshow on the laptop advancing from slide 2 to slide 5 while COMET-Test runs](/images/comet_test_cycle.gif)

*The test on my laptop at 2× speed. The slideshow advances from slide 2 to slide 5, one Page Down per demo cycle from the keyboard report.*
{: .caption}

![Windows Add a device dialog listing COMET-Test with a mouse icon](/images/comet_test_add_device.png)

*Windows lists COMET-Test with a mouse icon, taken from the HID mouse appearance value (0x03C2) in the advertising data.*
{: .caption}

Serial output after the power cycle:

```text
connected, demo starts in 3 s
pairing done: encrypted=1 bonded=1
1/3 square (relative mouse)
2/3 jump to centre (absolute pointer)
3/3 Page Down (keyboard)
```

The second `pairing done` line is not a new pairing. On reconnect, Windows and the XIAO restore encryption from the stored bond, and NimBLE reports it through the same callback; `bonded=1` confirms the stored keys were used.

![COMET-Test running on a XIAO ESP32-C3 on the breadboard](/images/comet_breadboard_test.jpg)

*The test board: the lower XIAO ESP32-C3, on USB-C power, runs COMET-Test with nothing else wired to it. Its u.FL antenna socket, between the B and R buttons, is empty for this test.*
{: .caption}


**What it shows.** Windows accepts one Bluetooth LE device acting as keyboard, relative mouse and absolute pointer, so the presenter design needs no second device. The absolute report working means the double-tap recenter is feasible. Bonding works, so a known PC reconnects on power-up.

**What it does not show.** Anything about the classroom PC, whose Bluetooth adapter must support LE and whose IT policy may block new input devices; range, since no antenna was fitted; and anything about the gyro, buttons or battery.

## Next steps

1. **Repeat the test on a classroom PC.** Check Device Manager for "Microsoft Bluetooth LE Enumerator", pair, confirm all three reports and Ctrl+L in its PowerPoint, then walk the room with the antenna fitted.
2. **If the classroom PC fails,** the fallback is a USB dongle: an ESP32-S3, whose native USB can present as a wired keyboard and mouse, receiving from the air mouse over ESP-NOW. That needs no Bluetooth on the PC and no drivers.
3. **Breadboard the MPU-6050 and four buttons** on the draft [pin map](/comet-air-mouse/#pin-map-draft), then write the presenter firmware.

> **Update, 3 October 2026:** The classroom PC passed after a restart, and the MPU-6050 was wired without buttons first. See [Part 1](/comet-air-mouse/part-1-classroom-test-and-gyro-cursor/).

**Next:** [Part 1: Classroom PC Test and a First Gyro Cursor](/comet-air-mouse/part-1-classroom-test-and-gyro-cursor/)
