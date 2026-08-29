# Verify with measurements

Screenshots show that something is off. Numbers show what, by how much,
and whether the fix is exact. Measure first, screenshot once.

## Formats (minimum set)

| Viewport | Pointer | Level expected |
|---|---|---|
| 1440×900 | mouse | scene (must be strictly unchanged) |
| 1280×720 | mouse | expansion-only + flat board |
| 1024×680 | mouse | expansion-only + flat, navbar with burger |
| 1133×744 | touch (iPad mini landscape) | flat, no transition chrome |
| 744×1133 | touch, mobile | mobile |
| 390×844 | touch, mobile | mobile |

Add the client's real devices when known. Scroll **both directions**
for anything that toggles (navbar colour, sticky sources).

## Tooling

### chrome-devtools MCP (preferred)

Per format: `emulate` viewport (`WxHxDPR[,mobile][,touch][,landscape]`)
→ `navigate_page` (reload) → `evaluate_script` for numbers →
`take_screenshot` once. The emulated page has no window chrome, so
short heights are reachable.

### claude-in-chrome (real browser tab)

`resize_window` + `javascript_tool`. Limit: a real Chrome window on a
13" laptop does not go below ~680 px of inner height and cannot emulate
touch. Use it for the scene level, use emulation for the rest.

### By hand

DevTools device toolbar with a custom size; run the snippets below in
the console.

## Snippets

Paste into `evaluate_script` / the console. Adapt selectors.

### Rest state of a pinned entrance (panel vs label vs photo)

```js
(async () => {
  await new Promise(r => setTimeout(r, 2500));       // fonts, first refresh
  scrollTo(0, 0); await new Promise(r => setTimeout(r, 800));
  const label = document.querySelector('a[href="#services"]');
  const panel = label.previousElementSibling;          // the scaled block
  const text  = label.querySelector('span span');
  const media = document.querySelector('[data-hero-media]');
  const l = label.getBoundingClientRect(), p = panel.getBoundingClientRect(),
        s = text.getBoundingClientRect(),  m = media.getBoundingClientRect();
  const atRest = {
    panelH: +p.height.toFixed(1), labelH: +l.height.toFixed(1),
    panelRightMinusLabelRight: +(p.right - l.right).toFixed(2),
    panelRightMinusPhotoLeft:  +(p.right - m.left).toFixed(2),
    textCentreOffset: +((s.top + s.height/2) - (p.top + p.height/2)).toFixed(1),
  };
  const during = [];
  for (const y of [150, 400, 650, 790]) {
    scrollTo(0, y); await new Promise(r => setTimeout(r, 900));
    const pp = panel.getBoundingClientRect(), mm = media.getBoundingClientRect();
    during.push({ y, overlap: +(pp.right - mm.left).toFixed(2) });
  }
  return { atRest, during };
})();
```

Pass: `panelH === labelH`, both "minus" values `0`, `textCentreOffset 0`,
`overlap` constant and equal to the designed value (for example `1`).

### Navbar height and pin start

```js
({ navbar: document.querySelector('.site-navbar').getBoundingClientRect().height,
   stageTop: document.querySelector('[data-stage]').getBoundingClientRect().top })
```

Pass: equal at scroll 0. A gap means the pin start uses a wrong navbar
value (token read as `NaN`).

### Navbar colour in both directions

```js
(async () => {
  const out = [];
  for (const y of [0, 300, 600, 830, 1500, 2300, 1500, 600, 0]) {
    scrollTo(0, y); await new Promise(r => setTimeout(r, 900));
    out.push({ y, on: document.documentElement.hasAttribute('data-nav-on-blue') });
  }
  return out;
})();
```

Pass: off at top, on from the entrance threshold, off again when the
next non-coloured section arrives, symmetric on the way back.

### Stacked panels: media at the same height, board height stable

```js
(async () => {
  const panels = [...document.querySelectorAll('[data-service-panel]')];
  const mediaOffsets = panels.map(p =>
    Math.round(p.querySelector('[data-service-media]').getBoundingClientRect().top - p.getBoundingClientRect().top));
  const board = document.querySelector('.board');
  const heights = [];
  for (const t of document.querySelectorAll('[role=tab]')) {
    t.click(); await new Promise(r => setTimeout(r, 250));
    heights.push(Math.round(board.getBoundingClientRect().height));
  }
  return { mediaOffsets, heights };
})();
```

Pass: all `mediaOffsets` equal; all `heights` equal.

### Cards / figures aligned across a row

```js
({ h3: [...document.querySelectorAll('#method li h3')].map(e => Math.round(e.getBoundingClientRect().top)),
   p:  [...document.querySelectorAll('#method li p')].map(e => Math.round(e.getBoundingClientRect().top)),
   hairlines: [...document.querySelectorAll('[data-about-hairline]')].slice(0,3).map(e => Math.round(e.getBoundingClientRect().top)) })
```

Pass: each array holds identical values.

### Photo in a fixed column

```js
(() => { const img = document.querySelector('[data-about-photo]'); const col = img.parentElement; const text = col.nextElementSibling;
  const i = img.getBoundingClientRect(), c = col.getBoundingClientRect(), t = text.getBoundingClientRect();
  return { photoW: Math.round(i.width), colW: Math.round(c.width), overlapsText: i.right > t.left, bottomsAligned: Math.abs(i.bottom - t.bottom) < 1 }; })()
```

### Navbar link fit

```js
(() => { const r = s => document.querySelector(s).getBoundingClientRect();
  const b = r('.site-navbar [data-brand]'), n = r('.site-navbar nav > div'), c = r('.site-navbar .navbar-cta-primary');
  return { fits: n.left >= b.right + 16 && n.right <= c.left - 16 }; })()
```

## Pitfalls

- **Device mode emulates touch** → `hover: none` → no entrance
  animation. That is the tablet criterion working, not a bug. Emulate
  without `touch` to test the mouse levels at short heights.
- **Fonts**: measure after `document.fonts.ready`; a 2.5 s wait after
  load is a cheap proxy.
- **Smooth-scroll libraries** (Lenis) intercept `scrollTo` mid-animation;
  wait ~900 ms after each programmatic scroll, or scroll once and
  measure.
- **Hydration warnings** on `<html>` may predate your work: `git stash`,
  reload, check, `git stash pop`.
- **Bounded rounds**: one full round across formats, fix everything it
  shows, one confirmation round, stop. Open-ended screenshot loops cost
  money and find less than the numbers above.
- **Forced dark mode is invisible on iOS.** Chrome Android and Samsung
  Internet repaint light-only sites; Safari never does. Test on an
  Android phone in dark mode, or emulate `prefers-color-scheme: dark`
  in Chrome's Rendering panel *after* removing `color-scheme: only
  light` to see what the client saw.
- **Anchor landing**: after a same-page hash jump,
  `target.getBoundingClientRect().top` must be ≥ the navbar height.
  Below it, the navbar hides the heading.

