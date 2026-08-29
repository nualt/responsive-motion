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
  mode is a browser feature, not a device setting — declare
  `color-scheme: only light` (fixes Chrome Android; Samsung Internet
  ignores it by default, document that as a known limit) and ask which
  phone.
- **Trust the router's scroll handler for same-page anchors.** It runs
  on pathname change; a hash jump on the same page is native and lands
  under the sticky navbar. Rule: `scroll-padding-top` on `:root`.
- **Test devices instead of the worst case of each mode.** Six formats
  were verified and none was the client's laptop (1366×768 at 125 % =
  1093×614, or Full HD at 150 % minus the browser bar = 1280×630). Rule:
  the real viewport is screen ÷ OS scale − browser chrome; verify the
  shortest viewport where flat applies and the shortest where each scene
  applies, and let fluid values cover the rest.
- **Size a photo with a rem budget.** `100dvh − 17rem` was exact until
  the intro wrapped one more line at a narrower width (Safari zoomed
  twice): photo cut. Rule: definite-height column, photo `flex-1
  min-h-0`, capped by a max-height.
- **Lower a shared threshold to rescue one scene.** The sticky column
  fit at 600 px, the pinned board did not fit under 700; one number
  cannot serve both. Rule: one threshold per scene, mirrored JS/CSS.
- **Fix the photo when the column is the problem.** At 1280 the board
  overflowed because the tab labels wrapped, not because the photo was
  too tall. Rule: measure which column is tallest before touching
  anything; widen the column first.
- **Write a CSS hook against a utility class.** `.grid` matched nothing
  because the div had `lg:grid`; the fix "applied" for hours. Rule: a
  dedicated class for every hook, then check the compiled CSS.
- **Change a grid's column count without resetting spans.** A
  `col-span-2` meant for three columns jumps into an implicit column
  when the grid has two: the structure looks destroyed. Rule: every
  column-count rule resets its children's `grid-column`.
- **Let an animation's rest state style the first paint.** A panel
  born at a hard-coded scale and recalibrated by GSAP jumps one frame at
  load. Rule: CSS styles what is visible at load; animated elements mount
  invisible and appear once measured.
- **Do more than the sentence asked.** "Commit and push" got an extra
  improvement; "enlarge the photo" touched two sections; "no browser"
  got one more measurement. Each was rolled back. Rule: quote the request
  in one line, do exactly that, stop.

## Do

- Decide the levels once, write them in the design-system doc after
  visual validation, and point every future fix at that table.
- Keep the fallback identical to an existing layout.
- Measure at 0 px. "Looks fine" is not a pass.
- Keep the client's animated identity intact where it fits. The skill
  exists to protect the animation, not to remove it.
