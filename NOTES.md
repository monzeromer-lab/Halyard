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
