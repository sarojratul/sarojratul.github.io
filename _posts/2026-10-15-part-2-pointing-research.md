---
layout: series-post
title: "Part 2: Absolute Pointing Research and the Plan for the Drift Fix"
date: 2026-10-15 20:00:00 -0600
series: comet-air-mouse
part: 2
covers: "15 October 2026"
excerpt: "A research session with no bench work: whether the presenter needs true laser-pointer aiming, how commercial presenter remotes point without it, and which open-source code to combine with the v0.3 firmware to fix the drift at rest."
permalink: /comet-air-mouse/part-2-pointing-research/
---

This entry continues from [Part 1](/comet-air-mouse/part-1-classroom-test-and-gyro-cursor/), which ended with gravity-compensated pointing working and the cursor [drifting while the board was held still](/comet-air-mouse/part-1-classroom-test-and-gyro-cursor/#open-problem-drift-at-rest). No hardware was touched on 15 October. The session settled the pointing approach before the next firmware round.

<details class="toc" markdown="1">
<summary>Contents</summary>

* TOC
{:toc}

</details>

## Absolute or relative pointing

<p class="entry-date">15 October 2026</p>

**Objective.** Decide whether the presenter needs absolute pointing, where the cursor lands exactly where the device points from any position in the room, as a physical laser does, or whether relative gyro pointing is enough.

**Method.** A review of optical tracking architectures, open-source lightgun firmware, sensor datasheets and DigiKey.ca availability.

**Result.** A gyro only measures rotation, so it can never know where the screen is. Absolute pointing needs an optical reference at the screen, and there are two ways to arrange it:

| | Outside-in: camera at the screen, IR LEDs in the remote | Inside-out: camera in the remote, IR markers at the screen |
|---|---|---|
| What it measures | Where the remote is in the camera's view. One LED gives only the direction from camera to remote; the pointing direction needs four or more LEDs on the remote and a pose solve | A 4-point homography from image to screen; the camera's centre maps straight to a screen pixel, with perspective and roll handled |
| Pointing error at 2.5 m (calculated) | About 135 mm | About 2.3 mm |
| Remote | Very simple: LEDs, MCU, battery | Camera, MCU, battery |
| Prior art | Research prototypes | Wii Remote; open-source lightguns such as OpenFIRE and Samco |

Both error figures assume 0.1 px blob-centroid noise on a PixArt tracking camera (98 px across 52.5°, so 0.94 mrad per 0.1 px). Inside-out: 0.94 mrad × 2.5 m ≈ 2.3 mm. Outside-in with a 60 mm LED baseline on a pen-sized remote: each LED is located to about 2.3 mm, so the axis is known to about (2.3 × √2) / 60 ≈ 0.054 rad ≈ 3.1°, and 2.5 m × tan 3.1° ≈ 135 mm. These are estimates, not measurements, but the ratio of roughly 60 is why every lightgun puts the camera in the hand.

> **Lesson:** One camera at the screen tracking one IR LED in the remote does not act like a laser pointer. It finds where the remote is, not where it points. I had considered that arrangement as the simple option before this review.

The shortest absolute path found was the inside-out lightgun design: OpenFIRE's ESP32 firmware, a DFRobot SEN0158 IR camera (DigiKey.ca 1738-1250-ND), two XIAO ESP32-S3 boards (the remote and a USB dongle; the ESP32-C3's USB is serial and JTAG only, so it cannot be a USB HID device) and twelve ams OSRAM SFH 4547 940 nm LEDs as four markers. From DigiKey.ca price snapshots that is about CAD $100–115. Two catches: the SEN0158's 33° × 23° field of view keeps all four markers in sight only from about 2 m away, and the 141° PixArt PAJ7025R3 needed for closer use is not stocked by DigiKey.ca.

**Conclusion.** The presenter does not need absolute pointing. The requirement is now that it feels natural to use and works when standing left or right of the screen at an angle. Relative gyro pointing meets that on the hardware already wired, so the IR design is parked.

## How commercial presenter remotes point

<p class="entry-date">15 October 2026</p>

The Logitech Spotlight has no camera or IR parts. iFixit's teardown shows a Nordic nRF51802 Bluetooth chip and an InvenSense 6-axis IMU. It points the same way as this build:

- The gyro measures how fast the wrist turns; the accelerometer gives gravity, so the axes stay level however the remote is rolled.
- The pointer moves only while the button is held, and each press starts from where the pointer already is. A press lasts a few seconds, too short for gyro drift to show.
- The highlight is drawn by Logitech's PC software; the remote only sends motion.

The user's eyes close the loop: they steer the dot onto the target, so the remote never needs to know where the screen is. The Point button in the [presenter design](/comet-air-mouse/part-0-presenter-design-and-ble-test/#design-for-classroom-presenting) is the same clutch.

**Standing to the side (calculated, not measured).** At 45° to one side, the screen spans about cos 45° ≈ 0.71 of the horizontal angle it does head-on. With a fixed gain of 35° per screen width, a swing that looks as though it should cross the screen moves the cursor only about 70% of the way. The eye corrects for this during use; a sensitivity setting would remove it if it proves annoying.

> **Note:** One source in the review described Bluetooth absolute pointers as unreliable on Windows. The absolute recenter report already [passed on my laptop](/comet-air-mouse/part-0-presenter-design-and-ble-test/#results) and on the [classroom PC](/comet-air-mouse/part-1-classroom-test-and-gyro-cursor/#bluetooth-test-on-a-classroom-pc), so the measured result stands for this design.

## Code for the next firmware round

<p class="entry-date">15 October 2026</p>

**Objective.** Find open-source code that fixes the drift at rest and can be merged into the v0.3 firmware instead of being written from scratch.

**Result.**

| Code | License | What it would do here | Verdict |
|---|---|---|---|
| [COMET-AIR-MOUSE](https://github.com/CoreCometIndustries/COMET-AIR-MOUSE) v1.0.0, the base of v0.3 | MIT | Bluetooth LE HID; click-or-hold button handling for later | Keep |
| [GamepadMotionHelpers](https://github.com/JibbSmart/GamepadMotionHelpers) (Jibb Smart) | MIT | Single C++ header, needs only `<math.h>`. <code>SetCalibrationMode(Stillness &#124; SensorFusion)</code> updates the gyro bias automatically while the device is still, which is the fix [Part 1](/comet-air-mouse/part-1-classroom-test-and-gyro-cursor/#open-problem-drift-at-rest) planned. `GetPlayerSpaceGyro()` takes yaw about gravity, the same idea as v0.3's gravity compensation, so the two can be compared | Use |
| [OneEuroFilterArduino](https://github.com/casiez/OneEuroFilterArduino) (Casiez) | To check | Reference 1€ filter, to replace or cross-check the in-house one in v0.3 | Use |
| [gavin9399/Air_Mouse](https://github.com/gavin9399/Air_Mouse) | None shown | Raw board axes (`gz/256`, `gy/256`), the v0 problem from Part 1 | Not used |
| [gautam-dev-maker/Air-Mouse](https://github.com/gautam-dev-maker/Air-Mouse) | GPL-3.0 | ESP-IDF rather than Arduino; uses the MPU-6050's onboard DMP | Not used; the DMP is a fallback |

Integration notes:

- GamepadMotionHelpers takes gyro in °/s and acceleration in g on a Y-up frame, so the MPU-6050 axes need remapping to its convention. v0.3 already reads both in those units.
- The ESP32-C3 has no hardware floating-point unit, so the added maths runs in software. At 125 Hz this is expected to fit in the 8 ms loop, but it has not been timed.

**What it shows.** A merge path for the drift fix that keeps the firmware already tested on the classroom PC.

**What it does not show.** That it works: nothing was compiled or run on 15 October.

## Decision and plan

<p class="entry-date">15 October 2026</p>

Next session combines the COMET-based v0.3 firmware with GamepadMotionHelpers and OneEuroFilterArduino, then tests it on the breadboard:

1. With v0.3 unchanged, measure the residual rate at rest for a few minutes after the boot calibration, as Part 1 called for. This is the baseline.
2. Add GamepadMotionHelpers' automatic calibration and repeat the measurement.
3. Compare its player-space yaw against v0.3's own gravity compensation.
4. Swap in OneEuroFilterArduino and compare against the in-house filter.
5. Test standing left and right of the screen at an angle.

**Next:** the combined firmware on the breadboard, then the four buttons on the [draft pin map](/comet-air-mouse/#pin-map-draft).
