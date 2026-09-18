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

## Milestone 10 — logs, build detail

- **Regex validation without `try/catch`.** `LogStore.patternOk` scans the
  pattern with a `reduce` over its characters — balanced `( )` and `[ ]`,
  no trailing backslash, no quantifier with nothing before it — and the
  `RegExp` is only built when the scan passes. It catches what people type
  by mistake; an exotic pattern the scan accepts but the engine rejects
  would still throw. `RegExp(p, "i")` is called as a function: there is no
  `new` in the language.
- **Log lines are buttons** with the selection in `aria-pressed` (the
  artboard's `aria-selected` is not valid on a button) and an `aria-label`
  reading the whole line; ↑/↓ on the list moves the selection.
- **The trace detail** is an `if` beside the list; under 1024px it wraps
  below, and the list keeps `flex-basis: 100%` so the two never share a
  line at phone width.
- **`/app/builds/:hash`** reads `params.hash`, opens it in `BuildStore` and
  shows an empty state for a hash it does not know. The page is not
  pre-rendered (dynamic route): a static host serves `404.html`, whose
  static paint is the Not-found page for the instant before the router
  takes over. The diff's line tint is `data-kind` plus a background from
  the store; the attestation is a `CodeBlock` with escaped braces.
- Compiler fixes this milestone needed: an action parameter named like
  another action read the store member (`move(step)` beside `step`);
  `placeholder`/`disabled`/`src` and the other recognised attribute names
  were painted once even when bound to state; `404.html` addressed its
  assets relatively, so a dynamic route under a static build never
  hydrated. `for` inside an action body is silently dropped — not fixed,
  avoided with `.reduce`/`.map`.

## Milestone 11 — settings, explorer

- **No regex literals**: `RegExp("…")` is called as a function; a `{1,38}`
  quantifier inside the string is written `\{1,38\}` because `{` starts an
  interpolation.
- **Form controls are components** (`FlInput`, `FlSelect`) with the label
  named by `labelId`; the caller writes `on:input`/`on:change` on the call
  and the handler reaches the root element. Page state holds the draft for
  an input, the store owns the validation.
- **The explorer's filter rows are a list of ids**, with the field, operator
  and value in maps keyed by id. A `for` re-renders wholesale when its list
  changes, which would blur the input being typed in; keeping the typed
  value out of the list means only add and remove re-render the rows.
- The sub-nav under 768px is a horizontal, scrollable row of the same
  buttons. The icons `globe`/`lock`/`users`/`grid`/`eye-off` are not in the
  runtime's set of 30 and are replaced by `link`/`eye`/`user`/`menu`/`close`.
- Compiler fixes this milestone needed: a select's bound value was set
  before its options existed (runtime), and a slot inside a `Select`
  component was appended after the value (codegen).

## Milestone 12 — graph, editor, onboarding, overlays

- **The infrastructure graph is a table** of eleven hops in request order,
  each row's name a button with the selection in `aria-pressed` and the row
  tinted by `data-selected`; the inspector shows the hop's detail and its
  neighbours as chips. The artboard's zoom controls have no meaning for a
  table and are gone; the legend stays. The upstream row is badged "not
  ours" and the inspector shows a warning callout for it.
- **The editor has no syntax highlighting**: each line is a `Code` run with
  a line number in the first column and a `data-mark` (error/warn) that
  tints the row; the problems panel, the status bar and the resolved
  routing preview follow the open file. "Test a request" matches the path
  against the rules in the store.
- **Onboarding** keeps one `h1` (the A12 lint counts every `h1` in a page,
  even in exclusive `if` branches) whose text is a derived per step. Steps
  animate with `slideLeft`; the progress bar is the `ProgressBar` component.
  A step prop cannot be called `state`; it is `phase`.
- **Overlays are components over a `Scrim` button**, not the builtin `Modal`,
  whose `visible:` binds only to page-local state and whose paint cannot be
  restyled from a style block. The command palette, the project menu and
  the toast stack live in `AppShell`, so ⌘K works on every product page.
  Each overlay closes on its scrim, on Escape (a `keydown` on the dialog)
  and on its own buttons; focus is not trapped or restored, and the
  palette's input is not focused on open, because the language has no way
  to call `focus()` after a render.
- **Tooltips** are the builtin `Tooltip`, hover-only; its text box takes the
  structural sheet's inverted colours.
- A handler written as a block on a component call must be action
  statements only; an `if` inside makes it content, so the palette's
  options use an explicit `on:click { }`.
- Compiler fixes this milestone needed: an icon button drew its glyph twice
  (runtime); `data-state:` was a parse error because `state` is a keyword.

## Milestone 13 — polish

- **Every route in §5 exists** and is reachable: the marketing nav, the
  rail and the phone tab bar, the command palette (⌘K on every product
  page), the launchers on `/ui`, and `/patterns` from the footer. The
  catch-all page carries the site nav and three ways out.
- **Build is clean**: zero warnings. The last one (A13, a white label on
  `--color-primary`) was a lint that checked a variant the site never
  writes; the lint now looks for the modifier before it warns.
- **SEO**: the five marketing pages are in `sitemap.xml`; every `/app/*`
  page, `/new`, `/ui`, `/patterns` and the catch-all carry
  `noindex, follow`; the dynamic build route is not pre-rendered.
- **Accessibility sweep** on every route: one `h1` per page, every icon
  button and empty link named, every input and select labelled (a `label`,
  `aria-labelledby` or `aria-label`), every icon drawn (none falls back to
  its name as text), state in `aria-*` (pressed, checked, selected,
  current, expanded, invalid, sort) with a word beside every colour. Not
  done: focus is not trapped in or restored after an overlay (see 12).
- **Responsive pass** at 1440, 768 and 375 on every route: no horizontal
  overflow; marketing grids 3→2→1; product tables become card lists or
  scroll inside their card; side panels (trace detail, inspector, resolved
  routing, settings sub-nav, explorer rail, onboarding steps) wrap below
  the content; overlays become sheets under 480px; the rail becomes the
  Mobile artboard's tab bar under 768px.

## Compiler changes made for this design, in order

Each is one commit in the WebFluent repository with its own tests.

Fixes of documented behaviour: `Icon("home")`; component `children`; the
modifier vocabulary (`fluid circle multiple ordered xs sm md lg xl header`);
`Row(gap:/align:/justify:)` CSS; `Tcell` in `Thead` → `th`, `Tcell(header)`,
`Table(caption:)`; `Link(active:)` + `aria-current`; `Progress(value:)`
reactive; lambda parameters; V01/V02 and semantic checks in `wf build`;
`404.html`; `wf serve` directory index; `document.title` on route; keywords
after a dot; `Number`/`String`/`Boolean`/`Map` as globals; two-parameter
lambdas; map literal as arrow body; store derived reading derived/actions;
action order in `createStore`; else-if chains; Slider/Input bind + handler;
sub-component style/attrs/handlers; Button block mixing content and
actions; A11/A12 through components; A03 via `aria-labelledby`;
`IconButton` named arguments and label; `Option(value, label)`; an action
parameter shadowing a store member; known attributes (`placeholder`,
`disabled`, `src`…) following state; a select's value after its options
(runtime and codegen); an icon button drawing its glyph once; `data-state:`
(a keyword after a hyphen); A13 scoped to used variants.

Features: hyphenated named arguments → attributes (reactive); pseudo-state
blocks (`hover focus active disabled placeholder focus-within current
pressed selected checked expanded invalid`) and `@media` compiled into
`styles.css`; custom properties in style blocks; reactive style values;
`meta.fonts` / `meta.stylesheets`; digit-leading token segments;
`props` as live getters with declared defaults; handlers on component
calls attached to the root.

## Still not possible, and how the site lives with it

- No inline SVG, no `@keyframes`, no `try/catch`, no `new`, no regex
  literals, no `for` inside an action body, no way to call `focus()` after a
  render, no `indeterminate` property, no call-site `style { }` on a
  component, positional arguments to components, an `if` inside a
  component call's handler block, a `,` or `:` inside `{ }` in a string, a
  prop or key named `on`/`use`/`state`/`token`.
- Each has a workaround recorded in the milestone it came up in.
