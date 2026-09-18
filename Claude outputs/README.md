# The Halyard design, as files

This folder is the design that `HALYARD_BUILD.md` describes in prose. Where the two
disagree, **these files are what the design actually says** — the prose is a summary
written from them.

```
design/
  artboards/*.dc.html   the exact source of all 19 artboards  ← authoritative
  preview/*.html        the same screens as plain HTML        ← openable, approximate
  canvas.json           the frame, size and title of each artboard
```

## Which file to open when

**Reading the design — use `artboards/`.** Every value you need is in there and it
is exact: the hex, the padding, the radius, the font size, the class name, the
attribute that carries state. Read it the way you would read any source file. Do
not round a number you find here to a "nicer" one; the design was drawn with those
values on purpose.

**Looking at the design — use `preview/`.** Open one in a browser, or screenshot
it. These are generated from the artboards and are close but not exact (see the
caveats below), so never copy a value out of a preview — go back to the artboard.

## What an artboard file is

An artboard is one screen, written in the canvas editor's `.dc.html` format. It is
HTML with four additions, all of which are easy to read past:

| You will see | What it means |
|---|---|
| `<x-dc>` … `</x-dc>` | The wrapper around the screen. Ignore it. |
| `<helmet><style>` … | The page's CSS — the Fluant Ink tokens as custom properties, plus the `.fl-*` class contract. **This is the design system.** |
| `{{ some.value }}` | A data hole: a value that comes from the logic block at the bottom of the file. |
| `<sc-for list="…" as="x">`, `<sc-if value="…">` | A loop and a conditional. Children render once per item, or when the condition holds. |
| `<script type="text/x-dc">` at the end | The screen's state and handlers, as a plain JS class. Read it to see what each control actually does. |

Everything else is ordinary HTML with inline styles.

## The two things worth copying

**1. The token block.** Every artboard's `<helmet><style>` opens with the same
`:root { … }` — the complete Fluant Ink palette, fonts, radii, easings and
durations. That block is the single source of truth for colour and type. It is also
reproduced in `HALYARD_BUILD.md` §2 with each token's purpose spelled out.

**2. The class contract.** After the tokens comes a set of rules like:

```css
.fl-btn[data-variant="primary"] { background: var(--brand); color: var(--on-brand); }
.fl-badge[data-tone="success"]  { background: var(--success-soft); color: var(--text-success); }
.fl-switch[aria-checked="true"] { background: var(--accent); border-color: var(--accent); }
.fl-table td[data-numeric="true"] { text-align: right; font-variant-numeric: tabular-nums; }
```

That is the whole porting story: **state lives in ARIA attributes, variants live in
`data-*` attributes, and neither is ever a class name.** Whatever WebFluent emits,
if it carries the same element with the same attributes, it inherits the design —
and the accessible state and the visual state cannot drift apart, because there is
only one of them. `artboards/Patterns.dc.html` is the reference board: every
component rendered beside the exact markup to emit.

## Caveats on the previews

Generated mechanically, so:

- **Loops render once.** A table with twelve rows shows one. Look at the artboard's
  logic block, or at `HALYARD_BUILD.md` §6, for the real data.
- **Conditionals all render.** Anything behind an `sc-if` is shown, so a screen with
  an empty state may show both the list and the empty state at once.
- **Dynamic values are marked**, not filled: `⟨label⟩`, `⟨status⟩`. In the real
  design those come from state.
- **Nothing is interactive.** No sorting, no filtering, no overlays opening. The
  artboard's logic block is where the behaviour is written down.
- Fonts load from Google Fonts, so a preview needs network to look right.

A preview is for answering "roughly what does this screen look like". Every other
question goes to the artboard.

## Reading order

1. `artboards/Patterns.dc.html` — the design system, component by component.
2. `artboards/Main.dc.html` — the landing page, and the fullest use of the
   marketing styles.
3. `artboards/App-Deployments.dc.html` — the densest product screen: table,
   filters, selection, bulk bar, pagination, empty state.
4. `artboards/App-Observability.dc.html` — read this one before you build any
   chart. It is also the screen WebFluent will struggle with most.
5. `canvas.json` — how the 19 screens are grouped, and each one's intended size.

Then `HALYARD_BUILD.md` for the routes, the seed data and the build order.
