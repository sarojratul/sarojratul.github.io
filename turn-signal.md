---
layout: project
title: Wireless Bicycle Turn Signal
permalink: /turn-signal/
series: turn-signal
description: A 32-LED amber arrow for a backpack, driven by an ESP32-C3 and commanded over ESP-NOW from a handlebar remote. Project overview and build log.
lede: A double-headed amber arrow worn on a backpack, so drivers behind can see which way I am turning without me taking a hand off the bars. Two handlebar buttons send the command over ESP-NOW to an ESP32-C3 on the backpack, which sweeps the arrow outward in the direction of the turn.
hero: /images/all_four_segments_lit.jpg
hero_alt: All four segments of the arrow lit on the breadboard
hero_caption: All four segments lit on the breadboard, 20 September 2026.
facts:
  - label: Display
    value: 32 amber LEDs, 4 segments
  - label: Drive
    value: 4 × AO3400A, low side
  - label: Controllers
    value: 2 × XIAO ESP32-C3
  - label: Link
    value: ESP-NOW, 2.4 GHz
  - label: Measured draw
    value: 265 mA at 4.81 V, all lit
  - label: Stage
    value: B7 done, B8 next
---

## Where it stands

*As of 21 September 2026.* **Paused for the winter**, since I am off the bike until spring; the plan is to finish it over the winter. [Details in Part 7](/turn-signal/part-7-whats-next/).

| Work | Status |
|---|---|
| Design, parts and documents | Done, [Part 1](/turn-signal/part-1-design/) |
| Simulation in LTspice, Wokwi and Tinkercad (Stage A) | Done, [Part 2](/turn-signal/part-2-simulation/) |
| Panel on the breadboard, all four segments verified (B5) | Done, [Parts 3 to 5](/turn-signal/part-3-soldering-and-panel-build/) |
| Safe bench power source (B6) | Done, [Part 5](/turn-signal/part-5-full-panel/#step-b6-closed) |
| ESP32-C3 driving the panel, sweep timing settled (B7) | Done, [Part 6](/turn-signal/part-6-esp32-drives-the-panel/) |
| Handlebar transmitter and ESP-NOW link (B8) | **Next**, on hold for winter |
| Power stage, TPS63070 buck-boost (B10) | Not started |
| Enclosure and weatherproofing (Stage C) | Planned, [Part 7](/turn-signal/part-7-whats-next/) |
| Bare-metal TM4C123 firmware and custom PCB (Phase 2) | Planned, [Part 7](/turn-signal/part-7-whats-next/#phase-2-bare-metal-firmware-and-a-custom-pcb) |

## Build log

{% include build-log.html series="turn-signal" %}

## Specifications

Current values. Where a number changed during the build, the entry where it changed is linked.

| | Current value |
|---|---|
| **Display** | One double-headed arrow, 32 amber LEDs (Lumex SSL-LX5093AD, 605 nm) |
| **Segments** | A / B / C / D, left to right: 10 / 6 / 6 / 10 LEDs; B and C serve both directions ([why](/turn-signal/part-1-design/#display-layout-one-double-headed-arrow)) |
| **LED strings** | 16 strings of 2 LEDs in series, each with one 33 Ω resistor |
| **Switching** | 4 × AO3400A N-MOSFET, low side; 220 Ω series gate resistor and 10 kΩ pulldown per segment |
| **Controllers** | 2 × Seeed XIAO ESP32-C3: a stateless handlebar transmitter and a backpack receiver holding all behaviour |
| **Link** | ESP-NOW, 2.4 GHz, peer to peer, no router. The panel goes dark if the link is quiet for more than 1000 ms |
| **Receiver pins** | GPIO 3 / 4 / 5 / 6 for segments A / B / C / D ([changed from 2 / 4 / 5 / 6](/turn-signal/part-6-esp32-drives-the-panel/#gpio-2-is-a-strapping-pin)) |
| **Panel power** | 4 × AA NiMH, 2800 mAh, 4.8 V nominal, direct to the LEDs |
| **Logic power** | TPS63070 buck-boost to 3.3 V (backpack); 3 × AAA NiMH 1100 mAh with its own TPS63070 (handlebar) |
| **Sweep** | Centre outward. LEFT: C, C+B, C+B+A. RIGHT: B, B+C, B+C+D |
| **Timing** | STEP_MS 450, HOLD_MS 788, CYCLE_MS 1688, 4 trailing cycles after release ([tuned on the panel](/turn-signal/part-6-esp32-drives-the-panel/#sweep-timing-tuned-by-eye-verified-on-a-scope)) |
| **Measured current** | 265 mA with all four segments lit at 4.81 V; the design model's ceiling was 396.5 mA ([why they differ](/turn-signal/part-5-full-panel/#segment-d-current-below-design)) |

![Block diagram: handlebar transmitter, ESP-NOW link, backpack receiver](/images/block_diagram.svg)

*System architecture: the transmitter only reports button state; every decision is made on the receiver.*
{: .caption}

## How the work is organised

Two documents existed before any hardware ([more on both](/turn-signal/part-1-design/#build-guide-and-work-plan)): a **build guide** (specification, architecture, BOM, calculations) and a **work plan** (the order of work, with written pass criteria for every step). The project files still say "jacket" because that is where it started ([why it changed](/turn-signal/part-0-why/#from-a-jacket-to-a-backpack-panel)).

| Stage | Scope | Steps referenced in the log |
|---|---|---|
| **A: Simulation** | Prove it before building | A2 LTspice, A3 Wokwi |
| **B: Breadboard** | Real parts, one stage at a time | B5 LED panel, B6 bench power, B7 microcontroller, B8 radio link, B10 power stage |
| **C: Enclosure and integration** | Housing, weatherproofing, final assembly | C13b enclosure test coupon, T12 rain test |

## What this project has taught me

- **Analog:** LED forward-voltage behaviour and why series resistors are required; MOSFET threshold and R<sub>DS(on)</sub> at low gate drive, and checking "logic-level" against the datasheet; why a battery that sags needs a buck-boost rather than an LDO.
- **Simulation:** LTspice operating point, DC sweep and transient analysis; building a device model from a datasheet curve; knowing what a simulation cannot show, which is most of Parts 3 and 4.
- **Firmware:** a state machine with a seam between behaviour and transport, so the same logic runs on buttons or a radio; timing that does not free-run; tuning a constant against observed behaviour.
- **Instrumentation:** the multimeter in series, in continuity mode and in diode mode; the AD2 where it is strongest; identifying a pinout empirically; tying every instrument's ground together before measuring.
- **Process:** writing pass criteria before running a test; troubleshooting outward from the source with an ordered ladder; keeping superseded designs marked rather than deleted; recording what a passing test does not prove; checking every proposed part against its datasheet before buying.
- **Soldering:** tip care, flux and tinning, learned at the cost of nine MOSFETs.
