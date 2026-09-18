# Halyard — build spec for a WebFluent port

You are building **Halyard**, a fictional edge deploy platform, as a real WebFluent
application in this repository. A complete visual design exists as 19 artboards and
**it is in this repo at `design/`** — read `design/README.md` before you start.

- `design/artboards/*.dc.html` — the exact design source. Authoritative: every hex,
  padding, radius and font size you need is in there, along with the class contract
  and each screen's behaviour. **Where this file and an artboard disagree, the
  artboard wins** — this file is a summary written from them.
- `design/preview/*.html` — the same screens as plain HTML you can open or
  screenshot. Approximate by construction; never copy a value out of one.

This file adds what the artboards do not carry: the routes, the seed data, the
build order, and the gaps between the design and WebFluent. Do not invent content
that is not specified in either place — where a real fact is missing the design uses
a visibly marked placeholder such as `[YOUR PRICE]`, and so should you.

Read `AGENTS.md` in this repo first. It is the language reference and it is
authoritative. **If this file and `AGENTS.md` disagree about WebFluent syntax,
`AGENTS.md` wins** — this file describes *what* to build, not how the language works.

---

## 0. Before you write any pages: verify five unknowns

The design was drawn in HTML/CSS. Five things it relies on may or may not exist in
WebFluent, and guessing wrong will cost you the whole port. **Spend the first
iteration on a throwaway page, build it, read the generated CSS/HTML in `build/`,
and write your findings into `NOTES.md`.** Everything after this section assumes
you have answers.

| # | Question | Why it matters | If the answer is no |
|---|---|---|---|
| 1 | Does a user-defined `Component` accept a **block of children** (`MyShell { ... }`)? `AGENTS.md` only ever shows components called with props and no block. | Decides whether the app screens can share one shell component or must repeat sibling elements. | Compose the shell as **siblings**: every `/app/*` page contains `AppRail(current: "…")` and `AppTopbar(crumb: "…")` next to its own content inside a `Row`. |
| 2 | Can a `style` block value take an **interpolated or bound value** — `style { height: "{bar.pct}%" }` or a prop, `style { background: color }`? | Every chart, meter and heatmap cell needs a data-driven dimension or colour. | Use `Progress(value:, max:)` for every bar (it is documented as taking a dynamic value), a `Table` for anything else numeric, and drop the heatmap to a table of values. Say so in `NOTES.md`. |
| 3 | Do **CSS custom properties** survive a `style` block — `style { "--surface-raised": "#131519" }` — and is there any way to emit a global `<style>`? | Decides whether the Fluant tokens can live in one place or must be repeated as literal hex. | Put the tokens WebFluent knows about in the `Theme` block (§2) and use **literal hex values inline** everywhere else. Keep them in one `.wf` comment block so they are greppable. |
| 4 | Does the `Theme` token list extend beyond the four names shown in `AGENTS.md` (`color-primary`, `color-secondary`, `font-family`, `radius-md`)? Check the compiler's baseline theme in the build output. | Determines how much of the design system the theme can carry. | Set the four, inline the rest. |
| 5 | Is there **any escape hatch for raw markup or inline SVG**? | The design uses inline SVG for icons, sparklines and an architecture graph. | Use only the 30 built-in icons (§4), build charts from `Progress`/styled containers, and replace the architecture graph with a styled table of hops. |

Also confirm the obvious: `wf build` runs clean on the scaffold as it stands today,
before you change anything.

---

## 1. Ground rules

1. **Only documented syntax.** If a component, modifier or attribute is not in
   `AGENTS.md`, it does not exist. Do not port a CSS idea by inventing a component.
2. **Build constantly.** Run `wf build` after every file you add and after every
   milestone in §9. Never write five pages then build.
3. **Zero `V01` and `V02` warnings** at the end of each milestone. `V01` means you
   typed a modifier that resolves to nothing; `V02` means a real modifier with no
   stylesheet rule. Both mean the thing you wrote has no effect.
4. **Address the `A` and `S` diagnostics.** Every page gets a `title` and a
   `description` under ~160 characters. Every image gets alt text, every input a
   label, every table a header row.
5. **Replace the scaffold.** Everything currently in `src/` is the `wf init spa`
   demo (dashboard / tasks / settings / profile). Delete it as you go; do not build
   Halyard around it.
6. **Ink only.** The design has three themes; build the dark one. Do not add a
   theme switcher.
7. **Record every adaptation** in `NOTES.md` — anything the design does that
   WebFluent cannot, and what you did instead. That file is as much a deliverable
   as the app.

---

## 2. The design system: Fluant Ink

### Colour

Ink is a warm-dark system: near-black grounds, an orange brand, a mint accent, and
brass for editorial emphasis.

```
surface-canvas   #0b0c0e   page background
surface-sunken   #060708   footers, code blocks, wells
surface-raised   #131519   cards, panels, table headers, sticky bars
surface-overlay  #1a1d23   modal, drawer, popover, menu, toast
surface-inset    #101318   inputs, select triggers, progress tracks
surface-hover    #1f232a   hover ground for rows and ghost buttons
surface-active   #262b33   pressed and selected ground
surface-disabled #15181d
surface-scrim    rgba(4,5,7,0.74)

line-hairline    #1c2026   dividers, grid lines, table row separators
line-default     #2a2f37   resting card and panel edges
line-strong      #3d444f   hovered edges, button-group dividers
border-control   #6b7482   any control whose edge carries meaning
focus-ring       #ff9a6b   2px solid, offset 2px, never restyled per component

text-primary     #f4f1ec
text-secondary   #b9bec7
text-tertiary    #949ba7
text-disabled    #5d646f
text-inverse     #0b0c0e
text-brand       #ff8a5c   links, active nav, brand emphasis in copy
text-accent      #58e8bf
text-brass       #e6c979

brand            #ff6a2b   primary button, active tab, progress fill
brand-hover      #ff804a
brand-soft       #2a1409   tinted ground for brand badges and selected rows
on-brand         #180a03   text on a brand fill

accent           #35e0b0   switch on-state, selected chip, second series
accent-hover     #5aeac3
accent-soft      #07221c
on-accent        #04130f

brass            #d9b45a   pull-quote rules, third data series
brass-soft       #241c0c
on-brass         #1a1405

success  #3ad07f   soft #06231a   text #66dd9f   on #04170f
warning  #f5b53c   soft #241a06   text #f7c869   on #1c1301
danger   #ff5d5d   soft #2a0d0f   text #ff8585   on #1e0505
info     #48b6ef   soft #06202c   text #79ccf5   on #03151e

viz-1 #ff6a2b   viz-2 #35e0b0   viz-3 #d9b45a   viz-4 #48b6ef
viz-5 #ff8fc0   viz-6 #a9d84a   viz-7 #9b8cff   viz-8 #c98a62

selection  rgba(255,106,43,0.34)
```

Rules that are not negotiable:

- **Spend `brand` once per viewport** — the primary button, *or* the active tab,
  *or* the progress fill. Not all three. Everything else that must read as brand
  uses `text-brand` or `brand-soft`.
- **Never put raw text on a fill.** Each fill has an `on-…` partner.
- **Status is never hue alone.** Every status badge ships with a word.
- **Charts draw `viz-1` … `viz-8` in order**, never reordered to look nicer. Cap
  any one chart at **four series** — beyond that the palette fails colour-blind
  separation on this surface, which was measured, not guessed.
- **One gradient on the whole site**: the hero backdrop, brand fading into
  `surface-canvas`. No gradient on a button, card, border or icon.

### Type

Three families, loaded from Google Fonts:

```
display  Fraunces          "Fraunces", "Iowan Old Style", Georgia, serif
sans     Manrope           "Manrope", "Segoe UI", system-ui, sans-serif
mono     JetBrains Mono    "JetBrains Mono", ui-monospace, Menlo, monospace
```

| Style | Size / line-height / weight | Used for |
|---|---|---|
| display-2xl | 96 / 0.92 / 600, -0.03em | Hero headline. **One per page.** |
| display-xl | 72 / 0.94 / 600, -0.025em | Section opener on a long marketing page |
| display-lg | 56 / 1.0 / 600, -0.02em | Chapter titles, large pull quotes |
| display-md | 40 / 1.08 / 600, -0.015em | Editorial card headlines |
| quote | 28 / 1.35 / 400 italic | Pull quotes, with a brass rule above |
| heading-xl | 34 / 1.15 / 700, -0.02em | Page title inside the product UI |
| heading-lg | 26 / 1.2 / 700 | Section heading, modal title |
| heading-md | 20 / 1.3 / 700 | Card title, drawer title |
| heading-sm | 17 / 1.35 / 700 | Sub-section heading |
| body-lg | 18 / 1.6 / 400 | Lead paragraph. Cap measure at 62ch |
| body | 16 / 1.6 / 400 | Default copy. Cap at 72ch |
| body-sm | 14 / 1.55 / 400 | Table cells, card bodies, helper text |
| body-xs | 12 / 1.5 / 400 | Captions, timestamps. Never smaller |
| label-lg | 16 / 1.2 / 600 | Large button and tab label |
| label | 14 / 1.2 / 600 | Default control label |
| label-sm | 12 / 1.2 / 600 | Small button, chip, table header cell |
| overline | 11 / 1.2 / 700, 0.14em | Section eyebrow. **Typed uppercase in the source**, never `text-transform` |
| code | 14 / 1.6 | Code blocks on `surface-sunken` |
| code-sm | 12 / 1.5 | Inline tokens, keyboard hints, IDs |
| numeric | 15 / 1.2 / 500, tabular-nums | Stat tiles, counters, table numbers |

Display type belongs to marketing and editorial surfaces only. **Inside the product
UI start at `heading-xl` and never use a display style.** Anything that changes on
screen — counters, stat tiles, numeric table columns — uses `numeric` with
`font-variant-numeric: tabular-nums` so digits do not shuffle.

### Space, radius, depth, motion

- **Space** is a 4px scale: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96, 128, 160.
  Inside a control 8/12/16 by size; between related blocks 24; between sub-sections
  32/48; section padding 64 mobile, 80 tablet, 96 desktop.
- **Radius**: 0 for full-bleed bands and data tables; 3 checkbox/tag; 6 small
  button, input, menu item; 10 default button, card, popover, toast; 16 large card,
  modal, drawer; 24 feature panel; 36 showcase panel; 999 pill; 50% circle.
- **Depth (Ink)** separates with hairlines and glow, not soft shadows:
  ```
  shadow-sm    0 2px 8px rgba(0,0,0,.55)
  shadow-md    0 10px 24px rgba(0,0,0,.55)
  shadow-lg    0 24px 60px rgba(0,0,0,.65)
  shadow-xl    0 48px 120px rgba(0,0,0,.7)
  glow-brand   0 0 0 1px rgba(255,106,43,.5), 0 12px 40px rgba(255,106,43,.28)
  glow-accent  0 0 0 1px rgba(53,224,176,.45), 0 12px 40px rgba(53,224,176,.24)
  ```
  A surface is raised by exactly one step: canvas → raised → overlay. Never two
  shadows on one element.
- **Motion**: durations 80 (hover colour), 140 (tooltip, switch, tab underline),
  220 (menu, popover, toast, accordion), 380 (modal, drawer), 620 (scroll reveal),
  1100 (hero entrance — one per viewport). Curves: `cubic-bezier(0.2,0,0,1)`
  standard, `(0.16,1,0.3,1)` entrance, `(0.7,0,0.84,0)` exit,
  `(0.34,1.56,0.64,1)` spring, `(0.68,-0.4,0.27,1.4)` anticipate (hero only).
  Animate transform and opacity; travel 8–24px for a reveal, 4px for a press.

  In WebFluent, prefer the built-in modifiers (`fadeIn`, `slideUp`, `scaleIn`,
  `fast`, `slow`) and `animate(...)` on `if`/`for` over hand-written transitions.
  Use `transition { … }` blocks only where a built-in does not cover it.

- **Sizes**: control heights 30 / 38 / 46. Icon 14 / 18 / 24. **Minimum touch
  target 44px.** Containers: 720 article, 1040 product UI, 1280 marketing, 1600
  showcase. Grid gutter 24.

### The theme block

Declare this in `src/Theme.wf` (adjust the token names to whatever answer you got
to unknown #4):

```wf
Theme FluantInk {
    token color-primary: "#ff6a2b"
    token color-secondary: "#35e0b0"
    token font-family: "Manrope, 'Segoe UI', system-ui, sans-serif"
    token radius-md: "10px"
}
```

Then point `webfluent.app.json` at it: `"theme": { "name": "FluantInk" }`.

---

## 3. Voice — how the copy is written

This is part of the design and the compiler will not catch it.

- Write to the reader as **you**. Write **we** only about the team's own
  commitments. Never *the user*.
- **Sentence case everywhere** — headings, buttons, labels, table headers, toasts.
  The only uppercase is `overline`, typed uppercase in the source.
- **Buttons name the outcome, not the mechanism**: "Publish changes", "Start the
  run", "Delete 3 rows". Never "Submit", "OK", "Click here".
- **Errors state the cause and the next move in one sentence**: "That key expired
  on 12 March. Generate a new one in Settings → Keys." Never apologise, never
  blame the reader.
- **Empty states** name the missing thing and offer the action that creates it:
  "No runs yet. Start one to see timings here."
- **Numbers are the argument**: exact in `numeric`, rounded only in prose, always
  with the unit and the period — "412 ms p95, last 24h".
- **No emoji anywhere.** No exclamation marks. No sentence starting with "Simply",
  "Just" or "Easily".
- Marketing copy may be a fragment; product copy may not.

---

## 4. Known gaps between the design and WebFluent

| The design does | WebFluent has | Do this |
|---|---|---|
| Inline stroke SVG icons | 30 built-in icons, `Icon("name")` | Use the mapping below. **Where no icon fits, ship the label alone** — a wrong metaphor is worse than no icon. |
| Sparklines, line charts, a 168-cell heatmap | no SVG | See §7 (Observability). Bars from `Progress`, everything else a `Table`. |
| A clickable SVG architecture graph | no SVG | Replace with a "request path" table: hop, kind, what it does, p95. Keep the click-to-inspect behaviour using a selected-row state. |
| Sortable table headers with `aria-sort` | plain `Table` | Header cells contain a `Button` that calls a store action; sort in the store. |
| A right-hand drawer | `Modal` / `Dialog` | `Modal(visible:)` with a style block that pins it right and full-height, or `show` + a styled `Container`. |
| Command palette on ⌘K | `Modal` + `Input(search)` | Open it from a visible button in the navbar. Do **not** register a global keydown handler. |
| `backdrop-filter` sticky nav | CSS via style block | Keep it; it is plain CSS. Fall back to a solid `surface-canvas` if it misbehaves. |
| Accordion | `if` / `show` + `Button` | Store the open id in page state. |

**Icon mapping.** Available: `home menu search close user settings check plus minus
edit trash star heart mail bell download upload eye link calendar filter info
warning arrow-left arrow-right chevron-down chevron-right chevron-left logout copy`.

```
Overview      → home          Deployments  → upload
Observability → eye           Logs         → menu
Builds        → settings      Explorer     → search
Settings      → settings      Status       → info
Deploy        → upload        Roll back    → arrow-left
Copy hash     → copy          Delete       → trash
Filter        → filter        Search       → search
Notifications → bell          Docs         → link
Incident      → warning       Success      → check
Expand/collapse → chevron-down / chevron-right
```
No icon for: git branch, region, queue, edge KV, attestation, terminal. Use the
word.

---

## 5. Project layout

```
webfluent.app.json
src/
  Theme.wf
  App.wf
  stores/
    project.wf        ProjectStore     project meta, regions, live release
    deploys.wf        DeployStore      deployments, filter, sort, selection, page
    metrics.wf        MetricsStore     series, range, legend toggles
    logs.wf           LogStore         lines, level facets, search, follow, selection
    builds.wf         BuildStore       stages, tab, diff file, artifacts
    settings.wf       SettingsStore    project settings, env vars, domains, team
    explorer.wf       ExplorerStore    source, filter rows, result view
    onboarding.wf     OnboardStore     wizard step, repo, env selection
    content.wf        ContentStore     changelog entries, incidents, docs tree
    ui.wf             UiStore          command palette, toasts, active overlay
  components/
    SiteNav.wf  SiteFooter.wf  Mark.wf
    AppRail.wf  AppTopbar.wf
    StatTile.wf  StatusBadge.wf  SectionHead.wf  Overline.wf
    FeatureCard.wf  PlanCard.wf  FaqItem.wf
    DeployRow.wf  LogLine.wf  MetricBar.wf  IncidentCard.wf
    CodeBlock.wf  Placeholder.wf
  pages/
    Home.wf  Pricing.wf  Docs.wf  Changelog.wf  Status.wf
    AppOverview.wf  AppDeployments.wf  AppObservability.wf  AppLogs.wf
    AppBuild.wf  AppSettings.wf  AppExplorer.wf  AppGraph.wf  AppEditor.wf
    Onboarding.wf  Overlays.wf  Patterns.wf  NotFound.wf
public/
NOTES.md
```

### Routes

| Route | Page | Title | Description |
|---|---|---|---|
| `/` | Home | Ship to every region at once | Halyard builds your app once, signs it, and switches routing in all 19 regions in the same second. |
| `/pricing` | Pricing | Pricing | Pay for requests, not for seats. Every plan gets all 19 regions, atomic deploys and rollback by hash. |
| `/docs` | Docs | Routing rules | A routing rule decides which build or function answers a request. Rules live in your repository. |
| `/changelog` | Changelog | Changelog | Every change to the Halyard runtime, newest first, with the breaking ones marked. |
| `/status` | Status | Halyard status | Live status of the edge runtime, builds, edge KV, queues and the dashboard, across 19 regions. |
| `/app` | AppOverview | Overview | |
| `/app/deployments` | AppDeployments | Deployments | |
| `/app/observability` | AppObservability | Observability | |
| `/app/logs` | AppLogs | Logs | |
| `/app/builds/:hash` | AppBuild | Build | use `params.hash` |
| `/app/settings` | AppSettings | Settings | |
| `/app/explorer` | AppExplorer | Explorer | |
| `/app/graph` | AppGraph | Infrastructure | |
| `/app/editor` | AppEditor | Routing rules editor | |
| `/new` | Onboarding | New project | |
| `/ui` | Overlays | Interface inventory | `noindex` |
| `/patterns` | Patterns | Design reference | `noindex` |
| `*` | NotFound | Not found | `noindex` |

Every `/app/*` and flow page is `noindex` except the four marketing routes and
`/status`.

### App.wf and the two layouts

There are two layouts — a marketing site (navbar + footer) and a product UI (left
rail + topbar). WebFluent has one `Router`. Keep `App.wf` minimal and let each page
bring its own chrome:

```wf
App {
    Router {
        Route(path: "/", page: Home)
        // … every route above
        Route(path: "*", page: NotFound)
    }
}
```

Marketing pages start with `SiteNav()` and end with `SiteFooter()`. Product pages
put `AppRail(current: "overview")` and `AppTopbar(crumb: "Overview")` inside a
`Row`, alongside their content — unless unknown #1 came back yes, in which case a
shell component with children is cleaner and you should use it.

---

## 6. Seed data

All of it is fictional product data for a fictional product. Put it in stores, not
in pages.

### Project and regions

```
project   storefront-edge · hl-2201 · production · storefront.example.com
release   sha256:8f2c41…a7 · promoted 2 minutes ago by [YOUR NAME] · 100% of traffic
regions   fra1 34, iad1 41, cdg1 36, sfo1 52, gru1 148*, hnd1 61, syd1 74,
          bom1 88, jnb1 96, arn1 44, dub1 39, sin1 67, yyz1 45, icn1 63,
          cpt1 104, lhr1 34, ams1 35, nrt1 59, pdx1 49      (* degraded)
```
p95 in milliseconds. 19 regions, all operational except gru1, which is degraded.

### Deployments (12)

| hash | branch | message | env | status | duration s | regions | age |
|---|---|---|---|---|---|---|---|
| 8f2c41 | main | Pin beta cohort to a known build | production | ready | 22.4 | 19 | 2 min |
| 7c91de | main | Bump edge runtime to v9.4 | production | ready | 24.1 | 19 | 4 h |
| 5a04bb | fix/cookie-trim | Trim trailing spaces in cookie rules | preview | building | — | 0 | 4 h |
| 2ee7f0 | feat/queues | Queue consumer for order events | preview | failed | 18.9 | 0 | 1 d |
| 91ab3c | main | Raise api timeout to 10 s | production | ready | 21.7 | 19 | 1 d |
| 6d13c8 | perf/split | Split checkout bundle by route | preview | ready | 26.3 | 3 | 2 d |
| 4f80aa | perf/split | Drop unused polyfills | preview | canceled | — | 0 | 2 d |
| 0b92ef | main | Add cpt1 to the region set | production | ready | 23.8 | 19 | 3 d |
| cc41d7 | fix/cart | Fix stale cart on cookie rule miss | preview | ready | 20.9 | 3 | 4 d |
| e77a20 | feat/kv | Move session store to edge KV | preview | failed | 31.2 | 0 | 5 d |
| 3ab5f1 | main | Rollback: restore 0b92ef | production | ready | 0.4 | 19 | 5 d |
| 9c0e64 | main | Instrument the checkout trace | production | ready | 22.0 | 19 | 6 d |

Status tones: ready → success, building → info, failed → danger, canceled → neutral.
The table shows 12 of 128; pagination pretends there are 11 pages.

### Log lines (16, newest first)

```
14:02:41.118  info   fra1  GET /             200  24 ms    request completed
14:02:41.102  info   iad1  POST /api/cart    201  61 ms    cart item added, sku HL-2201
14:02:40.984  warn   gru1  GET /api/stock    200  1840 ms  upstream slow, served stale for 12 s
14:02:40.771  info   cdg1  GET /p/HL-2201    200  38 ms    request completed
14:02:40.640  error  gru1  GET /api/stock    504  9412 ms  upstream timed out after 9.4 s
14:02:40.402  info   sin1  GET /p/HL-3390    200  112 ms   cold start 8.4 ms included
14:02:40.233  debug  fra1  GET /             200  19 ms    rule 3 matched: path /*
14:02:40.019  info   lhr1  GET /checkout     200  34 ms    request completed
14:02:39.884  warn   iad1  POST /api/cart    429  6 ms     rate limit, 120 rpm per session
14:02:39.601  info   ams1  GET /assets/…css  200  4 ms     served from immutable cache
14:02:39.442  error  gru1  POST /api/order   500  212 ms   TypeError: cannot read stock of undefined
14:02:39.118  info   syd1  GET /             200  74 ms    request completed
14:02:38.977  debug  fra1  GET /api/stock    200  28 ms    kv read hit, leader fra1
14:02:38.640  info   yyz1  GET /p/HL-8802    404  12 ms    no product for that sku
14:02:38.401  info   dub1  GET /             200  31 ms    request completed
14:02:38.120  warn   cpt1  GET /api/stock    200  640 ms   kv read crossed region, leader fra1
```
Level colours: error `#ff5d5d`, warn `#f5b53c`, info `#48b6ef`, debug `#949ba7`.
Status colour by first digit: 5xx `text-danger`, 4xx `text-warning`, else
`text-success`. Anything ≥ 1000 ms shows its duration in `text-warning`.

### Build 8f2c41 — pipeline stages

| id | label | duration | state | note |
|---|---|---|---|---|
| resolve | Resolve | 0.9 s | done | 412 modules from the lockfile |
| compile | Compile | 14.2 s | done | 38 routes to the edge target |
| test | Test | 3.8 s | done | 204 assertions, 0 failing |
| sign | Sign | 0.4 s | done | sha256:8f2c41…a7 |
| pack | Pack | 1.1 s | done | 6.8 MB, 2.1 MB gzipped |
| ship | Ship | 2.0 s | active | 19 regions acknowledged |

Commit `a91f3c2`, branch `main`, author `[YOUR NAME]`, trigger `push`, total 22.4 s.

Artifacts: `edge/bundle.js` 1.84 MB, `edge/bundle.js.map` 3.12 MB,
`static/app.css` 82.4 kB, `static/fonts/*.woff2` 204 kB,
`attestation.intoto.jsonl` 4.1 kB.

### Routing rules (used by the editor, the docs page and the graph)

```
1  cookie ht_beta=1                → build 8f2c41
2  geo.country in DE, FR, NL       → region fra1
3  path /api/*                     → fn api, timeout 10 s
4  path /*                         → build current           (weight 99 — warning)
5  path /legacy/*                  → build deleted           (error HL-4019)
```

Errors to surface: `HL-4019` a rule after the fallback can never match;
`HL-4021` weighted rules in a group do not sum to 100; `HL-4033` `match.fn`
exceeded the 2 ms budget.

### Environment variables

```
DATABASE_URL         secret   All          postgres://halyard:••••••@db.internal:5432/store
STRIPE_SECRET_KEY    secret   Production   sk_live_••••••••••••4Kd9
SESSION_SIGNING_KEY  secret   All          ••••••••••••••••••••••••
NEXT_PUBLIC_CDN      plain    All          https://cdn.example.com
FEATURE_QUEUES       plain    Preview      true
LOG_LEVEL            plain    All          info
```

### Domains

```
storefront.example.com   Primary    production   Valid until 12 Dec
www.example.com          Redirect   production   Redirects to storefront.example.com
preview.example.com      Wildcard   preview      Valid until 12 Dec
shop.example.dev         Pending    preview      Add a CNAME to cname.halyard.dev
```

### Team

```
[YOUR NAME]    you@company.com   Owner       active
[TEAMMATE 1]   ak@company.com    Admin       active
[TEAMMATE 2]   mr@company.com    Developer   active
[TEAMMATE 3]   js@company.com    Viewer      invitation pending
```

### Changelog (10 entries, newest first)

```
v9.4  18 Sep  Routing rules ship with the build         added, changed   ← the long entry
v9.3  04 Sep  Queues leave preview                      added
v9.2  27 Aug  Cold start cut to 8.1 ms                  changed
v9.1  19 Aug  Trace export to your own sink             added
v9.0  05 Aug  Reproducible builds by default            changed, security
v8.9  22 Jul  Region fra1 capacity doubled              changed
v8.8  11 Jul  Fix: cookie rules ignored trailing spaces fixed
v8.7  01 Jul  Signed artifacts, SLSA level 3            security
v8.6  18 Jun  Monorepo project detection                added
v8.5  06 Jun  Fix: 504 on slow upstream kept the isolate warm   fixed
```
Tag tones: added → success, changed → info, fixed → warning, security → brand.

### Incidents (status page)

```
ACTIVE   warning  Elevated response time in gru1        18 September, 11:06 UTC
  Monitoring    13:48  p95 in gru1 is back under 200 ms. We are leaving this open for
                       another hour before we call it resolved.
  Identified    12:14  An upstream transit provider in São Paulo is dropping about 3% of
                       packets. We have shifted gru1 traffic onto a second path.
  Investigating 11:06  p95 in gru1 crossed 200 ms for five consecutive minutes. Requests
                       are still being served. No other region is affected.

RESOLVED danger   Queue consumers stalled for 41 minutes  14 September, 02:17 UTC
  Resolved      02:58  Consumers drained the backlog. No messages were lost; 18,402 were
                       delivered late, the oldest by 41 minutes.
  Identified    02:31  A leader election in the queue coordinator did not complete after a
                       routine node replacement.
  Investigating 02:17  Queue consumers stopped acknowledging messages in all regions.
                       Publishing was unaffected.

RESOLVED warning  Builds queued behind a cache rebuild   08 September, 09:40 UTC
  Resolved      10:22  The queue drained. Build times are back to a 22 second median.
  Investigating 09:40  A build cache migration is holding new builds in the queue for up
                       to six minutes.
```

Component uptime, 90 days: Edge runtime 100.000%, Builds 99.981%, Edge KV 99.997%,
Queues 99.902%, Dashboard and API 99.994%. Overall 99.975%.

---

## 7. Page specifications

Only the substance is given. Apply §2 and §3 throughout. Where the design used an
inline SVG, use the fallback from §4.

### `/` Home

Long scroll, `container-lg`. In order:

1. **Sticky nav** (`SiteNav`): mark + "Halyard"; links Product, Pricing, Docs,
   Changelog, Status; right side a search button that opens the command palette,
   "Sign in" → `/app`, "Start building" (primary) → `/new`.
2. **Hero**, with the one gradient backdrop.
   - overline `EDGE RUNTIME · 19 REGIONS · ONE ARTIFACT`
   - h1 (display-2xl): **Ship to every region at once.**
   - lead: "Halyard builds your app once, signs the artifact, and switches routing
     in all 19 regions in the same second. No region flags, no staged rollout to
     configure, no cache to warm afterwards."
   - buttons: "Start building" (primary, large) → `/new`; "Read the runtime docs"
     (secondary, large) → `/docs`
   - fine print: "Free under 100k requests a month. No card until you cross it."
   - beside it a terminal card on `surface-sunken`: five ✓ lines — resolved 412
     modules / compiled 38 routes → edge / signed sha256:8f2c41…a7 / packed 2.1 MB
     gzipped / 19/19 regions acknowledged — then "→ live in 412 ms". Footer strip:
     build 22.4 s, propagate 412 ms, dropped 0.
   - under the hero, a row of the 19 region codes.
3. **Stat row**, four tiles, "THE HALYARD DEMO PROJECT, LAST 24 HOURS":
   median build 22.4 s (−18% vs last week), global propagation 412 ms p95 (steady),
   cold start 8.1 ms (−2.4 ms since v9), rollbacks completed 1,204 this quarter
   (all under 600 ms).
4. **Customer proof strip** — five boxes reading `[LOGO 1]` … `[LOGO 5]` with the
   line "Replace these with your customer wordmarks." Do not invent companies.
5. **Feature grid**, six cards. Section opener: "Six things, and none of them are a
   dashboard toggle." Lead: "Everything that changes how a request is answered
   lives in the repository and ships with the build. If it is not in the artifact,
   it cannot break production."

   | Title | Body |
   |---|---|
   | Atomic global deploys | Every build lands in all 19 regions in the same second. There is no region flag to set and no staged rollout to configure, because there is no staged rollout. |
   | Signed, reproducible builds | Each build is content-addressed and signed at the end of the pipeline. Rebuild the same commit next year and the hash matches or the deploy is refused. |
   | Request-level routing | Route on header, cookie, geo or a value you compute in a 2 ms middleware. Rules are versioned with the deploy, not with a dashboard toggle. |
   | Edge state that is not a cache | Strongly consistent KV and queues that read in the region the request landed in. Writes replicate in under 90 ms p95 between any two regions. |
   | A trace on every request | Not a sample. Every request carries a trace with the build hash, the region, the route rule that matched and the cold-start cost, kept for 30 days. |
   | Rollback by hash | Any build that ever ran is one command away. Rollback is a routing change, not a rebuild, so it completes in about 400 ms worldwide. |

6. **Tabbed product preview** — "Watch the same release from three sides." Tabs
   Build / Route / Observe, each listing five rows with a status dot, plus a side
   panel explaining what you are looking at and a link out (Build → `/app/builds/8f2c41`,
   Route → `/app/editor`, Observe → `/app/logs`).
7. **Interface section** — "A config file, a CLI, and one REST route." Tabs
   `halyard.config.ts` / Terminal / REST, each a `Code(…, block)`.
8. **Pull quote** with a brass rule, body `[Replace with a real customer quote…]`,
   attribution `[NAME], [ROLE] at [COMPANY]`.
9. **Pricing teaser** — three plan cards (see `/pricing`), link "Compare every line".
10. **FAQ accordion**, five items:
    - *What happens to in-flight requests during a deploy?* They finish on the
      build they started on. Halyard switches routing, not processes: the old build
      keeps serving anything already in its queue and is drained once the last
      response leaves, usually inside two seconds.
    - *Can I keep a region out?* Yes, but it is a routing rule, not a build flag.
      The artifact still lands everywhere so a rollback or a geo change does not
      need a rebuild — the rule decides who is allowed to answer.
    - *Is the edge KV eventually consistent?* No. Reads are served from the region
      the request landed in and writes are linearised through a per-key leader.
      Cross-region write visibility is 90 ms p95; if you need it faster, pin the
      key's leader to the region that writes most.
    - *How do you price a request that fans out?* One request in, one request
      billed. Internal calls between your own functions, KV reads and queue
      operations are counted separately and shown per route in the usage view.
    - *What do I have to change to move an existing app?* A config file and,
      usually, your session store. Node built-ins that touch the filesystem are the
      common blocker — the CLI reports them before the first deploy rather than at
      runtime.
11. **Closing CTA** — "Your first deploy takes about four minutes." / "Point
    Halyard at a repository. It detects the framework, imports your environment,
    and tells you what will break before it builds." Buttons "Connect a repository"
    → `/new`, "Read the docs first" → `/docs`.
12. **Footer** (`SiteFooter`): four link columns (Product, Developers, Company,
    Legal), a green "All 19 regions operational" badge linking to `/status`, and
    `© [YEAR] [YOUR COMPANY]`.

### `/pricing`

- h1 "Pay for requests, not for seats." Billing-period segmented control
  (Monthly / Annual). Annual takes 20% off the per-seat price — so the Team price
  reads `[YOUR PRICE]` monthly and `[YOUR PRICE − 20%]` annually.
- Three plans:
  - **Hobby** — 0, forever. "One person, one project, and the same runtime everyone
    else gets." 100k requests a month · 1 concurrent build · 7-day traces · 1 custom
    domain · community support.
  - **Team** — `[YOUR PRICE]` per seat per month, marked "Most teams". "Pooled
    request volume across every project your team owns." 2M pooled requests a month
    · 8 concurrent builds · 30-day traces · preview deploy per pull request ·
    routing rules, edge KV and queues · deploy approvals and audit log.
  - **Enterprise** — "Talk to us", billed annually. "Dedicated capacity, your
    compliance paperwork, a named engineer." Negotiated request volume · unlimited
    builds · 1-year traces and export · dedicated regions or your own cloud · SSO,
    SCIM and a 3-year audit log · named support engineer.
- **Estimator** — four `Slider`s driving a live total shown in *billing units*, so
  no currency is invented. Requests a month (0.1M…100M, cost = 12 × M), build
  minutes (200…30 000, cost = minutes / 200), edge KV GB (1…500, cost = 0.6 × GB),
  seats (1…40, cost = 18/seat monthly or 14 annually). Show the four components as
  a breakdown and recommend a plan: Enterprise above 20M requests or 25 seats,
  Hobby at the floor with one seat, Team otherwise.
- **Comparison matrix**, 24 rows in five groups — Runtime, Builds, State and
  routing, Observability, Team and governance. Use the numbers already in the plan
  lists; `yes` renders as a check in `success`, `no` as a dash in `text-disabled`.
- Three add-ons (Dedicated region, Compliance pack, Extended traces), each priced
  `[YOUR PRICE]`.
- Three short FAQs: what counts as a request, what happens if I go over, can I move
  between plans mid-month.

### `/docs` — Routing rules

Three panes: nav tree, prose, table of contents. Tree sections Start (4), Runtime
(6, current page "Routing rules"), Builds (4), Operate (4), Reference (4).

Prose sections: The match object · Evaluation order · Targets · Computed rules ·
Rule parameters · What goes wrong. A tabbed code block
(`routes.halyard.ts` / `routes.json` / CLI) using the rules in §6, two callouts
(info: "Rules are part of the build"; warning: "Weighted rules must sum to 100"),
and a parameter table:

```
match.path     string | string[]         —        Glob against the pathname. A trailing /* matches any depth.
match.header   Record<string,string>     —        All listed headers must match, compared case-insensitively.
match.cookie   Record<string,string>     —        Exact cookie value match. Absent cookies never match.
match.geo      { country?, region?, asn? } —      Resolved at the edge. Falls through when the lookup is unavailable.
match.fn       (req) => boolean          —        A pure function evaluated in under 2 ms. Throwing counts as no match.
build          string                    current  A build hash, or current. Pinning freezes the rule.
fn             string                    —        Edge function name. Mutually exclusive with build.
timeout        number                    10000    Milliseconds before the request is cut and a 504 returned.
weight         number                    100      Share of matching traffic, 0–100. Must sum to 100.
```

Previous / next cards at the bottom (Edge functions ← → Middleware).

### `/changelog`

Left rail: type filter chips (Everything, Added, Changed, Fixed, Security) with
counts, then the ten releases, filtered live. Right: the full v9.4 entry —

- h1 "Routing rules now ship inside the build.", byline `[AUTHOR NAME]`,
  `[ROLE], runtime team`, 7 minute read.
- Opening: "Until this release, a routing rule was something you changed in a
  dashboard, and it took effect on whatever build happened to be live. That is a
  reasonable design right up to the first incident, when the rollback restores the
  code and leaves the routing exactly as broken as it was."
- A figure placeholder: `[DIAGRAM: rule evaluation at the edge]`.
- "What changed": rules read from `routes.halyard.ts` at build time and embedded in
  the artifact; the dashboard rule editor is now read-only and shows which file and
  line a rule came from; weighted splits must sum to exactly 100 or the deploy is
  refused with `HL-4021`.
- Pull quote: "If it is not in the artifact, it cannot be rolled back. That sentence
  is the whole design."
- Migration table: dashboard rules → exported on first deploy (automatic); rules
  edited after a deploy → refused (breaking); weighted splits → must sum to 100
  (breaking); `match.fn` → new, no action; `routes.json` v2 → still read (automatic).
- Two footnotes, and a subscribe form with real email validation (the error message
  is "That address is missing an @ or a domain. Fix it and try again.").

### `/status`

Public page, its own minimal header. Banner: "One region is degraded, everything
else is operational" in the warning tone, with 99.975% uptime over 90 days beside
it. Then:

- **Components** — the five from §6, each with an uptime figure and a 90-day bar
  strip. Ninety bars is a lot of elements; if that is slow or ugly in WebFluent,
  drop to 30 days and say so in `NOTES.md`.
- **Regions** — 19 cards, code, p95, and an Operational/Degraded badge.
- **Incident history** — the three incidents from §6, collapsible, with a filter
  (All / Active / Resolved). Phase colours: Investigating `#f7c869`, Identified
  `#79ccf5`, Monitoring `#58e8bf`, Resolved `#66dd9f`.
- **Subscribe** form, same validation as the changelog.
- Footer line: "Checks run every 30 seconds from 6 independent locations."

### `/app` Overview

Rail + topbar + content. Header: project name, Ready badge, `production` badge,
build hash, "promoted 2 minutes ago by [YOUR NAME]", the live domain. A time-range
segmented control (1h / 24h / 7d / 30d) that actually changes the numbers.

Four stat tiles (p95 response ms, error rate %, median build s, regions), a recent
deployments list (first five rows from §6), an activity timeline with five entries
and a show/hide detail toggle, and the 19-region health grid.

Timeline entries: release rl_4a91 promoted to 100% (14:02) · canary at 10% for 40
minutes (13:22) · build 8f2c41 signed (13:19) · alert cleared: gru1 latency (11:48)
· alert opened: gru1 latency (11:06, still active).

### `/app/deployments`

The densest screen. All 12 rows from §6, plus:

- search across hash, message and branch;
- status facet chips with counts;
- sortable columns (build, message, branch, environment, status, duration, regions,
  created) — sort lives in `DeployStore`;
- a select-all checkbox with an indeterminate state, per-row selection, and a
  floating bulk-action bar that appears when anything is selected (Promote, Export,
  Delete, and a clear-selection button);
- a per-row actions menu (open build, promote, copy hash, redeploy, delete);
- density toggle (comfortable / dense);
- pagination, 11 pages;
- an empty state when the filters match nothing: "No deployments match" / "Nothing
  in the last 128 deployments matches that search and those filters. Clear them to
  see the full list again." with a "Clear filters" button.

### `/app/observability`

Read §4 first — this is the screen most affected by the missing SVG.

Four panels:
1. **Response time p95** — four series (fra1, iad1, sin1, gru1) over 13 two-hourly
   buckets. As a chart: one horizontal `Progress` bar per region per bucket is too
   many, so render the *current* value per region as four bars, and put the full
   13×4 series in the table view beneath. Legend toggles a series on and off.
   Series data (24h): fra1 34 33 32 34 38 41 44 46 43 40 37 35 34 · iad1 44 42 40
   39 41 47 53 58 55 51 47 45 43 · sin1 62 60 58 61 66 69 72 70 68 66 64 63 61 ·
   gru1 88 84 82 90 118 164 198 176 132 104 96 92 89.
2. **Responses by status class** — 2xx / 4xx / 5xx per two-hour bucket, as stacked
   bars if unknown #2 allowed it, otherwise three `Progress` bars of the totals
   plus a table. 2xx 88 91 94 96 99 104 112 118 114 106 98 92 · 4xx 6 5 6 7 6 8 9
   11 10 8 7 6 · 5xx 1 1 0 1 2 3 7 12 5 2 1 1 (thousands).
3. **Latency by hour of week** — a 7×24 heatmap. Only build this if unknown #2 came
   back yes; otherwise a table of the seven daily peaks. The Wednesday-afternoon
   band is the weekly catalogue import, and the caption should say so.
4. **Share of requests by route** — one series, so no legend and every bar carries
   its value: `GET /` 31.4%, `GET /p/:sku` 24.8%, `GET /api/stock` 16.2%,
   `POST /api/cart` 11.5%, `GET /checkout` 9.3%, Other (42 routes) 6.8%.

A time range, a region select, an environment select, and a **table view toggle**
that shows the p95 series as a real `Table`. The table view is not optional — it is
how this screen stays usable without SVG.

### `/app/logs`

Two panes. Left: toolbar (search with a regex toggle, four severity facet chips
with counts, a follow switch, a drain button) over the 16 lines from §6, monospace,
each row clickable. An invalid regex shows a danger callout: "That pattern will not
compile. Fix the expression, or turn the `.*` switch off to search for it
literally."

Right: a detail panel for the selected line — trace id, route, status badge,
duration, region; a span waterfall (edge accept 2 ms, route match 2 ms, kv read
stock 18 ms, upstream fetch 168 ms, render 16 ms, egress 6 ms, total 212 ms) built
from `Progress` bars; the message; six request headers; and the routing rule that
matched, linking to the build.

### `/app/builds/:hash`

Header with the hash, status badges, and Redeploy / Roll back actions; a metadata
strip (commit, branch, author, trigger, duration, regions); the six pipeline stages
as a clickable strip; then `Tabs`:

- **Output** — terminal-style log for the selected stage. Compile shows a warning
  line: "! /api/order imports node:fs, stubbed at build time".
- **Diff** — four changed files (`routes.halyard.ts` +18 −4, `src/api/cart.ts`
  +6 −2, `src/lib/session.ts` +0 −22, `halyard.config.ts` +3 −1) and a unified diff
  of the routing rules, added lines on a green tint, removed on red.
- **Artifacts** — the table from §6, each row with a download button.
- **Attestation** — a "Reproducible" card (SLSA level 3, in-toto v1) beside an
  in-toto JSON statement in a `Code(…, block)`.

### `/app/settings`

Five sections, driven by a sub-nav: General, Domains, Environment, Team, Danger zone.

- **General** — project name with live validation (`^[a-z0-9][a-z0-9-]{1,38}$`;
  the error is "Lower case letters, digits and hyphens only, between 2 and 39
  characters."), framework preset select, root directory, two switches (reuse the
  build cache; require a matching attestation), and a three-option radio group for
  deploy approvals (none / one approver / two approvers, one an owner — the last is
  the current setting).
- **Domains** — add-a-domain form validating a hostname (error: "That is not a
  hostname. Enter something like shop.example.com, with no scheme and no path.";
  success: "Add a CNAME for … pointing at cname.halyard.dev, then it starts
  serving.") over the four domains from §6.
- **Environment** — the six variables, secrets masked with a per-row reveal toggle,
  plus an info callout: "Three of these are referenced by a build that is live right
  now. Changing one does not restart that build — the next deploy picks it up."
- **Team** — invite form and the four members with role selects.
- **Danger zone** — a reversible "Pause the project" card, then a destructive delete
  card whose button stays disabled until the project name is typed exactly.

### `/app/explorer`

Query builder. Saved queries rail (5), a source select (traces / logs / builds),
time range, group-by and measure selects, and **filter rows you can add and
remove** — field select, operator select, value input, remove button. The generated
query shows live underneath:

```
from traces where region is 'gru1' and duration_ms > 500 and status is not '404'
group by region, route select count(), p50(duration_ms), p99(duration_ms)
```

Results as a table or as bars, toggled:

```
gru1  GET /api/stock    412  1840  9412  18.4%
sin1  GET /p/:sku       286   112   940  12.8%
bom1  GET /api/stock    204    96   812   9.1%
jnb1  GET /             168    88   640   7.5%
cpt1  GET /p/:sku       142   104   588   6.4%
syd1  POST /api/cart    121    74   420   5.4%
hnd1  GET /              98    61   388   4.4%
icn1  GET /checkout      84    63   344   3.8%
```
(region, route, requests, p50 ms, p99 ms, share). A p99 over one second is shown in
`text-warning` — and the number says it too, never the colour alone. Note under the
results: "8 groups from 1,024,918 traces, scanned in 340 ms", replaced by "The query
changed. Run it to refresh these 8 groups." when a filter is edited.

### `/app/graph`

The SVG graph becomes a **request path table** with a selected row and an inspector
panel. Hops, in order, with kind and what they do:

```
Browser / Mobile app / Partner API   client    where the request starts
Anycast edge, 19 POPs                edge      TLS terminated here; p95 accept 1.8 ms
Rule engine                          edge      5 rules, evaluated in order, 2 ms budget
Middleware                           edge      auth, geo, experiments
Immutable assets                     edge      1 year, edge cached
fn api                               compute   38 routes, 10 s timeout, cold start p95 8.1 ms
fn checkout                          compute   8 routes
SSR render                           compute   isolate pool
Edge KV                              state     leader fra1, 90 ms p95 replication, 12.4 GB
Queue                                state     cart.updated
Upstream inventory                   external  [YOUR UPSTREAM HOST], p95 184 ms, 504 after 9.4 s
```

The inspector shows the selected hop's detail and its neighbours as chips you can
click. Mark the upstream row clearly: it is the only hop Halyard does not control.

### `/app/editor`

File tree, open-file tabs, a line-numbered view of `routes.halyard.ts` (and
`session.ts`, `cart.ts`, `halyard.config.ts`), a problems panel, and a
resolved-routing preview pane showing the five rules with ok / warning / error
marks. `session.ts` carries one error: "node:fs is not available in the edge
runtime. Move this read to halyard/kv." Without syntax highlighting, use
`Code(…, block)` and put the line numbers in a first column.

### `/new` Onboarding

Four steps with a progress bar, real back/next state, and per-step validation.

1. **Repository** — filterable list of five repos (`company/storefront` selected,
   `storefront-api`, `design-system`, `marketing-site`, `internal-tools`).
2. **Framework** — "Next.js 15 detected from package.json. 38 routes, 6 of them
   dynamic." Preset, node version, build command, output directory. Two warnings:
   *danger* — "src/lib/session.ts imports node:fs. The edge runtime has no
   filesystem. Move the session store to edge KV, or keep this route on a node
   function."; *warning* — "3 environment variables are unset in preview."
3. **Environment** — the six variables with checkboxes and a select-all.
4. **Review** — a summary table with per-row Edit buttons that jump back to the
   right step, and a final warning that the unresolved `node:fs` import will make
   that route return 500 until it is fixed. The final button creates the project
   and goes to the build page.

### `/ui` Overlays

The interface inventory: four alert tones, skeleton / spinner / progress / empty
state, tooltips, a toast stack you can add to and dismiss, and launchers for the
command palette, a modal (a rollback confirmation), a drawer (a trace filter
panel), a dropdown menu and a popover. Useful to you as a smoke test — build it
early.

### `/patterns`

The design reference: colour swatches with token names and hex, the type scale as
specimens, and one example of each component with a short note. Plus five porting
rules, which are worth restating because they are why the port works at all:

1. The element is part of the contract — a button is a button, a field has a label.
2. State lives in attributes, not class names, so visual and accessible state
   cannot drift apart.
3. Variants live in data attributes.
4. Layout is flex or grid with `gap`, never margins between siblings.
5. Colour is never the only signal.

### `*` NotFound

"That route is not here." / "No build claims this path. The fallback rule sent you
to this page." with links back to `/` and `/status`.

---

## 8. Accessibility — the parts a compiler will not catch

- Real elements everywhere: `Button` for actions, `Link` for navigation, `Input`
  with a `label` for every field. Never a clickable container.
- Every icon-only button has a label — `IconButton(icon: "close", label: "Close")`.
- Touch targets ≥ 44px, including in dense tables.
- Text contrast 4.5:1 (3:1 above 24px). The two that fail most often are caption
  grey on a raised surface and white text on a coloured fill — darken both.
- Every table has a header row and a caption, even a visually hidden one.
- Status, always a word beside the colour.
- Respect `prefers-reduced-motion`. If WebFluent's animation modifiers do not
  already handle it, add the media query in a style block and say so in `NOTES.md`.

---

## 9. Build order

Do these in order. **Run `wf build` at the end of every milestone and do not start
the next one until it is clean.**

| # | Milestone | Done when |
|---|---|---|
| 0 | The five unknowns (§0) | `NOTES.md` has five answers and a throwaway page built clean |
| 1 | Config, `Theme.wf`, `App.wf` with every route, `NotFound`, empty page stubs | `wf build` clean, every route reachable, no `V01`/`V02` |
| 2 | `SiteNav`, `SiteFooter`, `Mark`, `Overline`, `SectionHead`, `StatTile`, `StatusBadge`, `Placeholder` | `/patterns` renders every one of them |
| 3 | `/` Home, end to end | Hero, features, tabs, FAQ and footer all work; one display headline on the page |
| 4 | `/pricing` | Sliders recompute the total; the matrix is complete |
| 5 | `/docs`, `/changelog`, `/status` | Filters and accordions work; `/status` shows the active incident |
| 6 | Stores: project, deploys, logs, metrics, builds, settings | Built and used by at least one page each |
| 7 | `AppRail`, `AppTopbar`, `/app` Overview | Range control changes the numbers |
| 8 | `/app/deployments` | Sort, filter, search, selection, bulk bar, pagination, empty state |
| 9 | `/app/observability` | Charts within the §4 constraints **and** a working table view |
| 10 | `/app/logs`, `/app/builds/:hash` | Row selection drives the detail panel; tabs switch |
| 11 | `/app/settings`, `/app/explorer` | Validation fires; filter rows add and remove |
| 12 | `/app/graph`, `/app/editor`, `/new`, `/ui` | Wizard validates per step |
| 13 | Polish: SEO meta, sitemap, `noindex`, `csp`, a11y sweep | Zero `V`, `S` and `A` warnings you have not consciously accepted |

Commit at every milestone. Message format: `halyard: <milestone> — <what changed>`.

Suggested config once §0 is answered:

```json
{
  "name": "Halyard",
  "theme": { "name": "FluantInk" },
  "build": { "output": "./build", "minify": true, "ssg": true, "csp": false },
  "meta": {
    "site_url": "https://halyard.example.com",
    "site_name": "Halyard",
    "description": "An edge runtime that treats a deploy as a routing change.",
    "lang": "en",
    "sitemap": true
  }
}
```

`ssg: true` matters here: the marketing pages and the status page should be
readable before JavaScript runs.

---

## 10. Definition of done

- `wf build` is clean, and `wf serve` gives a site you can click through end to end.
- Every route in §5 exists and is reachable from the navigation.
- No lorem ipsum, no invented company names, no invented prices. Every gap is a
  visible `[PLACEHOLDER]`.
- The Ink palette is used as specified: one `brand` element per viewport, every
  status carrying a word, charts capped at four series in fixed order.
- `NOTES.md` lists every place WebFluent could not do what the design does, and
  what you did instead. If that file is empty, you have not been honest about the
  gaps.

---

## Appendix — where this came from

The design is a 19-artboard canvas of the same product, built on the Fluant Ink
design system. It is in this repo at `design/`, and every token, every piece of copy
and every row of seed data in this file is lifted from it.

So when a question is about **what something looks like or how it behaves**, open
the artboard rather than guessing from this file:

| Question | File |
|---|---|
| What does this component look like, and what markup does it need? | `design/artboards/Patterns.dc.html` |
| How is the marketing page built? | `design/artboards/Main.dc.html`, `Pricing.dc.html`, `Docs.dc.html`, `Article.dc.html`, `Status.dc.html` |
| How is the product shell built? | `design/artboards/App-Overview.dc.html` |
| How does the dense table behave? | `design/artboards/App-Deployments.dc.html` |
| What do the charts actually do? | `design/artboards/App-Observability.dc.html` |
| What does a drawer / palette / toast look like? | `design/artboards/Overlays.dc.html` |
| What does the phone version look like? | `design/artboards/Mobile.dc.html` |

Each artboard's `<script type="text/x-dc">` block at the bottom holds that screen's
state and handlers in plain JavaScript — that is where the behaviour is written
down, and it maps almost directly onto a WebFluent store.

Where both this file and the artboards are silent, the honest answer is that the
design did not specify it: make the smallest reasonable choice, write it in
`NOTES.md`, and move on.
