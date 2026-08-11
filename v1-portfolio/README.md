# marta-fyi

Personal portfolio site. Hand-written static HTML, structurally modelled on
[tomreeve.co](https://tomreeve.co/) and restyled from there.

## Ground rules

- **Zero dependencies.** No build step, no npm, no framework, no CDN. Every file here is
  something a browser can open directly off disk.
- **No JavaScript.** Motion is CSS scroll-driven animation (`animation-timeline`), layered
  on inside `@supports` so browsers without it get a correct static page. See the caveat
  under *Motion* below.
- Fonts will be self-hosted `.woff2` in `assets/fonts/` — never linked from Google Fonts.

## Files

```
index.html    the homepage — 8 sections
style.css     tokens, layout primitives, components, motion
work/         one page per case study            (Phase 8)
assets/       fonts/ and img/                    (Phases 3 and 7)
```

## The one structural idea

Nearly every section is the same grid: a 280px label column plus a content column,
`gap: 4rem`, collapsing to one column at 860px. It's the `.row` class, and it's reused by
the intro, the point-of-view rows, and every experience row. Change `--label-col` or
`--col-gap` and the whole page re-proportions.

Other primitives worth knowing before editing:

| Class | Does |
| --- | --- |
| `.wrap` | 1200px max, `--wrap-pad` inline padding |
| `.row` | the label + content grid |
| `.section-head` | ink rule + title left, count right |
| `.list` | hairline-separated rows, with `.k` `.t` `.d` inside |
| `.mono` | metadata voice — uppercase, 0.72rem, tracked. **Never used on prose.** |
| `.site-head.over` | header overlaying a hero; plain `.site-head` stays in flow |

Hierarchy comes from weight (`--w-strong`, 550) and from `--ink` vs `--mid` — not from
colour. There is no accent colour yet, on purpose; one arrives in Phase 7.

## Motion

| What | How |
| --- | --- |
| Hero word-scrub | per-word colour, driven by the 260svh `.hero-track` |
| Section reveals | 16px rise, once, on entry |
| Scroll hint | 1.8s 4px bob |
| Card hover | `--block` → `--block-hover`, 0.3s. The entire hover treatment. |

**Support caveat:** `animation-timeline` is Chrome/Edge 115+, Safari 26+, Firefox 156+ —
roughly 85% of browsers. Everything is therefore authored finished-state-first, with motion
added inside `@supports (animation-timeline: scroll())`. The rest get a complete static
page. If the scrub ever needs to work everywhere, the same `@keyframes` can be driven by a
~35-line scroll listener instead; nothing else changes.

## Phase state

- [x] **0** — skeleton, tokens, all 8 section shells (lorem)
- [x] **1** — header + hero, including the scrub
- [x] **2** — Currently *(drafted from Marta's notes; scope claims need her confirmation)*
- [ ] **3** — work grid — *built and styled; deferred on content. Cards show
      labelled slots, not lorem. Needs per project: title, `·` descriptor, 2–3
      tags, an outcome line with one hard number, and media.*
- [ ] **4** — point of view
- [x] **5** — experience, teaching, studies — *all facts from Marta's CV. The
      planned "off the clock" section was dropped in favour of Teaching and
      Studies, which are real and more distinctive.*
- [ ] **6** — contact footer
- [ ] **7** — type and palette: make it hers
- [ ] **8** — first case page
- [ ] **9** — meta, favicon, print, a11y sweep

Placeholders in `index.html` are tagged `[PH-n]` with the phase that resolves them.

## Verifying a change

```sh
open index.html
```

Headless Chrome for repeatable renders. Three traps on this machine:

1. **The viewport floors at ~500px** — a `--window-size=390,…` render is a 500px viewport
   cropped to 390px, which mimics an overflow bug that isn't there. Measure instead:
   compare `scrollWidth` to `clientWidth` and list elements whose `right` exceeds the
   viewport.
2. **URL fragments are ignored** — `file://…#work` screenshots the top of the page.
3. **A `svh`-based hero scales with the window**, so the usual one-tall-render trick keeps
   the sticky panel pinned and never reaches the content. Render a copy with
   `--hero-track: auto` and `.hero-sticky { position: static }` to inspect the sections.
4. **Screenshot mode never captures scrolled sticky state** — those renders come back blank.
   Verify scroll-driven motion by *probing computed values* instead: inject a script that
   scrolls and writes results to `html[data-probe]`, then read it out of `--dump-dom`. Two
   things make that work:
   - Scroll **at parse time** (a script at the end of `<body>`), not on `load`. Scrolling
     later means the animation timeline is sampled before the scroll lands, and the values
     come back stale.
   - `scroll-behavior: smooth` silently swallows programmatic scrolls under
     `--virtual-time-budget`. Set `scrollBehavior = 'auto'` and pass `behavior: 'instant'`.
5. **Running animations outrank inline styles.** To mock a mid-animation state by injecting
   `style="color:…"`, you must also set `animation-name: none !important`, or the animation's
   fill state wins and you'll photograph the wrong frame.

Contrast is checked numerically, not by eye. Current state: `--ink` on `--paper` 17.6:1,
`--mid` on `--paper` 5.1:1, `--mid` on `--block` 4.5:1 — all AA. `--ghost` is 1.4:1 by
design; it's the hero text mid-animation and it resolves to full `--ink`.
