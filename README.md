# bike-turn-signal
A wireless, backpack-mounted LED turn-signal system personal project for cycling, controlled from a handlebar remote over ESP-NOW.


Design and build of a bicycle turn-signal system: a 32-LED amber arrow panel mounted on a backpack, driven by an ESP32-C3 and switched in four independent segments (left arrowhead, two shared body segments, right arrowhead) through logic-level AO3400A MOSFETs. A second ESP32-C3 on the handlebars sends held/released button state over ESP-NOW; the receiver runs the whole behaviour model, which is a direction-dependent centre-out sweep, continuous repeat while held, four trailing cycles after release, and immediate cancel on any new press. Both units run on NiMH packs the builder already owns (4×AA for the panel, 3×AAA for the controller), with a TPS63070 buck-boost regulating logic power while the LED panel runs directly off the raw 4.8 V rail. Enclosures are 3D-printed with gasketed lids for weather resistance.

The project is being worked stage by stage with verification at each step: LTspice simulation of the LED strings and MOSFET switching, Wokwi and Tinkercad rehearsals of the firmware and breadboard topology, then physical bring-up with a multimeter and an Analog Discovery 2. A second phase migrates the control logic to a TM4C123 Cortex-M4 at register level, with a custom two-layer KiCad BoosterPack carrier board, as a coursework tie-in.

Current status (Sept 2026): breadboard build of the four-segment panel assembled; gate networks measure correct, LEDs not yet lighting, debugging in progress.
