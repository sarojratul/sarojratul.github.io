# sarojratul.github.io

Personal site and project build logs, served by GitHub Pages with the minima theme.

## Layout

- `index.md`: home page. Intro, a card for the current project, then every post newest first.
- `turn-signal.md`: project overview at `/turn-signal/`. Status table, the build log index (generated automatically), specs at a glance, how the work plan is organised, and lessons so far.
- `about.md`: about page.
- `_posts/`: one markdown file per build-log entry, named `YYYY-MM-DD-part-N-slug.md`.
- `images/`: all photos, screenshots and diagrams, referenced as `/images/filename`.
- `_layouts/series-post.html` and `_includes/series-nav.html`: the "part N of M" box, full parts list, and Previous / Next links on every build-log entry.
- `_data/series.yml`: the name and overview URL for each series.
- `assets/main.scss`: small style additions on top of minima.

## Adding a build-log entry

Front matter for a new entry:

```yaml
---
layout: series-post
title: "Part 8: Short Descriptive Title"
date: 2026-09-27 12:00:00 -0600
series: turn-signal
part: 8
covers: "27 September 2026"
excerpt: "One sentence on what happens in this entry. Shown on the home page and the project overview."
permalink: /turn-signal/part-8-short-slug/
---
```

Conventions inside an entry:

- One short intro paragraph, then the table of contents block (copy it from any existing entry) if there are three or more sections.
- Each section is `## Title`, followed by `<p class="entry-date">27 September 2026</p>`, then Objective / Method / Result / What it means where that fits.
- `---` between sections.
- Image captions are an italic paragraph directly under the image, followed by `{: .caption}` on its own line.
- Units as symbols: Ω, kΩ, mΩ, µA, µF, ×.
- When a later entry changes an earlier number or decision, leave the original and add a `> **Update, <date>:** ...` note with a link to where it changed. Superseded, not deleted.
- Notes to self (photos still wanted, etc.) go in HTML comments: `<!-- Photo wanted: ... -->`.
- When a new entry changes project status, update the "Where it stands" and "At a glance" tables in `turn-signal.md`.

Nothing else needs editing: the parts list, navigation and home page pick new entries up automatically.

Posts dated in the future are not published by GitHub Pages, so keep `date` at or before the day you push.
