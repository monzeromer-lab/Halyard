# Halyard

Halyard is a fictional edge-deploy platform — "an edge runtime that treats a
deploy as a routing change" — built entirely in [WebFluent](https://github.com/monzeromer-lab/WebFluent),
from a 19-artboard design on the Fluant Ink design system.

It exists to answer one question honestly: can a real product design be
implemented *in the language*, without escaping to hand-written HTML, CSS or
JavaScript? Every page, component, store and style in `src/` is `.wf`. Where
the language could not do what the design does, the gap is written down in
[NOTES.md](NOTES.md), together with what was done instead — and, in most
cases, the compiler commit that closed it.

## What is in here

| Route | Page |
|---|---|
| `/` `/pricing` `/docs` `/changelog` `/status` | Marketing site, pre-rendered and indexed |
| `/app` | Project overview: tiles, recent deployments, activity |
| `/app/deployments` | Sortable table, facets, tri-state select-all, row menu, pager; a card list on phones |
| `/app/observability` | Four charts built from styled containers, legend toggles, a real table view |
| `/app/logs` | Two-pane log viewer, regex search, keyboard selection, trace detail with span waterfall |
| `/app/builds/:hash` | Pipeline strip, output, diff, artifacts, attestation |
| `/app/settings` | General, domains, environment (masked secrets), team, danger zone |
| `/app/explorer` | Query builder with filter rows you add and remove, live HQL, table or bar results |
| `/app/graph` | The infrastructure graph as a request-path table with an inspector |
| `/app/editor` | Routing-rules editor: file tree, tabs, line-numbered code, problems, resolved routing |
| `/new` | Four-step onboarding with per-step validation |
| `/ui` | Overlay inventory: command palette, modal, drawer, menu, popover, toasts |
| `/patterns` | The design reference: tokens, type, every component |

Everything is fully responsive down to 375px; under 768px the product rail
becomes the Mobile artboard's bottom tab bar and tables become card lists.

```
src/
  App.wf           the router, 18 routes
  Theme.wf         Fluant Ink as 94 theme tokens (→ CSS custom properties)
  pages/           one file per route
  components/      95 components, the Fluant Ink library in .wf
  stores/          14 stores: all data (seeded from the design) and all logic
public/base.css    three lines no element-level style block can express
design/            the artboards (authoritative), previews, canvas
HALYARD_BUILD.md   the build spec: routes, seed data, page specs, milestones
NOTES.md           every adaptation, every compiler change, every remaining gap
AGENTS.md          the WebFluent language reference the build was written against
```

## Building it

You need the `wf` compiler at commit `d0888a8` or later — the build relies on
features and fixes made for this project (component `children`, pseudo-state
blocks, hyphenated `aria-*`/`data-*` arguments, reactive style values,
`Tcell(header)`, `meta.fonts`, and more; see NOTES.md).

```bash
git clone https://github.com/monzeromer-lab/WebFluent.git
cd WebFluent && cargo install --path .
```

Then, in this repository:

```bash
wf build
```

```bash
wf serve
```

`wf build` writes a static site to `build/` (pre-rendered marketing pages,
`sitemap.xml`, `robots.txt`, `404.html` for the dynamic and catch-all
routes) and should finish with zero warnings. `wf serve` serves it on
port 3000.

## How it was built

Two tracks, in order, one commit per step:

1. **The compiler.** Read the WebFluent source (not just its docs), find what
   the design needs that the language does not do, fix it there with tests.
   Fifty-odd commits: documented-but-broken behaviour first, then small
   features.
2. **The app.** Milestones 0–13 from `HALYARD_BUILD.md` — config and theme,
   the component library, each page in turn, then polish — each verified in
   a browser at 1440, 768 and 375px and committed as
   `halyard: <milestone> — <what changed>`.

The rules the port follows are the design's own: state lives in ARIA
attributes, variants in `data-*`, every status carries a word beside its
colour, one primary action per viewport, charts never exceed four series in a
fixed order, and nothing is invented — every gap is a visible placeholder.
