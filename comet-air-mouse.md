---
layout: page
title: COMET Air Mouse
permalink: /comet-air-mouse/
description: A motion-controlled Bluetooth mouse built on an ESP32-C3 and an MPU6050 gyroscope. Project overview and build log index.
---

A mouse that follows my hand through the air instead of sliding on a desk. An ESP32-C3 reads hand movement from an MPU6050 gyroscope and sends it to the computer as a standard Bluetooth mouse, so no driver or receiver dongle is needed. I am building my own from CoreComet Industries' open-source [COMET Air Mouse v1.0.0 release](https://github.com/CoreCometIndustries/COMET-AIR-MOUSE/releases/tag/v1.0.0).

## Where it stands

*As of 21 September 2026.* Just started. Nothing is built yet.

## The build log

{% assign series_posts = site.posts | where: "series", "comet-air-mouse" | sort: "part" -%}
{%- if series_posts.size > 0 -%}
<ol class="parts-list">
{%- for p in series_posts %}
  <li>
    <a class="parts-list__title" href="{{ p.url | relative_url }}">{{ p.title | escape }}</a>
    <div class="parts-list__meta">{{ p.covers }}</div>
    <p>{{ p.excerpt | strip_html }}</p>
  </li>
{%- endfor %}
</ol>
{%- else -%}
*No entries yet. They will appear here as the build goes.*
{%- endif %}
