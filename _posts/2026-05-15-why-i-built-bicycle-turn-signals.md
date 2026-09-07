---
layout: post
title: "Why I Built Wireless Bicycle Turn Signals"
date: 2026-05-15
categories: bike-turn-signals
---

## May 2026: The money problem, solved

I have wanted to build things like this for years. Two things were always in the way: I did not know enough electronics, and I did not have the money for tools.

The first one fixed itself. By the end of second year I had EE 221 (analog electronics) and EE 232 behind me, and for the first time I looked at a circuit and felt like I could reason about it instead of just copying a lab manual.

The second one fixed itself in a way I did not expect. I won several awards that year, and one of them, the **Ritenburg Family Foundation Bursary in Engineering, $3,500**, was large enough that I got brave. That is the honest word for it. Not "I made a strategic investment in my professional development." I got brave enough to spend award money on a hobby.

What I bought:

- Soldering iron and station, solder, fume extractor, heat gun, glue gun
- **Digilent Analog Discovery 2**, a USB oscilloscope / logic analyzer / waveform generator / power supply
- Multimeters, breadboards, jumper wire, perfboard, helping hands
- A 480-piece, 24-value ceramic capacitor assortment; a resistor assortment
- Everything on the bill of materials for this project (DigiKey.ca)

The whole time I was checking out I had one thought going: *what if I lose interest after spending all of this?* That is a real risk when you buy tools for a hobby you have not started yet.

It did not happen. This became the thing I do when I am not working or studying. It is my favourite hobby now, and the AD2 in particular turned out to be the single best purchase. It is what turns "it doesn't work" into "it doesn't work *here*."

![24-value, 480-piece ceramic capacitor assortment](/images/capacitor_kit.jpg)

*Part of what the bursary bought. A 480-piece, 24-value ceramic assortment, 10 pF to 10 uF, 50 V. The 100 nF row is the one this project actually needed.*

*Photo wanted: the full workbench, soldering station and fume extractor and AD2 together.*

---

## May 2026: Why turn signals specifically

Three reasons, all of them from actually riding here.

1. **My right wrist is injured.** Signalling a left turn means holding the bar with my weak hand while my good arm is out in the air. On a rough Saskatoon street that is a genuinely bad position to be in.
2. **Night.** Drivers do not see an arm. They see lights.
3. **Impatient drivers.** If someone honks or does something reckless behind me, I need both hands on the bar *right then*, which is exactly the moment I am supposed to be signalling with one of them.

A hand signal costs you a hand at the moment you most need it. That is the whole design brief.

---

## May 2026: It was a jacket first

The original plan was a jacket with LEDs sewn into the back. I even named the project folder `Jacket turn signal`, which is why every document in this repository still says "jacket."

I gave up on that idea on a bike ride the next day. Three problems, in order of how badly they killed it:

- **Nobody wears a jacket in July.** The thing needs to work in summer, and in summer I am in a t-shirt. But almost everyone riding has a **backpack**.
- **Cost and materials.** Flexible wiring, textile-safe mounting, washability: all of it is a materials problem I did not want to solve on my first real project.
- **It has to survive.** A jacket gets stuffed in a bag. A rigid enclosure on a backpack does not.

So: **backpack-mounted rigid panel**, with the jacket kept in the documents as an explicit out-of-scope fallback. I have not deleted that section. If I ever want it, the reasoning is still there.

> **Log discipline, decided here:** superseded designs get marked SUPERSEDED, not deleted. Every time I have gone back to check "wait, why did I rule that out," it has saved me an hour.

---
