# Recipes

Code targets GSAP 3 + ScrollTrigger and Tailwind v4, but every recipe is
a layout idea first. Replace the syntax, keep the idea.

## 1. One criterion, two consumers

```ts
// motion-mode.ts — single source of truth
export const DESKTOP_MIN_WIDTH_PX = 1024;
export const SCENE_MIN_HEIGHT_PX = 800;

const SCENE_FITS =
  `(min-width: ${DESKTOP_MIN_WIDTH_PX}px) and (min-height: ${SCENE_MIN_HEIGHT_PX}px) and (hover: hover)`;

/** Appear animations: width only. */
export const DESKTOP_MOTION_MQ =
  `(min-width: ${DESKTOP_MIN_WIDTH_PX}px) and (prefers-reduced-motion: no-preference)`;
/** Entrance transition (short pin): mouse + motion, no height condition. */
export const EXPANSION_MQ =
  `(min-width: ${DESKTOP_MIN_WIDTH_PX}px) and (hover: hover) and (prefers-reduced-motion: no-preference)`;
/** Scenes that must fit: sticky viewport, sticky column. */
export const CHOREOGRAPHY_MQ = `${SCENE_FITS} and (prefers-reduced-motion: no-preference)`;
/** Scroll-bound content (an image swap that is the content): fits, even in lite. */
export const SCENE_MQ = SCENE_FITS;
```

```css
/* globals.css — Tailwind v4, same threshold, same condition */
@custom-variant scene {
  @media (width >= 1024px) and (height >= 800px) and (hover: hover) { @slot; }
}
@custom-variant flat {
  @media (width >= 1024px) and ((height < 800px) or (hover: none)) { @slot; }
}
```

Usage: `scene:sticky scene:top-[…]`, `flat:grid flat:grid-cols-3`. Keep
`lg:` for what is purely about width.

## 2. gsap.matchMedia helpers

```ts
const isLite = () => document.documentElement.dataset.motion === "lite";

function addDesktopMotion(media, setup) { media.add(DESKTOP_MOTION_MQ, () => isLite() ? undefined : setup()); }
function addExpansion(media, setup)     { media.add(EXPANSION_MQ,     () => isLite() ? undefined : setup()); }
function addChoreography(media, setup)  { media.add(CHOREOGRAPHY_MQ,  () => isLite() ? undefined : setup()); }
function addSceneLayout(media, setup)   { media.add(SCENE_MQ, setup); } // runs in lite too
```

`gsap.matchMedia` re-runs and reverts on media-query change: resizing a
window moves cleanly between levels.

## 3. Machine lever before first paint

A synchronous inline script in `<head>` sets `html[data-motion="lite|full"]`
from `prefers-reduced-motion`, `navigator.connection.saveData`,
`navigator.deviceMemory ≤ 2`, `navigator.hardwareConcurrency ≤ 2`. CSS
and JS read the attribute. Re-evaluate on `prefers-reduced-motion` change.

## 4. Static CSS block, in two parts

A choreographed section carries "waiting for the animation" styles:
`opacity: 0`, a sticky viewport, a tall `min-height` scroll budget, a
negative margin overlapping the previous section. One block must cancel
them when the scene does not exist, and it must distinguish two cases:

```css
/* No entrance animation (touch, reduced-motion, lite):
   transition chrome hidden, section placed after the hero, headings visible. */
@media (width >= 1024px) and (prefers-reduced-motion: reduce),
       (width >= 1024px) and (hover: none) {
  .choreo-chrome { display: none !important; }
  .showcase { margin-top: 0 !important; background: var(--brand); pointer-events: auto; }
  .showcase [data-heading] { opacity: 1 !important; visibility: visible !important; }
}

/* Static board (reduce, short viewport, touch): no sticky viewport,
   panels told apart by aria-hidden, media back to its design cap. */
@media (width >= 1024px) and (prefers-reduced-motion: reduce),
       (width >= 1024px) and (height < 800px),
       (width >= 1024px) and (hover: none) {
  :root { --media-max-height: 448px; }          /* no dvh formula */
  .showcase__scroll { min-height: auto; }
  .showcase__viewport { position: relative; height: auto; overflow: visible; }
  .showcase [data-panel][aria-hidden="true"] { visibility: hidden; } /* not display:none */
}
```

The trap: forcing "visible / brand background / margin 0" in the short
viewport case **while the entrance animation still plays** makes the
section slide over the hero. Hence two blocks.

## 5. Stacked panels: `visibility`, not `display: none`

Panels stacked in the same grid cell (`col-start-1 row-start-1`) and
hidden with `visibility: hidden` keep their height, so the board does
not jump between tabs. `display: none` changes the height on every
click.

## 6. Subgrid for everything that must line up

- Panels (text, media): wrapper `grid-rows-[auto_auto]`, each panel
  `grid row-span-2 grid-rows-subgrid`. The media sits at the same height
  in every panel; the longest text sets the row.
- Cards (image, number, title, text): `row-span-4 grid-rows-subgrid`. A
  two-line title no longer shifts its neighbours. A wrapper around
  title + text becomes `display: contents` so both reach the grid.
- Figures (number, caption, hairline): `row-span-3`. Hairlines stay on
  one line when a caption wraps.

Trap: inside a grid, an `aspect-ratio` element capped with `max-height`
transfers the cap to its width and shrinks; in flex it kept full width.
Set `w-full` explicitly so the width is definite and the height crops
(`object-cover`).

## 7. Photo in a fixed column

Absolute photo, height = neighbour block, width auto: too small when the
neighbour is short, wider than its column when the neighbour is long.
Rule: width = column, height = neighbour, `object-cover` with
`object-position` on the area to protect (faces: `top`). Never
distorted, never overflowing; at worst the edges are cropped.

## 8. Reserve space instead of overlapping

An anchored element (badge, note) in `absolute` reserves nothing: the
centred content ends up under it on short screens. Reserve the same
height at the top and bottom of the column
(`py-[calc(var(--badge-h)+gap)]`): the centre does not move, nothing
overlaps, the column grows if space runs out. Fluid size:
`clamp(64px, 12dvh, 112px)`, width `auto`, `self-start` (otherwise a
flex column stretches the image).

## 9. Pinned scene geometry: measure the real boxes

A panel that must coincide with a label at rest:

```ts
const measure = () => {
  const stageBox = stage.getBoundingClientRect();
  const labelBox = label.getBoundingClientRect();     // sub-pixel
  baseScaleX = labelBox.width  / stage.offsetWidth;
  baseScaleY = labelBox.height / stage.offsetHeight;
  // the photo may already be pushed when a refresh happens: remove its current translate
  mediaLeft = media.getBoundingClientRect().left - stageBox.left
            - Number(gsap.getProperty(media, "x"));
};
// pushX = scaleX * stageWidth - mediaLeft   (panel edge − photo edge)
```

- `onRefreshInit: measure` + `invalidateOnRefresh: true` + function
  values in the `fromTo`.
- **Refresh after fonts**: `document.fonts.ready.then(() =>
  ScrollTrigger.refresh())`. On short screens the hero is content-driven
  (not `min-h`): its height changes when the web font arrives, and
  ScrollTrigger does not know.
- **Navbar height**: `header.getBoundingClientRect().height`, not the
  token (rule 5). A 68 px fallback when the navbar is 63.5 px starts the
  pin 4.5 px early → at rest the animation is already at 0.6 % → panel
  4 px too tall, text off-centre, overflow onto the photo.

## 10. Trigger order and pins

A ScrollTrigger created **before** a pin located above it (child React
effect runs before the parent's) computes its position without the
pin's spacer, so it fires at the top of the page. `refreshPriority: -1`
on the child trigger makes it recompute after the pin.

## 11. Navbar colour: sources, not a boolean

Each coloured zone announces itself under an id
(`setNavColourSource(id, bool)`); the navbar is coloured while at least
one source is active. A global boolean gets overwritten when two zones
are adjacent. When a scene is cut, decide **who switches off** the
source set by the entrance animation (typically the section's own static
trigger, `onLeave`).

## 12. Navbar links do not fit at 1024

Six centred links (~510 px) plus brand plus two CTAs do not fit between
1024 and ~1220 px. Links from `xl` (1280); between `lg` and `xl`, brand +
CTA + burger. Measure: `nav.left ≥ brand.right + 16` and `nav.right ≤
cta.left − 16`.

## 13. Fluid, not per-format

Every value that "depends on the format" should be a `clamp()` on a
viewport unit, a real measurement, or a flex/grid consequence. If you
are about to write `@media (width >= 1024px) and (width < 1280px) and
(orientation: landscape)`, stop: that is a device, not a criterion.

## 14. Phone "dark mode" the site never declared

Chrome for Android ("Auto dark theme") and Samsung Internet repaint any
site that declares no colour scheme when the phone is in dark mode:
inverted background, shifted brand blues, and a logo that the navbar
turns white (`filter: brightness(0) invert(1)`) comes out black. iOS
Safari never does this, so an iPhone in dark mode reproduces nothing.
A light-only site opts out once:

```html
<meta name="color-scheme" content="only light">
```

```css
:root { color-scheme: only light; }
```

Both: the meta avoids a flash of darkened content, the CSS covers
elements rendered before the meta is parsed. Symptom to recognise:
"the colours on my phone are not the ones on the PC", "the logo turns
black", "put the white background back" — with no dark theme in the
code.

## 15. Native anchors land under the sticky navbar

A same-page `<a href="/#services">` (mobile menu on the home page) is a
native hash jump: the router's pathname does not change, so a
`useEffect([pathname])` scroll handler with a navbar offset never runs.
The section's top lands at viewport top, and the sticky navbar hides its
first 60–70 px — "the photo and the text are offset on this page only".
One line for every anchor, native or scripted:

```css
:root { scroll-padding-top: var(--navbar-height); }
```

Keep the scripted handler for cross-page navigation; the CSS covers the
native path and costs nothing where a script already offsets.

