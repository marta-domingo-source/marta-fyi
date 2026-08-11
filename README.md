# marta-fyi

A personal site built on the structure of an Are.na **profile** page: a title row, three
control columns — Info / View / Order — and a grid of square cards.

```
index.html      the page
style.css       tokens + components
assets/         images go here
v1-portfolio/   an earlier, abandoned direction — kept for reference
```

## Ground rules

- **Zero dependencies.** No build, no npm, no framework, no CDN, **no font files.**
- **Almost no JavaScript.** View, Order and the pop-ups are CSS. The one exception is
  the Random order — about ten inline lines at the foot of `index.html`, because CSS has
  no source of randomness. With scripting off, Random simply behaves like Featured and
  nothing else changes.
- **No motion.**
- Open `index.html` and it works.

## Why there are no webfonts

Are.na's typeface "areal" is an Arial-metric clone, and its `--fonts-mono` token resolves to
the same stack. So `Arial, Helvetica, sans-serif` *is* the look — nothing to download.

## The system, taken from Are.na's stylesheet

| | |
| --- | --- |
| Greys | `#FFF` `#F7F7F7` `#EDEDED` `#DEDEDE` `#999` `#696969` `#333` `#000` |
| Accents | red `#B93D3D`/`#DFBEBE`, green `#238020`/`#B4D6B3` |
| Type | 12.5px · 16px · 28px · 32px |
| Spacing | 5px base: 5 / 10 / 15 / 20 / 25 / 35 / 45 / 65 / 80 / 100 / 130 |
| Borders | `1px solid` |
| Radius | 3px |
| Grid | four columns, `gap: 20px` |

In Are.na the red and green mark channel privacy. Here they mark category: green for
Writing, red for Teaching, default for everything else.

The header is just the name. The `More` disclosure holds LinkedIn, GitHub and CV.

## The grid is the layout

Are.na's profile is **one four-column grid**, and the three control columns sit on the first
three of it. Info aligns to card 1, View to card 2, Order to card 3 — exactly, to the pixel.
That single fact does most of the styling work, so `--cols` drives both grids and the
breakpoints move them together:

| Width | Columns |
| --- | --- |
| ≥ 1400px | 4 |
| 1000–1400px | 3 |
| < 1000px | 2 cards, controls stacked |

The page is near-full-bleed with 80px side padding, capped at `--page` (120rem) so cards
don't become absurd on a very large display. At 1920px that gives 425px squares, which is
what the reference has.

**Selection is marked by a dot and nothing else** — no weight or colour change on the
selected View/Order option, which is what Are.na does. Every option is bold and `#333`; the
light 3px tick to the left of each is inset top and bottom so the ticks read as segments
rather than one continuous rule.

## How the interactions work

View and Order are radio groups. A `:has()` test on `.page` sees which one is checked:

```css
/* View — hide what doesn't match */
.page:has(#view-projects:checked) .grid > li:not([data-sec="projects"]) { display: none }

/* Order — pick which rank feeds the CSS order property */
.page:has(#order-az:checked) .grid > li { order: var(--o-az) }
```

Each `<li>` carries its rank in every sort mode as an inline custom property:

```html
<li data-sec="projects" style="--o-new:2;--o-old:10;--o-az:3">
```

Three things to know before editing:

1. **The grid items are the `<li>`, not the `<a class="card">` inside.** Put `data-sec` or
   `--o-*` on the anchor and sorting silently does nothing, while filtering leaves empty
   cells where the hidden cards were.
2. **Every card needs all three ranks** (`--o-new`, `--o-old`, `--o-az`). A missing one resolves to `0` and that card jumps to
   the front. Regenerate the ranks as a set rather than hand-editing them.
3. **Inline custom properties beat every stylesheet rule**, which is exactly why this works:
   the switchable thing is *which* property gets read, never the value itself.

The radios are positioned off-screen at 1×1px with `opacity: 0` rather than
`display: none`, so they stay keyboard-focusable.

**Pop-ups use `:target`.** Each card links to `#slug`; the matching `.modal` is
`display: none` until it's the URL target. `body:has(.modal:target) { overflow: hidden }`
stops the page behind from scrolling, and Close links point at `#grid` so closing returns
you to the grid rather than the top of the page. The back button closes a pop-up, which is
what a URL change should do.

Two things this approach doesn't give you, both fixable only with script: **Escape doesn't
close**, and **focus isn't trapped** inside the sheet. Swap in `<dialog>` if either
matters more than staying script-free.

## Experience is one square

Rather than a card per role, Experience is a single square whose pop-up holds the whole CV:
five roles, each with title, org (linked, with an ↗), years, a one-line summary in full ink,
and its bullets in grey. Markup lives in `.roles` inside `#experience`.

**That text is Marta's own wording, verbatim.** It uses American spellings — *organization*,
*prioritization*, *prioritized* — and those are deliberate, not typos to fix. It also
supersedes the PDF CV in places: the 2019–2023 role is **Senior Product Designer**, not
"Product Design Lead", and several bullets are reworded. Take this file as the source of
truth over the PDF.

The same treatment would suit Teaching, which is currently four separate squares.

## Current options

**View** — All · Currently · Projects · Experience · Teaching · Writing

**Order** — Featured (document order) · Newest first · Oldest first · Alphabetical · Random

Featured runs: Now, then projects, then experience, then teaching, then writing — what's true
today, then the work, then the record, then the thinking.

Random reshuffles every time you click it, not just on load.

An earlier "Products, systems, teams" order — sorting by scope — was dropped to match the
five you asked for. Re-adding it means one radio, one CSS rule, and one rank per card.

## Verifying a change

```sh
open index.html
```

Headless Chrome can't click, so to test a View or Order state, copy the file to `/tmp`, move
the `checked` attribute to a different radio, and render that. Assert on computed values —
which `<li>` have `display: none`, and their visual order by bounding box — rather than
eyeballing a screenshot. Note the viewport floors at ~500px, so a sub-500px render is a
500px viewport cropped, which mimics an overflow bug that isn't there.

Contrast is checked numerically, both schemes.

## State

Eleven squares. Real content: the Info column (though Barcelona is inferred, not stated), the
Experience square and its full five-role pop-up, and the four Teaching squares. Placeholder:
three projects, two notes, the Now card, those pop-up bodies, and the LinkedIn and CV links.
**`20XX` means there is no date for it yet** — those cards still sort correctly, but the shown
date is a slot.

Marta's Are.na profile is auth-gated, so the real channel names couldn't be read from it.
