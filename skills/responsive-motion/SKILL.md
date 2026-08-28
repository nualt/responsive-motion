---
name: responsive-motion
description: Make a scroll-animated site (GSAP pins, sticky viewports, sticky columns with image swaps, expansion transitions) degrade cleanly on every device without per-device hacks. Use when a site "works on my Mac" but breaks on tablets, short laptops (1366×768, 1280×720), windows with devtools open, weak machines, or prefers-reduced-motion; when a pinned photo is crushed; when a sticky element sticks with no room or refuses to stick; when an animated block overlaps its neighbour by a few pixels on some formats; when the navbar changes colour at the wrong moment; when whole sections stay invisible (opacity 0 waiting for an animation that never runs); or when the codebase is accumulating "iPad landscape" media queries. Triggers - responsive animation, scroll choreography on tablet, GSAP ScrollTrigger pin on small screen, sticky not fitting viewport, degrade animations, reduced motion fallback, crushed image in pinned section, media query per device.
---

# Responsive motion

Scroll choreographies need room. This skill decides **once** where they
exist, guarantees a clean static layout everywhere else, and verifies the
result with measurements instead of screenshots.

The goal is not to remove animations. It is to stop patching formats one
by one.

## Task patterns this skill solves

- "The pinned section crushes the photo on 1366×768."
- "Sticky column taller than the viewport, it never sticks / sticks and cuts."
- "iPad landscape shows the desktop choreography and it jumps."
- "The blue block / label / badge overlaps the photo by 4 px on some screens."
- "Navbar turns blue at the top of the page on laptops."
- "We keep adding `@media (1024px–1279px) and (orientation: landscape)` rules."
- "Weak machines and reduced-motion users must still see all content."

## Sub-commands

`/responsive-motion` with no argument runs the full procedure below.
With an argument, run only that phase:

| Command | Does |
|---|---|
| `/responsive-motion diagnose` | Phase 1 only. Lists scenes, criteria, magic values, reproduces on the six formats with numbers, and proposes the file list. Writes nothing. |
| `/responsive-motion implement <section>` | Phase 2 on one named section (for example `implement services-board`). Refuses more than one section per call. |
| `/responsive-motion verify` | Phase 3 only: one batched round across the six formats with the measurement snippets from `reference/verify.md`, then a findings list. Writes nothing. |

## Read first

1. [reference/principles.md](reference/principles.md) - the seven rules and the decision model (levels: scene, expansion-only, flat, mobile, lite). Read before touching code.
2. [reference/recipes.md](reference/recipes.md) - thirteen concrete recipes with code (GSAP + Tailwind v4, readable without them).
3. [reference/verify.md](reference/verify.md) - how to measure, which formats, which numbers must be 0.
4. [reference/anti-patterns.md](reference/anti-patterns.md) - what not to do, with the failures that taught each rule.

## Procedure

### 1. Diagnose before editing

- List the scenes: what is pinned, what is sticky, what drives the navbar colour, which elements wait for an animation (`opacity: 0`).
- List every activation criterion in JS (`gsap.matchMedia`, `window.matchMedia`) and CSS (media queries, data attributes). Note where they diverge.
- List magic values: `100dvh − X rem`, per-format caps, JS fallbacks for CSS tokens read with `getPropertyValue`.
- Reproduce on the six formats in `verify.md` with numbers, not just screenshots.
- Name the sections you will touch, file by file. Announce the list before writing. One named section at a time.

### 2. Implement, in this order

1. **One criterion, two consumers.** A JS constant (`SCENE_MQ`) and a CSS custom variant carry the same media query: `lg` width **and** minimum height **and** `(hover: hover)`. Its complement inside `lg` is `flat`. The machine lever (`html[data-motion="lite"]`) is separate and set before first paint. Nothing renders differently yet.
2. **Static CSS block in two parts** for each choreographed section: "no entrance animation" (touch, reduced-motion, lite) versus "no sticky" (short viewport). Never force visible/opaque what the entrance animation still controls.
3. **Swap `lg:` for `scene:` / `flat:`** on scene elements only: sticky, top offsets, list paddings, per-step images, transition chrome. Pure-width layout keeps `lg:`.
4. **Measure real boxes**: navbar height from the element, rest state of a pinned panel from its label's rect, photo edge minus its current transform. Refresh after fonts. `refreshPriority` on triggers created before a pin.
5. **Alignment**: stacked panels hidden with `visibility`, subgrid for anything that must line up, `w-full` on aspect-ratio media inside grids, `object-cover` for photos in fixed columns, symmetric reserved space for anchored elements.
6. **Navbar**: links from the width where they actually fit, measured.
7. **Design-system doc** after visual validation, never before.

### 3. Verify in bounded rounds

One batched round across the six formats, fix everything it shows, one confirmation round, stop. See `verify.md` for the exact measurements and the emulation pitfalls.

## Decision model (summary)

| Level | Condition | What plays |
|---|---|---|
| scene | width ≥ lg **and** height ≥ threshold **and** `hover: hover` | everything: pin, sticky viewport, sticky column, swaps, masks |
| expansion-only | lg + mouse, height < threshold | the short entrance pin plays; what follows does not stick |
| flat | lg but (height < threshold **or** touch) | the desktop static layout (clickable board, bento, image per step) |
| mobile | < lg | mobile layout, no scenes |
| lite (machine) | reduced-motion, Save-Data, ≤ 2 GB, ≤ 2 cores | no Lenis, no tweens, no masks, no entrance; independent of format |

Pick the height threshold from the tallest scene and align it with a
breakpoint the design system already uses. Do not invent a "tablet"
layout: the fallback is the layout that already exists.

## Non-negotiables

- Never read geometry from a CSS token: `getPropertyValue('--x')` returns `calc(...)` as a string, `parseFloat` gives `NaN`, and the fallback hides the bug on the developer's screen.
- Never `display: none` on stacked panels that must keep a stable height.
- Never a per-device media query.
- Never rewrite the global activation mechanism to fix one section. If a format needs its own rule, the criterion is wrong, not the format.
- Never patch many files with a script. Nominal edits, one file at a time, reversible.
