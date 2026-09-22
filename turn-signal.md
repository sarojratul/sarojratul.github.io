---
layout: page
title: Wireless Bicycle Turn Signal
permalink: /turn-signal/
description: A 32-LED amber arrow for a backpack, driven by an ESP32-C3 and commanded over ESP-NOW from a handlebar remote. Project overview and build log index.
---

A double-headed amber arrow that mounts on a backpack, so drivers behind me can see which way I am turning without me taking a hand off the bars. Two buttons on the handlebar send the command over ESP-NOW to an ESP32-C3 on the backpack, which sweeps the arrow outward in the direction of the turn.

![All four segments of the arrow lit on the breadboard](/images/all_four_segments_lit.jpg)

The build log is written as I go rather than afterwards, so the failures are still in it. If you are new here, start at [Part 0](/turn-signal/part-0-why/) and use the Previous / Next links at the top and bottom of each entry.

<details class="toc" markdown="1">
<summary>On this page</summary>

* TOC
{:toc}

</details>

## Where it stands

*As of 21 September 2026.*

**Paused for the winter.** I am not riding right now and will not be until winter is over, so the project is on hold. That leaves the whole winter to finish it before spring. Meanwhile I am building a [COMET Air Mouse](/comet-air-mouse/). [More in Part 7](/turn-signal/part-7-whats-next/).

| Work | Status |
|---|---|
| Design, parts and documents | Done ([Part 1](/turn-signal/part-1-design/)) |
| Simulation: LTspice, Wokwi, Tinkercad (Stage A) | Done ([Part 2](/turn-signal/part-2-simulation/)) |
| Panel on the breadboard, all four segments verified (B5) | Done ([Parts 3 to 5](/turn-signal/part-3-soldering-and-panel-build/)) |
| Safe bench power source (B6) | Done ([Part 5](/turn-signal/part-5-full-panel/#b6-closed-by-the-equipment-already-on-the-bench)) |
| ESP32-C3 driving the panel, sweep timing settled (B7) | Done ([Part 6](/turn-signal/part-6-esp32-drives-the-panel/)) |
| Handlebar transmitter and ESP-NOW link (B8) | **Next**, on hold until I pick the project back up this winter |
| Power stage, TPS63070 buck-boost (B10) | Not started |
| Enclosure and weatherproofing (Stage C) | Planned ([Part 7](/turn-signal/part-7-whats-next/)) |
| Bare-metal TM4C123 firmware and custom PCB (Phase 2) | Planned ([Part 7](/turn-signal/part-7-whats-next/#phase-2-bare-metal-migration-and-a-custom-pcb)) |

## The build log

<ol class="parts-list">
{%- assign series_posts = site.posts | where: "series", "turn-signal" | sort: "part" -%}
{%- for p in series_posts %}
  <li>
    <a class="parts-list__title" href="{{ p.url | relative_url }}">{{ p.title | escape }}</a>
    <div class="parts-list__meta">{{ p.covers }}</div>
    <p>{{ p.excerpt | strip_html }}</p>
  </li>
{%- endfor %}
</ol>

## At a glance

These are the current values. Where a number changed during the build, the entry where it changed is linked.

| | Current value |
|---|---|
| **Display** | One double-headed arrow, 32 amber LEDs (Lumex SSL-LX5093AD, 605 nm) |
| **Segments** | A / B / C / D, left to right: 10 / 6 / 6 / 10 LEDs. B and C are shared by both directions ([why](/turn-signal/part-1-design/#two-arrows-became-one)) |
| **LED strings** | 16 strings of 2 LEDs in series, each with one 33 Ω resistor |
| **Switching** | 4 × AO3400A N-MOSFET, low side; 220 Ω series gate resistor and 10 kΩ pulldown per segment |
| **Controllers** | 2 × Seeed XIAO ESP32-C3: a stateless handlebar transmitter and a backpack receiver that holds all behaviour |
| **Link** | ESP-NOW, 2.4 GHz, peer to peer, no router. Panel forces itself dark if the link is quiet for more than 1000 ms |
| **Receiver pins** | GPIO 3 / 4 / 5 / 6 for segments A / B / C / D ([changed from 2 / 4 / 5 / 6](/turn-signal/part-6-esp32-drives-the-panel/#a-gpio-pin-the-datasheet-caught-before-i-did)) |
| **Panel power** | 4 × AA NiMH, 2800 mAh, 4.8 V nominal, straight to the LEDs |
| **Logic power** | TPS63070 buck-boost to 3.3 V (backpack); 3 × AAA NiMH 1100 mAh with its own TPS63070 (handlebar) |
| **Sweep** | Centre outward: LEFT is C, C+B, C+B+A; RIGHT is B, B+C, B+C+D |
| **Timing** | STEP_MS 450, HOLD_MS 788, CYCLE_MS 1688, 4 trailing cycles after release ([tuned on the panel](/turn-signal/part-6-esp32-drives-the-panel/#tuning-the-sweep-by-eye-then-checking-it-with-a-scope)) |
| **Measured current** | 265 mA with all four segments lit at 4.81 V. The design model's ceiling was 396.5 mA ([why they differ](/turn-signal/part-5-full-panel/#segment-ds-current-came-up-short-and-why-i-stopped-chasing-it)) |

![Block diagram: handlebar transmitter, ESP-NOW link, backpack receiver](/images/block_diagram.svg)

## How the work is organised

Two documents existed before any hardware did, and the log refers to them constantly ([more on both](/turn-signal/part-1-design/#the-build-guide-and-the-work-plan)):

- **The build guide:** spec, architecture, bill of materials, calculations. The source of truth for *what and why*.
- **The work plan:** the order of work, in three stages. Every step has written pass criteria, set before the test is run.

| Stage | What it covers | Steps mentioned in the log |
|---|---|---|
| **A: Simulation** | Prove it before buying or building | A2 LTspice, A3 Wokwi |
| **B: Breadboard** | Real parts, one stage at a time | B5 LED panel, B6 bench power source, B7 microcontroller, B8 radio link, B10 power stage |
| **C: Enclosure and integration** | Housing, weatherproofing, final assembly | C13b enclosure test coupon, T12 rain test |

The project files still say "jacket" because that is what it started as. [Part 0](/turn-signal/part-0-why/#it-was-a-jacket-first) explains why it became a backpack panel instead.

## What this project has taught me so far

**Analog:** LED forward-voltage behaviour and why current-setting resistors are not optional. MOSFET threshold and R<sub>DS(on)</sub> behaviour at low gate drive, and why "logic-level" is a specification you check rather than a word you trust. Why a buck-boost and not an LDO on a battery that sags.

**Simulation:** LTspice DC operating point, DC sweep and transient analysis. Building a device model from a datasheet curve when the manufacturer does not provide one. Knowing what a simulation *cannot* tell you, which is most of Parts 3 and 4.

**Firmware:** a state machine with a deliberate seam between behaviour and transport, so the same logic runs against buttons or a radio. Timing that does not free-run. Tuning a constant against observed behaviour rather than guessing it.

**Instrumentation:** the multimeter in series, in continuity mode, in diode mode. The AD2 for what it is genuinely good at. Identifying a part's pinout empirically instead of trusting a silkscreen. Tying every instrument's ground together before measuring anything.

**Process, and this is the one I would defend hardest:** writing pass criteria before running the test. Working outward from the source with an ordered ladder instead of guessing at causes. Marking superseded designs instead of deleting them. Recording what a passing test does *not* prove. Checking every proposed part against its own datasheet before spending a dollar.

**And a soldering iron:** which I now know how to look after, at a cost of nine MOSFETs, three burns and a mark on the table.
