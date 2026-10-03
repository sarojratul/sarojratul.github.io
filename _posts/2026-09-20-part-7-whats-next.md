---
layout: series-post
title: "Part 7: Next Steps, Enclosure and Phase 2"
date: 2026-09-20 20:00:00 -0600
series: turn-signal
part: 7
covers: "Planned, not built"
excerpt: "Decisions already made for the 3D-printed enclosure, and the Phase 2 plan: bare-metal firmware on a TM4C123 and a two-layer KiCad carrier board."
permalink: /turn-signal/part-7-whats-next/
---

> **Update, 21 September 2026:** the project is paused for the winter, since I am off the bike until spring. I plan to finish it over the winter so it is ready for riding season. Meanwhile I am building a [COMET Air Mouse](/comet-air-mouse/). The plan below is unchanged.

None of this is built. It records decisions already made, with their reasons. The immediate next bench step remains Stage B: **B8** (second ESP32-C3 and the ESP-NOW link), then the power stage.

## Enclosure and mechanical integration (Stage C)

**3D-printed in-house, replacing the Hammond 1553WBBK,** for both the handlebar controller and the backpack panel:

- **Gasket groove in the lid** for silicone cord or a printed gasket.
- **Heat-set brass inserts** for the lid screws, since printed threads strip with repeated opening.
- **Open:** material and process. PETG, ASA, PLA and resin are all candidates, and the inserts are not sourced until that is chosen. PLA is ruled out for anything left on a bike in a Saskatchewan summer.

**Step C13b, before assembly:** model the enclosure around the bulkiest component, design the gasket groove and insert bosses, and **print a test coupon first**, so an interference fit is checked on a fifteen-minute print rather than a six-hour one.

**Weatherproofing is gated by test.** A gasketed lid does not earn an "IP54-equivalent" claim until test T12, the rain test, confirms the seal. Until then the documents say "designed for", not "rated".

<!-- Screenshot wanted: the Fusion 360 enclosure model, and the printed gasket-groove test coupon. -->

## Phase 2: bare-metal firmware and a custom PCB

**Plan:** reduce the ESP32-C3 to a radio bridge and move all control logic to a **TM4C123 (ARM Cortex-M4)** over UART, written at register level with no Arduino layer:

- GPIO and SysTick configured by direct register writes
- Six-channel PWM for LED brightness
- ADC for battery monitoring
- **ARM assembly `PRIMASK` critical sections** guarding state shared between the UART receive path and the animation timer
- A **watchdog that fails safe** through hardware already present: the 10 kΩ gate pulldowns turn every segment off if the microcontroller stops driving them, a decision from [Part 1](/turn-signal/part-1-design/#architecture)

**Custom board:** a two-layer **KiCad** BoosterPack carrier PCB with an **antenna keepout** under the ESP32-C3 module (no copper pour or traces beneath it). Trace widths split by function: wide pours for LED return paths carrying up to about 400 mA, minimum-width traces for the four gate lines at about 320 µA. The two currents differ by three orders of magnitude, and both come from the measurements in Parts 2 to 5.

<!-- Screenshot wanted: the KiCad schematic and the two-layer board layout. -->

For what the project has taught so far, see the [project overview](/turn-signal/#what-this-project-has-taught-me).
