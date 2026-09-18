# NOTES — what the design does that WebFluent could not, and what was done instead

This file is a deliverable (HALYARD_BUILD.md §1.7). Every adaptation is listed
here, with the compiler change that removed the need where one was made. The
compiler lives in the user's WebFluent repository; the changes are its own
commits (`git log` there, from `refactor(codegen): share element_tag…` on).

## §0 — the five unknowns, answered from the compiler source

| # | Question | Answer, before | What changed |
|---|---|---|---|
| 1 | Does a `Component` accept a block of children? | Parsed (`children` keyword, SPEC §6.4) but both web backends dropped the block. | **Fixed in the compiler.** The block reaches the component as a thunk (SPA) or a slot stack (SSG), compiled in the caller's scope. `AppShell(current: …) { … }` is one component. |
| 2 | Can a `style` value be data-driven? | Yes, but assigned once at element creation. | **Fixed.** A value that reads state is set inside an effect, so `style { width: "{pct}%" }` follows `pct`. Charts are styled containers. |
| 3 | Do CSS custom properties survive, and is there a global `<style>`? | `style { --x: … }` cannot parse; no head injection. But a `Theme` accepts *any* token name and emits it as `--name` on `:root`. | The whole Fluant palette lives in `src/Theme.wf` (94 tokens) and is used as `"var(--surface-raised)"`. `meta.fonts` / `meta.stylesheets` were added to the compiler for the Google Fonts link and a two-line `public/base.css` (`html { background }`, `::selection`, font smoothing) — the only things no element-level style block can express. |
| 4 | Token names beyond the four? | Any hyphenated name; a segment could not start with a digit. | **Fixed** (`viz-1`, `radius-2xl` parse). |
| 5 | Raw markup, SVG, `aria-*` / `data-*`? | No raw markup; unknown named args became attributes but names could not be hyphenated. | **Hyphenated named args added**: `Button("x", aria-pressed: on, data-tone: "success")`, reactive when the value reads state. No raw SVG: icons are the 30 built-ins; see below for every place an SVG was replaced. |

Also found while reading the compiler, and fixed there because AGENTS.md
promised them: `Icon("home")` rendered the text "home"; `Container(fluid)`,
`Skeleton(circle)`, `Spacer(xl)`, `List(ordered)`, `FileUpload(multiple)` were
not in the modifier vocabulary; `Row(gap:)` emitted a class with no CSS and
`align`/`justify` knew three values; `Tcell` was always `<td>` and tables had no
caption; `Link` never got `.active`/`aria-current`; `Progress(value:)` was
one-shot; lambda parameters became signal reads inside pages; `wf build` did
not run the V01/V02 lint or the semantic check; the catch-all page was written
to a directory named `*`; the router never set `document.title`; `wf serve`
never served a pre-rendered `<route>/index.html`.

New language features added for this design: `hover { }` / `focus { }` /
`active { }` / `disabled { }` / `placeholder { }` / `focus-within { }` blocks
inside `style { }`, compiled with `@media` into `styles.css` under
content-hashed classes (one rule per distinct block, CSP-safe).

## Adaptations that remain

- **Focus ring colour.** In structural mode `:focus-visible` is painted with
  `--color-primary`, so `color-primary` carries the design's `focus-ring`
  (`#ff9a6b`). The A13 lint reports it against a white button label that does
  not exist on this site (no `primary` modifier is used). One warning, accepted.
- **No inline SVG.** Sparklines → 12-bar mini strips; the architecture graph →
  a request-path table; the brand mark → two rotated squares; status dots →
  a 7px circle `Container`. Icons are limited to the 30 built-ins per the §4
  mapping; where none fits, the word alone.
- **No `@keyframes`.** No region marquee, no live-badge pulse, no caret blink.
  Mount animations use the built-in `fadeIn`/`slideUp` modifiers.
- **No `-webkit-*` property names** in a style block: `-webkit-font-smoothing`
  lives in `public/base.css`.
- **`Tabs` builtin** keeps its active index internally; every tab strip in the
  design is a `Seg` component over page state.
- **`Toast` builtin** cannot be styled; the toast stack is a list in `UiStore`.
- **Positional arguments to user components** emit invalid JavaScript; every
  component is called with named arguments.
- **Interpolation** `"{expr}"` refuses `,` and `:` inside the braces; strings
  such as `rgba(…)` or `"19/19"` are precomputed in store `derived`s.

## Milestone 2 — what the component library needed from the compiler

Writing the `.fl-*` contract as user components exposed five more gaps, all
fixed in the compiler rather than worked around:

- **Props were snapshots.** `Chip(pressed: on)` copied `on` once, so nothing
  inside the component ever saw it change. Props are now handed over as
  getters and read live.
- **Prop defaults were dropped by the SPA** (`dot: Bool = true` arrived as
  `undefined`), while the static paint applied them.
- **Handlers on a component call were dropped**, so a styled button component
  was unusable; they now attach to the component's root element.
- **Custom properties** (`--hover-bg: hoverBg`) could not be written, which is
  the only way a static `hover { }` rule can take a per-variant value.
- **Prop names collided with the modifier vocabulary**: `Text(text)` in a
  component with a `text` prop rendered nothing (the word was the input-type
  modifier). Declared names now shadow the vocabulary; `error:` is also
  usable as a prop name, and the A03/A04 lints accept `aria-labelledby`.

A prop cannot be called `on` (`on:` is the event syntax) or `use` (keyword);
the library uses `pressed`, `checked` and `note`.

## Milestones 3–4 — landing page and pricing

- **No `@keyframes`**: the region strip does not scroll and the `live` badge
  does not pulse. The hero uses the built-in `fadeIn`/`slideUp` mount motion.
- **Range slider thumb.** The design draws a 20px brand thumb with a ring
  through `::-webkit-slider-thumb`, a pseudo-element no style block reaches.
  The compiler now paints range inputs with `accent-color` from the palette;
  the thumb is the browser's, in brand.
- **The estimator's dial positions** live in page state (a `Slider` binds to
  page state only) and are forwarded to `PricingStore`, which owns the math.
- Compiler fixes this pair of pages needed: store derived values could not
  read other derived values, call actions or use if-expressions; a store's
  actions were bound after its derived values ran; an `else if` chain lost
  every branch when a final `else` was present; a `Slider` with both `bind:`
  and `on:input` kept only one; the heading-outline lint could not see the
  headings inside components; a component named `Section` was silently the
  builtin `Section` (renamed `Wrap`).

## Milestone 5 — docs, changelog, status

- **Inline `<code>` in prose.** A paragraph cannot mix a `Text` run with a code
  span, so tokens like `match.fn` sit in the running text unstyled. The
  parameter table and the code blocks are styled as the design draws them.
- **Footnote superscripts** are the Unicode ¹ ² characters in the sentence,
  not a link to the note.
- **The 90-day bars** are 450 `Container`s, five `for` loops of 90, rendered
  from `StatusStore.rows` (a derived list built with `Array.from` and an
  index map). They render instantly; the 30-day fallback was not needed.
- Prop and argument names cannot be `state` (keyword): `IncidentCard` takes
  `status`.
- Compiler fixes this milestone needed: a keyword after a dot (`Array.from`)
  was a parse error; a map literal returned from an arrow was a block; a
  sub-component dropped its style block, attributes and handlers in the SPA;
  an Input with both `bind:` and `on:input` lost its binding; a Button's
  block that mixed content with an action ran the action at render time.

## Milestones 6–7 — stores, the app shell, overview

- **`AppShell` is one component with children** (unknown #1 came back yes
  after the fix), holding the rail, the topbar and, below 768px, the Mobile
  artboard's tab bar. Rail items are `Link`s: `Sidebar.Item` reloaded the
  whole app on click before it was fixed, and the design's rail is plain
  links anyway.
- **The rail pill says 18/19 regions healthy** with a warning dot. The
  artboard paints "19/19" in green beside a degraded gru1 on the same
  screen; the data wins over the drawing here.
- **The current rail item** is styled with the new `current { }` block, a
  rule keyed off `aria-current="page"`, which the router sets. Chips, tabs
  and switches could use `pressed`/`selected`/`checked` the same way; the
  library keeps its derived colours where the value also drives a
  custom property.
- Sort comparators needed two-parameter lambdas, `Number.isFinite` and
  `Array.from`, none of which parsed; all three are compiler fixes.

## Milestone 8 — deployments

- **The checkbox is a `Button` with `role="checkbox"`** (`CheckBtn`). The
  builtin `Checkbox` has no `indeterminate`, and the select-all control needs
  the third state; `aria-checked="mixed"` carries it and draws the minus
  glyph. The spec's `effect` over `document.getElementById` was not needed.
- **Sortable headers are `Tcell(header)`**, a new modifier: the `<th>`
  detection is contextual and cannot see through a component call, so
  `SortTh` used to render a `<td>`. `aria-sort` on the `th`, the sort key and
  direction in `DeployStore`; a new column sorts ascending first, except
  Created, which is newest-first.
- **The row menu is a `Stack(role: "menu")`** under an `if`, not the builtin
  `Menu`: it closes on Escape (a `keydown` handler on the menu), on any
  choice, and on any other change to the table. The trigger carries
  `aria-haspopup`/`aria-expanded` through `IconBtn`'s `popup`/`expanded`
  props, because a component only forwards the props it declares.
- **The pager is decorative data** (`page` state, 11 pages, the same 12 rows):
  the design shows "12 of 128" and the seed has 12. The current page carries
  `aria-current="page"` via `FlButton(current:)`.
- **Under 768px the table is a card list** — both are rendered, one hidden
  by `@media` — with the same selection, badges and menu.
- Compiler fixes this milestone needed: `IconButton` dropped every named
  argument but `icon`/`label` in the SPA and painted its label as visible
  text in the static backends; `Tcell(header)` added.

## Milestone 9 — observability

- **Every chart is boxes.** The p95 line chart is grouped vertical bars (13
  buckets × 4 regions) with the same y-axis, gridlines, legend toggles and
  hover tooltip as the artboard; the crosshair and the dashed line styles
  are gone with the lines. Responses by status class are stacked bars,
  which unknown #2 (reactive style values) made possible. The hour-of-week
  heatmap is 168 boxes on the artboard's seven-step ramp, each with a
  `title`. Route share is the artboard's own horizontal bars.
- **The table view is real**: a `Table` named by the visible caption
  (`aria-labelledby`), 13 rows × 4 regions, following the range and the
  legend. Each chart container has `role="img"` and a name that says what
  it shows; the numbers behind it are in the table or beside the bar.
- **Hover state lives in the store** (`hover`), not in the bar rows, so the
  bars are not re-rendered under the cursor; only the tooltip is.
- A string with a `,` inside `{ }` is not interpolated (`Math.min(a, b)`), so
  the tooltip's `left` is computed in an action.
- Compiler fix this milestone needed: `Option("value", "Label")` dropped the
  label and used the value as text, in all three backends.
