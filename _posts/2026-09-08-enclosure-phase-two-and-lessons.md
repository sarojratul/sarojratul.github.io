---
layout: post
title: "Enclosure, Phase Two, and What This Has Taught Me"
date: 2026-09-08
categories: bike-turn-signals
---

Not built. Recording the decisions that are already made, because they were made for reasons.

**3D-printed, in-house, replacing the Hammond 1553WBBK.** Both units: handlebar controller and backpack panel.

- **Gasket groove in the lid**, taking either silicone cord or a printed gasket.
- **Heat-set brass inserts** for the lid screws, rather than threading into plastic, because a lid that gets opened repeatedly will strip printed threads.
- **Undecided:** print material and process. PETG, ASA, PLA and resin are all still on the table, and the brass inserts are not sourced yet because the choice depends on it. PLA is out for anything left on a bike in a Saskatchewan summer.

**Work plan step C13b, which comes before assembly:** model the enclosure around the bulkiest component first, design the gasket groove and insert bosses, and **print a test coupon before printing the real thing.** Same principle as simulating before breadboarding: find out whether the interference fit works on a part that takes fifteen minutes, not one that takes six hours.

**The weatherproofing claim has a gate on it.** A printed lid with a gasket does not earn an "IP54-equivalent" claim from me until test T12, the rain test, actually confirms the seal. Until then the documents say "designed for," not "rated."

*Screenshot wanted: the Fusion 360 enclosure model, and the printed gasket-groove test coupon.*

---

This is the academic half, and it is the reason the architecture has a clean seam in it.

**The plan:** demote the ESP32-C3 to a radio bridge, and move all the control logic to a **TM4C123 (ARM Cortex-M4)** talking to it over UART.

Why bother, when the ESP32-C3 already works? Because doing it on the TM4C123 means doing it **at register level**, with no Arduino wrappers:

- GPIO and SysTick configured by writing registers directly
- Six-channel PWM for LED brightness control
- ADC for battery monitoring
- **ARM assembly `PRIMASK` critical sections** for the shared state between the UART receive path and the animation timer
- A **watchdog that fails safe**, and it fails safe through hardware that is already there, because the 10 kΩ gate pulldowns pull every segment dark if the microcontroller stops driving them. That is a design decision from Part 1 paying off in Part 5.

**And a custom board:** a two-layer **KiCad** BoosterPack carrier PCB, with an **antenna keepout** under the ESP32-C3 module. No copper pour, no traces under the antenna, which is the single easiest way to ruin a 2.4 GHz design on a two-layer board.

Trace widths will be split by function: wide pours for the LED return paths carrying up to ~400 mA, minimum-width signal traces for the four gate lines carrying about 320 µA. Those two numbers differ by three orders of magnitude, and the entire reason I know them is the measurements in Parts 2 and 3.

*Screenshot wanted: the KiCad schematic and the two-layer board layout.*

---

# What this project has actually taught me

Listed plainly, because this is the part I would want to read.

**Analog:** LED forward-voltage behaviour and why current-setting resistors are not optional. MOSFET threshold and R<sub>DS(on)</sub> behaviour at low gate drive, and why "logic-level" is a specification you check rather than a word you trust. Why a buck-boost and not an LDO on a battery that sags.

**Simulation:** LTspice DC operating point, DC sweep and transient analysis. Building a device model from a datasheet curve when the manufacturer does not provide one. Knowing what a simulation *cannot* tell you, which is most of Part 3.

**Firmware:** a state machine with a deliberate seam between behaviour and transport, so the same logic runs against buttons or a radio. Timing that does not free-run. Tuning a constant against observed behaviour rather than guessing it.

**Instrumentation:** the multimeter in series, in continuity mode, in diode mode. The AD2 for what it is genuinely good at. Identifying a part's pinout empirically instead of trusting a silkscreen.

**Process, and this is the one I would defend hardest:** writing pass criteria before running the test. Working outward from the source with an ordered ladder instead of guessing at causes. Marking superseded designs instead of deleting them. Recording what a passing test does *not* prove. Checking every proposed part against its own datasheet before spending a dollar.

**And a soldering iron:** which I now know how to look after, at a cost of nine MOSFETs, three burns and a mark on the table.

---

*Log continues. Next entry: one segment, from scratch.*
