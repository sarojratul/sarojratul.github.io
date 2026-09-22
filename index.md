---
layout: home
list_title: All entries, newest first
---

Third-year Electrical Engineering at the University of Saskatchewan, Class of 2028, Engineering Co-op. This site is where I keep build logs for the things I make, written as I go rather than afterwards.

<div class="project-card" markdown="1">

## Current project: COMET Air Mouse

A motion-controlled Bluetooth mouse: an ESP32-C3 reads hand movement from an MPU6050 gyroscope and moves the cursor, no desk required. Built from CoreComet Industries' open-source design. Just started, nothing logged yet.

<p class="button-row"><a href="/comet-air-mouse/">Project overview</a></p>

</div>

<div class="project-card" markdown="1">

## On hold for the winter: wireless bicycle turn signal

![The 32-LED arrow laid out on the breadboard](/images/arrow_topdown.jpg)

A 32-LED amber arrow that mounts on a backpack, driven by an ESP32-C3 and commanded over ESP-NOW from a handlebar remote. All four segments are working on the breadboard and sweeping under firmware control. I have paused it while I am off the bike for the winter, and will pick it back up over the winter so it is ready for spring. Next up is the handlebar transmitter and the radio link.

<p class="button-row"><a href="/turn-signal/">Project overview</a><a href="/turn-signal/part-0-why/">Start at Part 0</a>{% assign latest = site.posts | where: "series", "turn-signal" | sort: "part" | last %}<a href="{{ latest.url | relative_url }}">Latest: Part {{ latest.part }}</a></p>

</div>
