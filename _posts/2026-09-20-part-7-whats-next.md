---
layout: series-post
title: "Part 7: Next Up, the Enclosure and Phase 2"
date: 2026-09-20 20:00:00 -0600
series: turn-signal
part: 7
covers: "Planned, not built"
excerpt: "The decisions already made for the 3D-printed enclosure, and the Phase 2 plan: bare-metal firmware on a TM4C123 and a custom two-layer KiCad carrier board."
permalink: /turn-signal/part-7-whats-next/
---

> **Update, 21 September 2026:** I have paused this project. I am not riding right now and will not be until winter is over, so there is no reason to rush B8 before the snow. I will have the whole winter to finish it, and I plan to pick it back up then so it is ready for spring. In the meantime I am building a [COMET Air Mouse](/comet-air-mouse/). Nothing below has changed; it is still the plan.

Nothing in this entry is built yet. It records the decisions that are already made, because they were made for reasons. The immediate next step on the bench is still Stage B: **B8**, the second ESP32-C3 and the ESP-NOW link, then the power stage.

## Enclosure and mechanical integration (Stage C)

**3D-printed, in-house, replacing the Hammond 1553WBBK.** Both units: handlebar controller and backpack panel.

- **Gasket groove in the lid**, taking either silicone cord or a printed gasket.
- **Heat-set brass inserts** for the lid screws, rather than threading into plastic, because a lid that gets opened repeatedly will strip printed threads.
- **Undecided:** print material and process. PETG, ASA, PLA and resin are all still on the table, and the brass inserts are not sourced yet because the choice depends on it. PLA is out for anything left on a bike in a Saskatchewan summer.

**Work plan step C13b, which comes before assembly:** model the enclosure around the bulkiest component first, design the gasket groove and insert bosses, and **print a test coupon before printing the real thing.** Same principle as simulating before breadboarding: find out whether the interference fit works on a part that takes fifteen minutes, not one that takes six hours.

**The weatherproofing claim has a gate on it.** A printed lid with a gasket does not earn an "IP54-equivalent" claim from me until test T12, the rain test, actually confirms the seal. Until then the documents say "designed for," not "rated."

<!-- Screenshot wanted: the Fusion 360 enclosure model, and the printed gasket-groove test coupon. -->

---

## Phase 2: bare-metal migration and a custom PCB

This is the academic half, and it is the reason the architecture has a clean seam in it.

**The plan:** demote the ESP32-C3 to a radio bridge, and move all the control logic to a **TM4C123 (ARM Cortex-M4)** talking to it over UART.

Why bother, when the ESP32-C3 already works? Because doing it on the TM4C123 means doing it **at register level**, with no Arduino wrappers:

- GPIO and SysTick configured by writing registers directly
- Six-channel PWM for LED brightness control
- ADC for battery monitoring
- **ARM assembly `PRIMASK` critical sections** for the shared state between the UART receive path and the animation timer
- A **watchdog that fails safe**, and it fails safe through hardware that is already there, because the 10 kΩ gate pulldowns pull every segment dark if the microcontroller stops driving them. That is a design decision from [Part 1](/turn-signal/part-1-design/#architecture) paying off in Phase 2.

**And a custom board:** a two-layer **KiCad** BoosterPack carrier PCB, with an **antenna keepout** under the ESP32-C3 module. No copper pour, no traces under the antenna, which is the single easiest way to ruin a 2.4 GHz design on a two-layer board.

Trace widths will be split by function: wide pours for the LED return paths carrying up to ~400 mA, minimum-width signal traces for the four gate lines carrying about 320 µA. Those two numbers differ by three orders of magnitude, and the entire reason I know them is the measurements in Parts 2 to 5.

<!-- Screenshot wanted: the KiCad schematic and the two-layer board layout. -->

---

*The log continues. For what the project has taught me so far, see the [project overview](/turn-signal/#what-this-project-has-taught-me-so-far).*
