# Route card — `dwsd.flightroute` / `.dwsd-flightroute`

> **When prose and code disagree, the code wins.** Facts from
> `src/route/yaml-parser.ts` + `src/route/types.ts` + `src/route/rules/*`. Fuller
> human reference: `docs/reference/route/index.md`. For route refs, the
> `bound-to:` stage-refinement locator, and the shared `title:` + `tags:` header,
> see **`shared.md`** — this card does not restate the locator grammar.

A **flight route** is the **abstract design layer** of a delivery: the **bands**
work moves through, the **stages** (an item type at a band) it passes, and the
glyph-typed **edges** between them. The `path:` references **no board** — a
separate **`bound-to:`** layer maps bands onto concrete boards. Plain **YAML**;
tolerant parser (never throws → `route.*` diagnostic + skip). The
preview-renderable Markdown fence is the dot form **`dwsd.flightroute`** (standalone
language id: `dwsd-flightroute`). The only string-sugar is the edge line itself.

## Header + section keys (closed schema)

| Key | Values / shape | Diagnostic on misuse |
|-----|----------------|----------------------|
| `title` | string — the route name (shared header; see `shared.md`) | (none) |
| `tags` | string list — **document scope only** (shared header) | tolerant |
| `for` | string `@itemType` — the primary item type the route describes | `route.entity-ref-without-for` (warn) |
| `bands` | list of `{ name, fl?: number \| number[] }` | `route.invalid-band` (skip); `route.invalid-band-fl` (warn) |
| `triggers` | list of `{ name, generates }` — external stickies OUTSIDE bands, fan-out capable | `route.invalid-trigger` (skip) |
| `path` | glyph-typed edge list (see below) | `route.invalid-edge` (skip/error); `route.edge-band-constraint` (warn) |
| `bound-to` | band→board bindings (see below) | `route.invalid-bound-to` (skip); `route.flow-not-subset-path` (**error**) |
| `notes` | list of `{ on, text, resolved? }` — inline annotations; `on:` is a position-only locator: `stage:<itemType> @ <band>`, or `.` = whole route. See **`shared.md` › Notes** | `route.note-orphan` (warn) |

- **Bands** — each `{ name, fl? }`. `fl` is a single number (one flight level) OR
  a numeric array (`[2, 3]`) → a multi-FL **span** (one color, **no gradient**).
  No `fl` = a pure custom band. `fl` outside `1..3` → `route.invalid-band-fl`.
- **Stages** — a stage is an `itemType @ Band` pair (e.g. `epic @ Strategy`). Never
  declared separately (no `stages:` section) — the parser **infers** them by
  walking every edge endpoint + trigger `generates` target. The `@` splits on its
  first occurrence (multi-word band names are preserved). Stages carry **no tags**.
- **Triggers** — `{ name, generates }`. `generates` is a scalar `itemType @ Band`
  or a list (fan-out). External sticky that sits outside the bands.

## Edges — the closed vocabulary (SEED-004 / Klaus-#20)

Each `path:` line is `LEFT <glyph> RIGHT`, endpoints in `itemType @ Band` form.
The glyph selects the kind:

| Kind | Glyph | Bands | Style | Constraint diagnostic |
|------|-------|-------|-------|-----------------------|
| **generate** | `->` (or `→`) | the **same** band | solid | `route.edge-band-constraint` (warn) if it crosses bands |
| **copy** | `-->` | **different** bands; fan-out capable | dashed | `route.edge-band-constraint` (warn) if same band |
| **feedback** | `..>` | band-**unconstrained** (backward info-flow) | dotted, bidirectional | (none — never band-checked) |
| **delivery** | `..> ()` | terminates at the reserved outside-band `()`/⊙ sink | fine-dotted → ⊙ | `route.invalid-edge` (**error**) if `()` reached via `->`/`-->` |

Band constraints are **teaching nudges (warnings)**, never hard blocks — the
preview always renders. Equivalent expanded object form:
`{ from, to, kind }` with `kind` ∈ `generate`/`copy`/`feedback`/`delivery`. A
multi-hop line (`a @ X -> b @ Y -> c @ Z`) is a `route.invalid-edge` **error** —
one hop per line. Delivery is reachable **only** via the dotted `..>` glyph.

## Binding — `bound-to:`

`bound-to:` is a list of band→board bindings: each entry names one **`band`**
(must be declared in `bands:`) and a list of **`boards`**. A **coarse** binding is
a bare `board:` locator; a **fine** binding adds a per-stage refinement mapping a
stage's item type to a board position via the **shared locator** (see `shared.md`).
The relation is N:M permissive (one route → many boards). An unbound route is
valid. The one hard error: **`route.flow-not-subset-path`** — every bound band
must be in `bands:` and every pinned stage must live in that band per `path:`. A
stale `board:` ref is `route.unresolved-board-ref` (**warning**, never a blank).

## Diagnostics → fix (`route.*`, selected)

| Code | Severity | Fix |
|------|----------|-----|
| `route.syntax-error` | error | Fix the YAML (executable `!!`-tags rejected). |
| `route.retired-key` | **error** | Rename root `route:` → `title:`. |
| `route.retired-deliver` | **error** | Author delivery as a dotted `stage ..> ()`. |
| `route.invalid-edge` | warn/error | One hop per line; reach `()` only via `..>`. |
| `route.invalid-band` / `-band-fl` | warn | Band needs a `name`; `fl` ∈ `1..3` (or a numeric span). |
| `route.edge-band-constraint` | warn | `generate` = same band, `copy` = different bands. |
| `route.flow-not-subset-path` | **error** | Bind only bands in `bands:` + stages the `path:` places in that band. |
| `route.invalid-trigger` / `-bound-to` | warn | Malformed entry skipped — fix its shape. |

## Not keys — do NOT write them

- **`bind:` / `realizes:` / `flow:`** → the band→board section is the single
  **`bound-to:`** key (rejected, not aliased).
- **`deliver` edge** — **retired**; author delivery as `stage ..> ()` (hard
  `route.retired-deliver`).
- **`route:` header** — **retired**; the document title is `title:` (hard
  `route.retired-key`).
- **`fl-path`** — legacy pre-M4 linear path key; do not author it in new routes.

## Parse-clean examples

```dwsd.flightroute
# minimal — one band, one generate edge between two stages.
title: Minimal Route
for: feature
bands:
  - name: Delivery
    fl: 1
path:
  - epic @ Delivery -> story @ Delivery
```

```dwsd.flightroute
# all three canonical edges: generate (solid), copy (dashed), feedback (dotted).
title: Edge Vocabulary
for: feature
bands:
  - name: Strategy
    fl: 3
  - name: Delivery
    fl: 1
path:
  - epic @ Strategy -> story @ Strategy
  - story @ Strategy --> task @ Delivery
  - task @ Delivery ..> epic @ Strategy
```

```dwsd.flightroute
# a copy into the delivery band, then a dotted `..> ()` delivery to the ⊙ sink,
# and a coarse `bound-to:` mapping the Ship band onto a concrete board.
title: Delivery Route
for: feature
bands:
  - name: Build
    fl: 2
  - name: Ship
    fl: 1
path:
  - account @ Build --> account @ Ship
  - account @ Ship ..> ()
bound-to:
  - band: Ship
    boards:
      - board: "[[team-alpha]]"
```

```dwsd.flightroute
# `notes:` — inline annotations. `on: "."` is a whole-route note (header bubble);
# `on: stage:…` anchors a painted stage; `resolved: true` recedes.
title: Notes
tags: [q3]
for: feature
bands:
  - name: Delivery
    fl: 1
path:
  - epic @ Delivery -> story @ Delivery
notes:
  - on: "."
    text: "**Whole-route retro** — revisit the hand-offs."
  - on: "stage:epic @ Delivery::before"
    text: Confirm the API contract before this stage
    resolved: true
```
