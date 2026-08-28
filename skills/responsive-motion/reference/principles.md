# Principles and decision model

## The seven rules

1. **A pinned scene only makes sense if it fits the viewport.** A pin
   whose content exceeds the screen is broken by construction. This is
   not an optimisation; it is a guard rail.
2. **Two independent levers, never merged.**
   - *Format*: width, height, pointer type. Decides whether the scene
     exists.
   - *Machine*: `prefers-reduced-motion`, Save-Data, RAM, cores. Decides
     whether we animate (smooth scroll, tweens, masks).
   A 956 px tall laptop is in scene mode; a 1366×768 laptop is not,
   whatever its GPU.
3. **One criterion, mirrored exactly between JS and CSS.** The same media
   query in a JS constant (`gsap.matchMedia`) and in a CSS variant. Two
   criteria that drift produce two inconsistent states.
4. **The fallback must already exist.** The static layout is the mobile
   or reduced-motion layout, not a third design. Reuse, do not invent a
   "tablet" mode.
5. **Measure the real DOM, never tokens.** `getComputedStyle().
   getPropertyValue('--x')` returns `calc(...)` verbatim for an
   unregistered custom property. `parseFloat` gives `NaN`, the fallback
   value silently wins, and the bug is invisible on the screen where the
   fallback happens to be right. Read navbar height, label boxes, photo
   edges on the elements.
6. **No per-device cases.** If a format needs its own rule, the criterion
   is wrong, not the format.
7. **One change = one named section.** Never rebuild the whole page's
   activation mechanism to fix a photo. List files before writing.

## Levels

| Level | Condition | What plays |
|---|---|---|
| **scene** | `lg` (≥ 1024) **and** height ≥ threshold **and** `(hover: hover)` | pin, sticky viewport, sticky column, image swaps, reveal masks |
| **expansion-only** | `lg` and mouse, but height < threshold | the entrance animation (short pin) plays identically; what follows does not stick |
| **flat** | `lg` but (height < threshold **or** touch) | desktop static layout: clickable board, bento cards, image per step |
| **mobile** | < `lg` | mobile layout, no scene |
| **lite** (machine) | reduced-motion, Save-Data, ≤ 2 GB, ≤ 2 cores | cuts smooth scroll, tweens, masks, entrance. Independent of format. Content tied to scroll (an image swap that *is* the content) may stay, without its mask |

### Choosing the height threshold

Take the tallest scene (for example a board with four tabs plus a call
to action inside `100dvh − navbar`) and compute the height under which
it no longer fits with the design's own compaction applied. Round to a
breakpoint the design system already uses (for example the typography
compaction under 800 px). One constant, one place.

### Why `(hover: hover)`

Touch tablets in landscape pass the width test and often the height
test, yet pins are fragile there: the URL bar changes `dvh`, there is no
real hover, and there is rarely room. `(hover: hover)` excludes them. An
iPad with a trackpad reports hover and gets the scene; that is intended.

### What does not depend on the format

Appear animations (fade / slide on entry) stay on `lg` + motion. There
is no reason to cut them on a short laptop. Only scenes that **occupy**
the viewport (pin, sticky) are subject to the height criterion.

## Expected result per device (test all at every delivery)

| Device | Expected |
|---|---|
| Desktop ≥ threshold, mouse (MacBook Air, 1440×900, 1920×1080) | full scene, strictly unchanged |
| Laptop 1280×720 / 1366×768, mouse | entrance identical, then the board scrolls instead of sticking; cards in bento |
| iPad landscape (1024×768, 1133×744, 1180×820) | flat: static board, bento, no transition chrome |
| iPad portrait, phones | mobile layout |
| Lite machine (any format) | no animation; content-bearing swaps remain |
| `prefers-reduced-motion` | as lite |

## Vocabulary

- **Scene**: a section that occupies the viewport while scroll drives it (pin, sticky viewport, sticky column).
- **Entrance / expansion**: a short pinned transition between two sections (for example a block growing from a label to full screen).
- **Chrome**: the decorative elements that only exist for a transition (panel, trail, label). Hidden whenever the transition does not run.
- **Static block**: the CSS that cancels "waiting for animation" styles when the scene does not exist.
- **Flat**: the desktop static layout, complement of scene inside `lg`.
- **Lite**: the machine lever, `html[data-motion="lite"]`.
