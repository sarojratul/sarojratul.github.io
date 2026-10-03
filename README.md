# sarojratul.github.io

Personal site and hardware build logs, served by GitHub Pages. No theme: the layouts and the stylesheet are in this repo.

## Layout

- `index.md`: home page. Intro text; the project cards and recent entries come from `_layouts/home.html`.
- `turn-signal.md`, `comet-air-mouse.md`: project overview pages (`layout: project`). Front matter sets the lede, hero image and the facts grid; the build-log list is `{% include build-log.html series="..." %}`.
- `about.md`: about page.
- `_data/series.yml`: one entry per project (title, URL, summary, status, card image, tags). The home page cards are generated from it.
- `_posts/`: one file per build-log entry, `YYYY-MM-DD-part-N-slug.md`.
- `_layouts/`: `default` (header, footer), `home`, `page`, `project`, `series-post` (build-log entries).
- `_includes/`: `build-log.html` (numbered entry list), `series-nav.html` (previous / next and all parts).
- `assets/main.scss`: the whole stylesheet, plain CSS. Light and dark themes follow the visitor's system setting.
- `images/`: photos, screenshots and diagrams, referenced as `/images/filename`.

## Adding a build-log entry

```yaml
---
layout: series-post
title: "Part 1: Short Descriptive Title"
date: 2026-10-03 12:00:00 -0600
series: comet-air-mouse
part: 1
covers: "3 October 2026"
excerpt: "One sentence on what happens in this entry. Shown under the title and in the build-log list."
permalink: /comet-air-mouse/part-1-short-slug/
---
```

Use only one colon in the title, after "Part N". The layout shows the text after it as the heading.

Inside an entry:

- The excerpt is shown as the lede, so the body starts with the contents box (copy it from any entry) or the first section.
- Each section is `## Title`, then `<p class="entry-date">3 October 2026</p>`, then Objective / Method / Result / Conclusion where that fits.
- Image captions are an italic paragraph directly under the image, followed by `{: .caption}` on its own line.
- Notes and lessons are blockquotes, which render as callouts.
- Units as symbols: Ω, kΩ, mΩ, µA, µF, ×. No em dashes.
- When a later entry changes an earlier number or decision, leave the original and add a `> **Update, <date>:** ...` note linking to where it changed.
- Notes to self go in HTML comments: `<!-- Photo wanted: ... -->`.
- When status changes, update "Where it stands" and the facts in the project page's front matter, and `status_label` in `_data/series.yml` if needed.

Posts dated in the future are not published by GitHub Pages, so keep `date` at or before the day you push.
