---
layout: project
title: COMET Air Mouse
permalink: /comet-air-mouse/
series: comet-air-mouse
description: A gyroscope-driven Bluetooth LE mouse on an ESP32-C3 and MPU-6050, reworked into a classroom presenter remote. Project overview and build log.
lede: A mouse that follows hand movement through the air. An ESP32-C3 reads an MPU-6050 gyroscope and reports to the computer as a standard Bluetooth LE HID device, so no receiver or driver is needed. This build adapts CoreComet Industries' open-source design into a presenter remote for classroom talks.
hero: /images/comet_breadboard_test.jpg
hero_alt: Two Seeed XIAO ESP32-C3 boards on a breadboard, the lower one powered over USB-C
hero_position: center 72%
facts:
  - label: Controller
    value: Seeed XIAO ESP32-C3
  - label: Sensor
    value: MPU-6050 gyro
  - label: Link
    value: Bluetooth LE HID
  - label: Inputs
    value: 4 buttons
  - label: Power
    value: 1S LiPo (planned)
  - label: Stage
    value: Gyro cursor tuning
---

## Where it stands

*As of 3 October 2026.* The Bluetooth test passed on a classroom PC as well as my laptop. The MPU-6050 is wired and the cursor follows the hand with correct directions at any wrist roll, but drifts while the board is held still. Fixing the drift is next.

| Work | Status |
|---|---|
| Parts ordered and received (DigiKey.ca) | Done, 26 to 30 September |
| Presenter design: buttons, pin map, HID reports | Drafted, [Part 0](/comet-air-mouse/part-0-presenter-design-and-ble-test/#design-for-classroom-presenting) |
| Bluetooth LE HID test on my laptop | Done, [Part 0](/comet-air-mouse/part-0-presenter-design-and-ble-test/#results) |
| Same test on a classroom PC | Done, [Part 1](/comet-air-mouse/part-1-classroom-test-and-gyro-cursor/#bluetooth-test-on-a-classroom-pc) |
| Breadboard: MPU-6050 | Done, [Part 1](/comet-air-mouse/part-1-classroom-test-and-gyro-cursor/#wiring-the-mpu-6050) |
| Gyro cursor firmware | In progress: gravity-compensated pointing works; [drift at rest](/comet-air-mouse/part-1-classroom-test-and-gyro-cursor/#open-problem-drift-at-rest) is **next** |
| Breadboard: four buttons | Not started |
| Presenter firmware | Not started |
| Battery, power switch and enclosure | Not started; LiPo still to buy |

## Build log

{% include build-log.html series="comet-air-mouse" %}

## Design

The upstream [COMET Air Mouse v1.0.0](https://github.com/CoreCometIndustries/COMET-AIR-MOUSE/releases/tag/v1.0.0) is a general-purpose air mouse. This build changes it for presenting while standing beside a screen:

| | Upstream v1.0.0 | This build |
|---|---|---|
| **Buttons** | 2: left and right click, hold to scroll | 4: Point, Next, Prev, spare |
| **Cursor** | Always follows the gyro | Moves only while Point is held, so drift between gestures does not matter |
| **Slides** | Mouse clicks | Page Down / Page Up from a keyboard report, which works in PowerPoint, Google Slides and PDF viewers |
| **Pointer** | Relative mouse | Relative mouse plus an absolute pointer, so a double tap on Point can recenter the cursor |
| **Laser dot** | None | Ctrl+L on Point press and Ctrl+A on release, using PowerPoint's built-in laser pointer |
| **Button pins** | GPIO 3 and GPIO 2 (a strapping pin) | GPIO 3, 6, 7 and 10; no strapping pins |

![Block diagram: XIAO ESP32-C3 with gyro and buttons, Bluetooth LE HID link, Windows PC with three HID reports](/images/comet_block_diagram.svg)

*System architecture. Solid blocks worked on 2 October 2026; dashed blocks are planned.*
{: .caption}

### Pin map (draft)

| XIAO pin | GPIO | Function | Note |
|---|---|---|---|
| D0 | 2 | unused | Strapping pin |
| D1 | 3 | Button: Point | Only free pin that can wake the chip from deep sleep |
| D2 | 4 | I²C SCL | MPU-6050 |
| D3 | 5 | I²C SDA | MPU-6050 |
| D4 | 6 | Button: Next | |
| D5 | 7 | Button: Prev | |
| D6, D7 | 21, 20 | spare | UART pins; safe as inputs |
| D8, D9 | 8, 9 | unused | Strapping pins; D9 is the BOOT button |
| D10 | 10 | Button: spare | |

Buttons are active low on internal pull-ups, with no external resistors. Upstream's I²C pins are D3/D2, not the SDA/SCL labels printed at D4/D5 on the XIAO.

### Parts

| Part | Specification | Qty | Status |
|---|---|---|---|
| Microcontroller | Seeed XIAO ESP32-C3 (113991054), u.FL antenna | 1 | Borrow or buy a third |
| Gyroscope | Adafruit 3886, MPU-6050 breakout | 1 | Owned |
| Buttons | E-Switch TL1150AF070Q, 6 × 6 mm | 4 | Owned |
| Battery | 1S LiPo, 500 to 1100 mAh, JST-PH 2.0 mm, rated for at least 410 mA charge | 1 | To buy |
| Power switch | MFS slide switch, in the battery positive lead | 1 | Owned |
| Battery cable and header | Adafruit 261 JST-PH cable, JST B2B-PH-K-S header | 1 each | Owned |
| Carrier board | DigiKey SolderFul DKS-SOLDERBREAD-01 | 1 | Owned |
| Enclosure | 3D printed, trigger and thumb layout | 1 | Later |

The battery's charge rating matters because the XIAO's onboard charger is fixed at 380 ± 30 mA.

## Credits

Based on [COMET Air Mouse](https://github.com/CoreCometIndustries/COMET-AIR-MOUSE) by CoreComet Industries, released under the MIT License.
