# Topology card — `dwsd-topology` / `.dwsd-topology`

> **When prose and code disagree, the code wins.** Facts from
> `src/topology/parser.ts` + `src/topology/validate.ts`. Fuller human reference:
> `docs/reference/topology/index.md`. For anchors and the shared `title:` +
> `tags:` header, see **`shared.md`**.

A **topology** describes a **Work-System Local View**: a focal Work System plus
the 1-hop ego-slice around it (the value chain contributing up to it, plus what it
contributes to). Scoped and focal — **not** a global org map. The only
relationship between Work Systems is **`contributes-to`**. Plain **YAML**;
tolerant parser (never throws → `topology.*` diagnostic + clamp/skip). The
live-render fence is **`dwsd-topology`** (all four spellings —
`wsd-topology` / `dwsd-topology` / `wsd.topology` / `dwsd.topology` — are accepted).

## Top-level keys (closed schema)

| Key | Values / shape | Diagnostic on misuse |
|-----|----------------|----------------------|
| `mode` | **`infer` \| `declared` \| `mixed`** (default `infer`) | `topology.unknown-mode` (warn; → `infer`) |
| `title` | string (shared header; see `shared.md`) | (none) |
| `tags` | string list — document-level chips (shared header) | tolerant |
| `focus` | **`self` \| `[[Target]]` \| `[[Target\|Alias]]`** | `topology.invalid-focus` (parse); `topology.unresolved-focus` (validate) |
| `radius` | a number \| `inf` \| `{ up, down, lateral }` (default `{1,1,1}`) | `topology.unsupported-radius` |
| `work-systems` (alias `work_systems`) | list of Work-System objects | `topology.invalid-work-system` (skip) |
| `contributes-to` (alias `contributes_to`) | list of edge objects | `topology.invalid-edge` (skip) |
| `notes` | list of `{ on, text, resolved? }` — inline annotations; `on:` is a position-only locator: `node:<name>` (a bare name or `[[node]]` too, incl. an inferred node in `mode: infer`), or `.` = whole topology. See **`shared.md` › Notes** | `topology.note-orphan` (warn) |

- **Modes**: `infer` (derive the Local View from the indexer), `declared` (author
  lists everything explicitly), `mixed` (declared entries overlay the inferred
  topology as a `delta` diff).
- **`radius`** bounds the ego-slice. A bare number applies to `up`/`down`/`lateral`;
  `inf` = transitive closure; the object form fills missing keys with `1`. Multi-hop
  `up`/`down` are honored; **`lateral > 1` is not yet supported** (`topology.unsupported-radius` info).

## Work-system fields (each `work-systems` entry)

| Field | Values / shape | Diagnostic on misuse |
|-------|----------------|----------------------|
| `name` | string (**required**) | `topology.invalid-work-system` (entry skipped) |
| `flightlevel` | **`1` \| `2` \| `3`** (1 operational, 2 coordination, 3 strategy) | `topology.invalid-flightlevel` (warn) |
| `delta` | **`add` \| `change` \| `remove`** (mixed-mode diff marker) | `topology.unknown-delta` (warn) |
| `tags` | string list (a scalar is a one-element list) | tolerant |
| `anchors` | map `typeKey → string list` (e.g. `agreement`, `meeting`, `individual`) | `topology.invalid-anchors` |

## Edge fields (each `contributes-to` entry)

| Field | Values / shape | Diagnostic on misuse |
|-------|----------------|----------------------|
| `from` | string (**required**) | `topology.invalid-edge` (skipped) |
| `to` | string (**required**) | `topology.invalid-edge` (skipped) |
| `delta` | `add` \| `change` \| `remove` | `topology.unknown-delta` (warn) |

A `contributes-to` edge points from a contributor **up** to the Work System it
feeds — it should land on the same or a higher flight level (a downward edge →
`topology.downward-contribution` warn at validate). A repeated `(from, to)` pair →
`topology.duplicate-edge` (info), second occurrence ignored.

## Not keys — do NOT write them

- **`host` / `member-of`** — membership and hosting are not part of the grammar.
  The only inter-system relationship is `contributes-to`.

## Closed value sets

- **`mode`**: `infer` · `declared` · `mixed`.
- **`flightlevel`** (work-system): `1` · `2` · `3`.
- **`delta`**: `add` · `change` · `remove`.
- **`focus`**: `self` or a `[[wikilink]]`.

## Parse-clean examples

```dwsd-topology
# minimal — the smallest valid topology: a single declared work-system.
mode: declared
work-systems:
  - { name: "Squad Aurora", flightlevel: 1 }
```

```dwsd-topology
# the three Flight Level bands (1 operational, 2 coordination, 3 strategy).
mode: declared
work-systems:
  - { name: "Squad Aurora", flightlevel: 1 }
  - { name: "Reliability Mid", flightlevel: 2 }
  - { name: "Delivery Ops", flightlevel: 3 }
contributes-to:
  - { from: "Squad Aurora", to: "Reliability Mid" }
  - { from: "Reliability Mid", to: "Delivery Ops" }
```

```dwsd-topology
# tags on work-systems + a document-level title/tags shared header.
title: Tagged Topology
tags: [ops, pilot]
mode: declared
work-systems:
  - { name: "Squad Aurora", flightlevel: 1, tags: ["delivery", "core"] }
  - { name: "Delivery Ops", flightlevel: 3, tags: ["strategy"] }
```

```dwsd-topology
# `notes:` — inline annotations. `on: "."` is a whole-topology note (header bubble);
# `on: node:…` anchors a rendered node; `resolved: true` recedes.
title: Notes
tags: [review]
mode: declared
work-systems:
  - { name: "Squad Aurora", flightlevel: 1 }
  - { name: "Delivery Ops", flightlevel: 3 }
notes:
  - on: "."
    text: "**Draft** for the org review."
  - on: "node:Squad Aurora"
    text: Carries the most WIP — candidate for a bottleneck review
```
