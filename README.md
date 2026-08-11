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
- **Two small inline scripts**, for the only two things CSS genuinely can't do:
  **Random** (no source of randomness) and **Search** (no text matching). Everything else
  — View, Order, the pop-ups, the More popover — is CSS. With scripting off, Random
  behaves like Featured and the search box hides itself rather than sitting there dead.
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
| Type | 12.5px meta · 14.4px UI · 16px reading · 28px card · 32px title |
| Spacing | 5px base: 5 / 10 / 15 / 20 / 25 / 35 / 45 / 65 / 80 / 100 / 130 |
| Borders | `1px solid` |
| Radius | 3px |
| Grid | four columns, `gap: 20px` |

In Are.na the red and green mark channel privacy. Here red marks a card that isn't quite like
its neighbours — currently just Teaching, which shares the experience section with Experience
but reads as a different kind of thing. It's keyed off `data-accent` on the `<li>`, not
`data-sec`, so accent and section can diverge. Green is unused.

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

**Selection** is three things at once, which is what Are.na does: a dot at the far left, the
selected row's segment of the tick rail darkened to `#696969`, and the label a shade darker
(`#000` against `#333`). The rail is *continuous* — the per-item bars butt together with no
inset, so it reads as one line with a highlighted spot rather than a row of dashes.

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

**Search** matches each card's own text *and* everything in the pop-up behind it — the
description and the fact rows included. So "governance" finds Experience even though the
card never says the word, and "strategic design and innovation" finds Elisava from a fact
row alone. It marks
non-matches with a `data-nomatch` attribute rather than an inline `display`, because an
inline style would beat the View filter's stylesheet rule and the two would fight; as
attributes they compose. Escape clears the field. It searches the cards and their pop-ups,
not the Info column.

**`More` is a popover** — a native `<details>` whose `<ul>` is absolutely positioned, so it
floats over the page instead of pushing it down. No script.

**Pop-ups** follow the structure of Are.na's block detail: a large **content pane** on the
left and a **metadata sidebar** on the right, in a sheet that nearly fills the viewport.

| | |
| --- | --- |
| Pane | whatever the substantial content is — the visuals, or long-form like the CV |
| Sidebar | prev / next / close, title, a description, then label-value fact rows |

At 1700px that's a 1182px pane beside a 415px sidebar. Below 900px they stack, **sidebar
first**, so the title and description arrive before a screenful of images.

The pane centres short content with `margin-block: auto` on its child rather than
`justify-content: center`, because auto margins collapse to zero when the content is taller
than the pane, where centring would clip the top and make it unreachable.

They work on `:target`, so opening and closing needs no script and the back button closes
them. Close links point at `#grid` so closing returns you to the grid rather than the top of
the page. **Prev/next walk document order**, not whatever Order is currently selected — they
are static links generated per card, and they wrap around at both ends.

Two limits of doing this without script: **Escape doesn't close**, and **focus isn't
trapped** in the sheet. Swap in `<dialog>` if either matters more than staying script-free.

## Experience is one square

Rather than a card per role, Experience is a single square whose pop-up holds the whole CV:
five roles, each with title, org (linked, with an ↗), years, a one-line summary in full ink,
and its bullets in grey. Markup lives in `.roles` inside `#experience`.

**That text is Marta's own wording, verbatim.** It uses American spellings — *organization*,
*prioritization*, *prioritized* — and those are deliberate, not typos to fix. It also
supersedes the PDF CV in places: the 2019–2023 role is **Senior Product Designer**, not
"Product Design Lead", and several bullets are reworded. Take this file as the source of
truth over the PDF.

Teaching now gets the same treatment: one square, four schools listed in the pane, reusing
the `.roles` markup rather than a second set of rules.

## Current options

**View** — All · Currently · Projects · Experience · About

**Order** — Featured (document order) · Newest first · Oldest first · Alphabetical · Random

Featured runs: Now, then projects, then Experience, then Teaching, then About — what's true
today, then the work, then the record, then the person.

Teaching is deliberately **not** a View option. It sits in the experience section, so
`View → Experience` shows both squares. Writing was removed entirely.

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

Seven squares. Real content: the Info column (though Barcelona is inferred, not stated), the
Experience square and its full five-role pop-up, and the Teaching square's four schools.
Placeholder: three projects, the Now card, those pop-up bodies, two of the three About
paragraphs, and the LinkedIn, GitHub and CV links.
**`20XX` means there is no date for it yet** — those cards still sort correctly, but the shown
date is a slot.

Marta's Are.na profile is auth-gated, so the real channel names couldn't be read from it.
