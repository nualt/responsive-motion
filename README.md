# responsive-motion

A Claude Code skill for one recurring job: a site with scroll
choreographies (GSAP pins, sticky viewports, image swaps, expansion
transitions) that works on the designer's laptop and breaks on tablets,
short laptops, weak machines and reduced-motion users.

It does not remove the animations. It decides **once** where they exist
(width, height, pointer), guarantees a clean static layout everywhere
else, and verifies with measurements instead of screenshots.

## Install

As a Claude Code plugin (the repo is its own marketplace):

```
/plugin marketplace add nualt/responsive-motion
/plugin install responsive-motion@responsive-motion
```

Or as a plain skill, for any agent that reads `SKILL.md` folders:

```bash
git clone https://github.com/nualt/responsive-motion /tmp/responsive-motion
ln -s /tmp/responsive-motion/skills/responsive-motion ~/.claude/skills/responsive-motion
```

Claude Code picks it up from the frontmatter in `SKILL.md`. Invoke with
`/responsive-motion` or let it trigger on the symptoms listed there.

## What is inside

| File | Purpose |
|---|---|
| `skills/responsive-motion/SKILL.md` | Trigger, task patterns, procedure, decision model, non-negotiables |
| `…/reference/principles.md` | The seven rules, the five levels, the per-device expectation table |
| `…/reference/recipes.md` | Thirteen recipes with code: single criterion, matchMedia helpers, static CSS in two parts, subgrid alignment, real-box calibration, trigger priority, navbar |
| `…/reference/verify.md` | Formats, tooling (chrome-devtools MCP, claude-in-chrome, by hand), copy-paste measurement snippets, pitfalls |
| `…/reference/anti-patterns.md` | What not to do, with the failure that taught each rule |
| `.claude-plugin/` | Plugin manifest and single-plugin marketplace |

## The model in one table

| Level | Condition | What plays |
|---|---|---|
| scene | width ≥ lg and height ≥ threshold and `hover: hover` | everything |
| expansion-only | lg + mouse, short viewport | entrance transition only |
| flat | lg but short or touch | desktop static layout |
| mobile | < lg | mobile layout |
| lite | reduced-motion / Save-Data / weak machine | no animation, any format |

## Stack assumptions

Examples use GSAP 3 + ScrollTrigger and Tailwind v4. Every recipe is a
layout idea first; the syntax is replaceable.

## License

MIT
