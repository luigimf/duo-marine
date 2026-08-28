# DUO MARINE — Frontend Specification

Reference implementation: `site/index.html`. This document describes it so the
UI can be rebuilt in React, Vue, Astro, Svelte or plain templates without
reverse-engineering the file.

---

## 1. Design tokens

### 1.1 Colour

| Token | Hex | Use |
|---|---|---|
| Tyrrhenian blue | `#071B2B` | Dark sections, headings on light, boot screen, footer |
| Foam | `#FDF9F5` | Light section ground, text on dark |
| Brand orange | `#F47F4B` | **All** hover states, product names, icons, active accents |
| Seafoam | `#8FB5B5` | Eyebrow labels on dark |
| Bronze | `#CB9865` | Eyebrow labels on dark (jet ski variant) |
| Slate grey | `#8F9399` | Eyebrow labels on light, meta text, counts |
| Body slate | `#3d4b57` | Paragraph text on light |

Alpha rules used throughout:

- Hairlines on light: `rgba(7,27,43,.14)`; inner grid lines `rgba(7,27,43,.10)`
- Hairlines on dark: `rgba(253,249,245,.18)`
- Muted text on dark: `rgba(253,249,245,.55)`; body on dark `rgba(253,249,245,.76)`

**Only two background colours exist in the whole page** — `#071B2B` and
`#FDF9F5`, alternating by section. Do not introduce a third.

### 1.2 Type

Two families, both self-hosted from `assets/fonts/` with a Google Fonts link
as fallback:

- **Barlow** — all display, headings, labels, buttons, numerals. Weights 100–900
  plus italics are all loaded (the client edits weights directly).
- **Funnel Sans** — body copy and subtitles only.

| Role | Spec |
|---|---|
| Hero headline | Barlow 200, `clamp(44px,8.4vw,150px)`, line-height .92, `letter-spacing .02em`, uppercase |
| Section h2 | Barlow 300, `clamp(36px,5.4vw,82px)`, line-height .98, uppercase |
| Product h3 | Barlow 400, 48px, line-height .9, uppercase, colour `#F47F4B` |
| Product subtitle | Funnel Sans 300, `clamp(17px,1.5vw,22px)`, line-height 1.4 |
| Eyebrow label | Barlow 400, 11px, `letter-spacing .28em`, uppercase |
| Spec card label | Barlow 400, 10px, `letter-spacing .2em`, uppercase, `#8F9399` |
| Spec card value | Barlow 300, 26px, line-height 1 |
| Accordion title | Barlow 400, `clamp(15px,1.5vw,20px)`, `letter-spacing .08em`, uppercase |
| Body paragraph | Funnel Sans 300, 16px, line-height 1.7, `text-wrap: pretty` |
| Button / CTA | Barlow **500**, 12px, `letter-spacing .22em`, uppercase |

### 1.3 Spacing & geometry

- Section padding: `clamp(72px,9vw,140px)` vertical, `clamp(20px,3.2vw,52px)` horizontal
- Content max-width: **1520px** (1180px for the FAQ column)
- Radii: pills `999px` for buttons and tabs; **everything else is square** — no
  rounded cards anywhere
- No box-shadows except the lightbox image (`0 40px 90px rgba(0,0,0,.45)`)

---

## 2. Page structure

```
<header>              fixed, z-60, translucent, blurred
#top                  Hero — 100svh
#vessels              RIBs — foam ground
#watercraft           Jet Skis — blue ground
#care                 Care programme — foam ground
#contact              Contact — blue ground, wave divider on top edge
#faq                  Questions — foam ground
<footer>              blue ground
#lightbox             fixed overlay, hidden until opened
#welcome              boot cover, injected + removed by script
```

Nav order: RIBs · Jet Skis · Care · Questions · Enquire.

---

## 3. Boot screen (`#welcome`)

Runs before anything else is visible. Owned by an inline IIFE in `<head>`, not
by the component tree — a re-render must never be able to resurrect it.

1. `html, body { background: #071B2B !important }` is set in a `<style>` at the
   very top of `<head>` so the **first painted frame** is brand blue, never white.
2. A full-bleed `#welcome` div is appended to `<body>`, centred, containing the
   wave mark as a masked `<i>` (`-webkit-mask: url(wave-mark.svg) center/contain`)
   filled `#FDF9F5`, width `min(38vw,260px)`, min-width 132px, aspect 279/105.
3. Mark animates in: `bootMark .78s cubic-bezier(.16,.84,.24,1) .12s both` —
   `translate3d(0,14px,0) scale(.965)` + opacity 0 → rest.
4. Dismissal waits for **fonts ready and the first 8 non-gallery images decoded**
   (`img.decode()`), with `MIN = 820ms` and a hard `MAX = 3000ms` failsafe.
5. On lift: `body[data-booted]` is set in the same frame the cover starts moving
   (this hands the foam ground back to the page with no blue edge exposed), then
   `#welcome[data-done]` runs `bootOut .58s` (opacity + `translate3d(0,-3%,0)`)
   and the mark runs `bootMarkOut .32s`. Node removed after 900ms.

While `body[data-booting]` is set, all page animations are paused and
`.shot-img` transitions are disabled — the page must be revealed complete, not
mid-load.

**Do not** animate a mask-clipped element on a loop or animate `filter: blur`
here. Both were tried; both jitter. One GPU-composited transform only.

---

## 4. Hero (`#top`)

- `height: 100svh`, `overflow: hidden`, ground `#071B2B`
- `<picture>`: `assets/hero-mobile-v2.webp` under 900px, `assets/hero-27gt-malta.webp` above
- Image sits in a `data-parallax="0.14"` layer at `top:-14%; height:128%`, driven by
  a scroll-linked animation (`animation-timeline: scroll(root)`, range `0 110vh`)
  with a JS fallback for browsers without scroll-driven animations
- Bottom gradient scrim into `#071B2B` for text legibility
- Headline: **THE WATERS, ON YOUR OWN TERMS**
- A "See the fleet ↓" cue and a short supporting paragraph, both fading in on a delay

---

## 5. Product sections (`#vessels`, `#watercraft`)

Identical mechanics, inverted colours. `#vessels` is foam with blue text;
`#watercraft` is blue with foam text.

### 5.1 Tabs

A pill-bordered row of `<button>`s, one per product. Active tab: filled
`#071B2B` with `#FDF9F5` text (inverted on the dark section). Inactive:
transparent. Hover: `#F47F4B`.

- `#vessels` — MV 23GT · MV 27GT
- `#watercraft` — GP SVHO · FX Limited

Selecting a tab shows only that panel (`display: none` on the others) and runs a
cross-fade: `panelInA`/`panelInB` alternating per click so re-selecting
re-triggers, `1.6s cubic-bezier(.22,.62,.24,1)`, from `opacity 0`
`translateY(30px) scale(.992)` and only 55% opaque at the 45% keyframe — a slow
dissolve, not a quick appear-then-drift.

**Tab state is per-section.** Choosing a jet ski must not re-animate the RIB
panel, and vice versa; keep two independent state keys and two tick counters.

### 5.2 Product hero row

Two-column grid, `minmax(320px,1fr)`, gap `clamp(28px,4vw,72px)`:

- Left: eyebrow, `<h3>` product name in `#F47F4B`, subtitle, description
  paragraph, a hairline-bounded stat row, and an **Enquire** CTA (Barlow 500)
  that smooth-scrolls to `#contact`
- Right: product cutout PNG→WebP (`assets/mv23gt.webp` etc.) with the engine
  image offset below it. On mobile the engine drops lower and scales down so the
  two never overlap.

### 5.3 Technical card

`display: grid; grid-template-columns: repeat(auto-fit, minmax(min(180px,100%),1fr))`
with 1px `rgba(7,27,43,.10)` borders forming a lattice (`border-left` on the
container, `border-right`+`border-bottom` on each cell).

Each cell: 26px/24px padding, uppercase 10px label, 26px Barlow 300 value.
Cells rise in on scroll (`riseSm 1.1s`, view timeline, `entry 0% → 38%`).

Example (MV 23GT): Overall length `6.99 m`, Overall width `2.68 m`, plus berths,
engine range, fuel capacity, max persons, CE category. Full values are in the
HTML and originate from the MV Marine and Yamaha PDFs.

### 5.4 Gallery

Between the Enquire CTA and the technical card. Width always matches the
technical card exactly (same flex column).

- `repeat(N, 1fr)` where **N = 6** ≥1100px, **3** ≥660px, **2** below —
  always full rows, never a ragged tail
- Cells are `aspect-ratio: 1/1; overflow: hidden`
- Images are `object-fit: cover`, loaded from `assets/gallery/{id}.webp` where
  id is `g23-1…6`, `g27-1…6`, `ggp-1…6`, `gfx-1…6`
- **Lazy loading:** `src` is held in `data-src` and assigned by an
  IntersectionObserver with `rootMargin: '600px 0px'`. A 2.2s fallback loads any
  still-unloaded images **in the visible panel only** — loading all 24 at once
  queues them against the ones actually on screen.
- Until an image decodes, a skin shows: `rgba(7,27,43,.045)` ground, the masked
  wave mark at `min(46%,88px)` drifting (`slotWave 2.1s`), and a light sweep
  across it (`slotSweep 1.9s`). Image fades in over it on load.
- **Hover: no scale.** A `#071B2B` veil at `opacity .45` over `.4s`. Driven by
  `[data-shot]:hover .shot-veil` — the veil needs the parent's hover state.

### 5.5 Accordions

Grouped spec detail below the technical card. Each header is a full-width
`<button>`: title on the left, **entry count** (zero-padded, e.g. `05`) beside
it, `+` indicator on the right rotating to `×` when open.

RIBs — five groups each: Console & seating (05), Deck & structure (06),
Systems (04), Equipment & accessories (09), Finishes (04).
Jet skis — two groups each: Technical, Features.

**All accordions default closed**, including the FAQ.

Open/close uses `display: grid` with `grid-template-rows: 0fr → 1fr` and
`transition: grid-template-rows .5s`, with the inner element `min-height: 0;
overflow: hidden` — animates real content height without measuring it in JS.

### 5.6 Supplier credit

After the last accordion, a "Brought to you by" line with the supplier mark —
MV Marine for RIBs, Yamaha for jet skis. The Yamaha mark must be **pure white**
on the blue ground.

---

## 6. Lightbox (`#lightbox`)

Opens on clicking any filled gallery cell.

- Fixed, `z-index: 400`, `rgba(7,27,43,.94)` + `backdrop-filter: blur(10px)`
- Main image: `max-height: min(76vh,1100px)`, `object-fit: contain` — the
  **full uncropped frame**, unlike the 1:1 grid crop
- Prev/next circular buttons, a thumbnail strip (active thumb outlined
  `#F47F4B`, others at `.45` opacity), and an `n / total` counter
- Keyboard: `Escape` closes, `←`/`→` step. `body` scroll locked while open.

Two implementation constraints, both learned the hard way:

1. **Visibility is CSS-driven**, not template-driven: the overlay ships
   `display: none` and is shown by `body[data-lb] #lightbox { display: flex }`.
   A conditional render flashed unresolved template placeholders during load.
2. The counter is written with `textContent`, not a template hole, for the same
   reason.

Image sources are resolved **live from the clicked cell** and matched by
identity, not by array position — position and DOM order can diverge while
images are still loading, which opened the wrong photo.

---

## 7. Care section (`#care`)

Six tiers, each an icon in `#F47F4B`, a heading in `#071B2B`, and a short
paragraph: **Delivery · Setup · Upkeep · Maintenance · Technical support ·
Lifetime care**. Three per row on desktop, collapsing responsively.

---

## 8. Contact (`#contact`)

- Blue ground with a **wave SVG divider** on its top edge: a single
  `<path>` in `#FDF9F5`, `preserveAspectRatio="none"`, height
  `clamp(64px,7.2vw,132px)`, drifting horizontally on a seamless loop. The path
  must tile exactly at its wrap point or a seam appears.
- Headline "YOUR JOURNEY STARTS HERE", an **Email us** pill CTA and the raw
  address, both `mailto:info@duomarine.mt`
- A full-bleed parallax lifestyle image sits above the FAQ section

---

## 9. Header

- `position: fixed`, `z-index: 60`, translucent with `backdrop-filter`
- **Desktop:** full logo left, nav links centre/right, Enquire pill right
- **Mobile (<900px):** wordmark only on the left (rendered small), burger icon
  on the **right**, in brand dark blue. The Enquire CTA is removed from the bar.
  Menu opens as a full overlay.
- All nav links hover to `#F47F4B`

---

## 10. Responsive

Single breakpoint that matters: **900px** (`narrow` state in JS, from
`window.innerWidth < 900` on a resize listener).

- Gallery columns step at 1100px and 660px (§5.4)
- Hero art swaps source at 900px via `<picture>`
- Section h2s stay larger than product h3s on mobile — the hierarchy must not invert
- Hero headline scales up on mobile rather than down
- Minimum touch target 44px throughout

---

## 11. Motion inventory

| Name | Duration / easing | Applies to |
|---|---|---|
| `bootMark` / `bootMarkOut` | .78s / .32s | Boot screen mark |
| `bootOut` | .58s cubic-bezier(.6,0,.22,1) | Boot cover lift |
| `panelInA` / `panelInB` | 1.6s cubic-bezier(.22,.62,.24,1) | Product panel cross-fade |
| `rise` / `riseSm` | 1.0–1.2s cubic-bezier(.2,.8,.2,1) | Scroll reveals (view timeline) |
| `heroPar` | scroll-linked | Hero parallax |
| `slotWave` / `slotSweep` | 2.1s / 1.9s loop | Gallery loading skin |
| grid-rows transition | .5s | Accordions |
| Hover transitions | .3–.4s ease | Veils, buttons, links |

Scroll reveals use CSS `animation-timeline: view()` where supported. Parallax
layers carry `data-parallax="<factor>"` and have a `requestAnimationFrame`
fallback. `prefers-reduced-motion` disables the looping animations.

---

## 12. Rebuild checklist

- [ ] Two background colours only; square corners; no shadows
- [ ] Every hover state is `#F47F4B`
- [ ] Accordions closed by default, with visible entry counts
- [ ] Per-section tab state (jet ski clicks don't animate RIBs)
- [ ] Gallery rows always complete (6/3/2)
- [ ] Lightbox shows uncropped frames and opens the clicked photo
- [ ] Boot screen: blue first frame, waits for decode, one composited move
- [ ] Enquire CTAs scroll to `#contact`; email CTAs are `mailto:`
- [ ] Barlow 500 on all product CTAs
