# responsive-motion

**A Claude Code skill for scroll-animated sites that must work on every device.**

Your landing page has GSAP pins, a sticky services board, a method column
whose photo changes as you scroll, an expansion transition from the hero.
It looks great on the MacBook it was built on. Then the client opens it
on a 1366×768 laptop, an iPad in landscape, or with DevTools docked, and
the photo is crushed, the sticky column has no room, a blue block
overlaps the photo by four pixels, the navbar turns blue at the wrong
moment. You add a media query for that device. The next device breaks.

This skill replaces that loop with one decision, made once.

## What it does

- **Decides where a scene exists** with a single criterion: viewport
  width, viewport height, real pointer. The same media query lives in one
  JavaScript constant and one CSS variant, so JS and CSS never disagree.
- **Keeps the animation where it fits.** Nothing changes on the designer's
  screen. On a short laptop with a mouse the entrance transition still
  plays; only what cannot fit stops sticking.
- **Falls back to a layout that already exists** (the mobile or
  reduced-motion layout), never to a third "tablet" design.
- **Separates the machine lever** (reduced-motion, Save-Data, weak
  hardware) from the format lever, so a powerful short laptop and a weak
  tall desktop each get the right thing.
- **Verifies with numbers.** Six viewports, measured with DevTools
  scripts to 0 px: panel edge equals label edge equals photo edge, board
  height stable across tabs, cards aligned across a row, navbar colour
  correct in both scroll directions.

## The model

| Level | Condition | What plays |
|---|---|---|
| **scene** | width ≥ `lg` and height ≥ threshold and `(hover: hover)` | everything: pin, sticky viewport, sticky column, image swaps, masks |
| **expansion-only** | `lg` and mouse, height < threshold | the entrance transition plays identically; what follows scrolls instead of sticking |
| **flat** | `lg` but short viewport or touch | the desktop static layout: clickable board, bento cards, one image per step |
| **mobile** | width < `lg` | mobile layout, no scene |
| **lite** | reduced-motion, Save-Data, ≤ 2 GB RAM, ≤ 2 cores | no smooth scroll, tweens or masks, on any format |

The height threshold comes from the tallest scene and is rounded to a
breakpoint the design system already uses. One constant, one place.

## Install

As a Claude Code plugin (this repository is its own marketplace):

```
/plugin marketplace add nualt/responsive-motion
/plugin install responsive-motion@responsive-motion
```

As a plain skill, for Claude Code or any agent that reads `SKILL.md`
folders:

```bash
git clone https://github.com/nualt/responsive-motion ~/.claude/skills/_responsive-motion
ln -s ~/.claude/skills/_responsive-motion/skills/responsive-motion ~/.claude/skills/responsive-motion
```

Then invoke `/responsive-motion`, or describe the symptom ("the pinned
photo is crushed on 1366×768") and let it trigger.

## How a session goes

1. **Diagnose.** The skill lists the scenes, every activation criterion in
   JS and CSS, and every magic value (`100dvh − 26rem`, per-format caps,
   CSS tokens read from JavaScript). It reproduces on six viewports with
   measurements before proposing anything.
2. **Announce.** It names the sections and files it will touch, one
   section at a time, and waits for agreement. It never rebuilds the
   page's activation mechanism to fix one photo.
3. **Implement in order.** Constants and variants first (no visible
   change), then the static CSS blocks, then the `scene:` / `flat:`
   swaps, then real-box measurement, then alignment, then the navbar.
4. **Verify in bounded rounds.** One full round across formats, fix
   everything it shows, one confirmation round, stop.

## What is inside

```
skills/responsive-motion/
  SKILL.md                    trigger phrases, task patterns, procedure, non-negotiables
  reference/principles.md     the seven rules, the five levels, expected result per device
  reference/recipes.md        thirteen recipes with code: single criterion, matchMedia helpers,
                              static CSS in two parts, visibility vs display, subgrid alignment,
                              photo in a fixed column, reserved space, real-box calibration,
                              trigger priority, navbar colour sources, navbar links, fluid values
  reference/verify.md         formats, tooling (chrome-devtools MCP, claude-in-chrome, by hand),
                              copy-paste measurement snippets, emulation pitfalls
  reference/anti-patterns.md  what not to do, with the failure that taught each rule
.claude-plugin/               plugin manifest and single-plugin marketplace
```

## Stack

Examples use GSAP 3 + ScrollTrigger and Tailwind v4 in a Next.js app.
Every recipe is a layout idea first; the syntax is replaceable. The
verification snippets are plain browser JavaScript.

## Origin

Extracted from a production site for a French executive-coaching firm,
after a week of per-device fixes that kept breaking the next device. The
anti-patterns file keeps the failures that paid for each rule.

## Contributing

Issues and pull requests welcome, especially: recipes for other animation
libraries (Framer Motion, Motion One, native scroll-driven animations),
measurement snippets for other frameworks, and real-device results that
contradict the expected-result table.

## License

MIT © nualt
