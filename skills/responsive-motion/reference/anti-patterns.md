# Anti-patterns, and the failures behind each rule

Every rule in this skill was paid for. The failures below are real,
anonymised, and kept so the rule reads as a consequence rather than a
preference.

## Do not

- **Rebuild the global activation mechanism to fix a local symptom.**
  Failure: a request about crushed photos turned into a new `html`
  attribute plus Tailwind variants across eight files, applied by a
  script. The CSS half was rejected; every animated section broke on
  every device at once. Rule: one named section, nominal edits, list the
  files first.
- **Patch many files with a script.** Same failure. Reversal took an
  hour of hand edits. Rule: editor tool, one file at a time.
- **Write a single static block for "short viewport".** Failure: the
  block forced the next section visible, opaque and un-overlapped while
  the entrance animation still ran; the section slid over the hero
  photo. Rule: two blocks, "no entrance" and "no sticky".
- **Read CSS tokens in JS.** Failure: `getPropertyValue('--navbar-height')`
  returned `calc(...)`, `parseFloat` gave `NaN`, the 68 px fallback won
  everywhere. Invisible at 1440×900 (navbar really 68), 4.5 px wrong
  under 800 px: pin started early, block 4 px too tall, label text
  off-centre, overflow onto the photo. Rule: measure the element.
- **Calibrate a pinned scene on the first layout.** Failure: stage
  height measured before the web font arrived; content-driven hero grew
  afterwards; scale factors stayed stale. Rule: `document.fonts.ready`
  → `ScrollTrigger.refresh()`.
- **Create scroll triggers before the pin above them without a
  priority.** Failure: a child component's trigger computed its start
  without the pin spacer; the navbar went blue at scroll 0. Rule:
  `refreshPriority: -1`.
- **Use `display: none` on stacked panels.** Failure: board height
  jumped with every tab. Rule: `visibility: hidden`.
- **Put aspect-ratio media in a grid without a definite width.** Failure:
  subgrid made the photo narrower (max-height transferred to width).
  Rule: `w-full`.
- **Anchor an element with `absolute` and hope.** Failure: badge under
  the CTA on short screens; then auto margins de-centred the copy; then
  the image stretched because a flex column stretches children. Rule:
  symmetric reserved space, `self-start`.
- **Let a photo's width follow its neighbour's height.** Failure: tiny
  on wide screens, overflowing onto text on narrow ones. Rule: fixed
  column width, `object-cover`, position on the face.
- **Add a media query for a device.** `(1024px–1279px) and (orientation:
  landscape)` was the previous attempt at the same problem. It fixed one
  iPad and broke the next laptop. Rule: a criterion (height, pointer),
  never a device.
- **Trust device emulation blindly.** Device mode adds touch; the
  entrance animation is *supposed* to be absent there. Rule: know which
  level you are testing.
- **Ask the user to look, then change something else.** Every "let me
  also…" outside the named section eroded trust faster than the bug
  did. Rule: announce, then do exactly that.

- **Reproduce a phone report on another OS.** A client on Android saw
  a dark, inverted site; the designer's iPhone in dark mode showed the
  PC colours, and the report looked "impossible". Rule: forced dark
  mode is a browser feature (Chrome Android, Samsung Internet), not a
  device setting — declare `color-scheme: only light` and ask which
  phone.
- **Trust the router's scroll handler for same-page anchors.** It runs
  on pathname change; a hash jump on the same page is native and lands
  under the sticky navbar. Rule: `scroll-padding-top` on `:root`.

## Do

- Decide the levels once, write them in the design-system doc after
  visual validation, and point every future fix at that table.
- Keep the fallback identical to an existing layout.
- Measure at 0 px. "Looks fine" is not a pass.
- Keep the client's animated identity intact where it fits. The skill
  exists to protect the animation, not to remove it.
