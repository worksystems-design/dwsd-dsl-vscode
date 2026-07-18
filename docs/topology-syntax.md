# Topology DSL — Syntax Reference

The **Topology DSL** describes a Flight-Levels **Work-System Local View**: a
focal Work System and the 1-hop ego-slice of Work Systems around it (the value
chain that contributes up toward it, plus the Work Systems it contributes to).
It is **not** a global org map — it is a scoped, focal slice.

The DSL is plain **YAML**. The fence language id for live-rendering viewers (the
`dwsd-vscode` extension preview, the website) is **`dwsd-topology`**:

````markdown
```dwsd-topology
mode: declared
work-systems:
  - { name: "Squad Aurora" }
```
````

A viewer that recognizes the `dwsd-topology` id renders the block live; in plain
Markdown / GitHub the same example is also shown as a static SVG (see
[Examples](#examples)), so the reference reads correctly in either context.

The parser is **tolerant**: it never throws on author input. Unknown or
malformed fields produce a `topology.*` diagnostic (a warning or info) and the
offending value is clamped to a default or skipped — the rest of the document
still parses. The diagnostic codes are listed per field below.

> **Note on field names that do _not_ exist.** The Topology DSL has **no
> `host` field** and **no `member-of` field**. Membership and hosting are not
> part of the grammar — do not write them. The only relationship the DSL
> expresses between Work Systems is `contributes-to`.

## Design notes — YAML, humans, and machines

The Topology DSL is **YAML by design**. Every example in this reference is also
a valid YAML document — block style (`work-systems:` as an indented list) and
flow style (`{ name: "...", flightlevel: 1 }`) both parse — so any YAML-aware
editor, linter, or formatter can read and write it. The package ships its own
tolerant parser (`src/topology/parser.ts`) that accepts the same shape.

The syntax optimizes for **readability and round-trippability, not brevity**.
Field names are spelled out (`flightlevel`, `contributes-to` — not
abbreviations), values are explicit, and the parser never throws: unknown or
malformed input yields a `topology.*` diagnostic and a sensible default rather
than a hard error. That verbosity is deliberate — the format is meant to be
read and written by humans, but **above all by machines**.

Increasingly, that machine is an AI. The raw DSL is a good target for
generation and repair by coding agents: the YAML shape, the closed and
documented set of fields, and the per-field diagnostics give an agent
everything it needs to author or fix a topology without a UI. Expect authoring
to shift toward AI tooling over time — for example, a Claude Code plugin or
similar agent integration that emits and edits `dwsd-topology` blocks directly.
The live fence preview and the static-SVG variants then serve as the way a
person checks what the machine produced.

## Modes

`mode:` selects how the Local View is assembled. Values: **`infer`**,
**`declared`**, **`mixed`**. The default (when `mode:` is absent) is `infer`.

| Mode | Meaning |
|------|---------|
| `infer` | Derive the Local View from the indexer — the focal Work System and its 1-hop neighbours are read from the surrounding DawnSeed vault. The author lists nothing explicitly. |
| `declared` | The author lists the `work-systems` and `contributes-to` edges explicitly. Nothing is inferred from the indexer. |
| `mixed` | The author's `declared` entries **overlay** the inferred topology as a diff — each Work System or edge carries a `delta` (`add` / `change` / `remove`) that is applied on top of what the indexer found. |

An unrecognized `mode:` value emits **`topology.unknown-mode`** (warning) and
falls back to `infer`.

## Syntax reference

Every field below is implemented by the frozen parser
(`src/topology/parser.ts`) and validated by `src/topology/validate.ts`. The
**Diagnostic** column names the `topology.*` code an author hits on misuse.

### Top-level fields

| Field | Values / shape | Default | Diagnostic on misuse |
|-------|----------------|---------|----------------------|
| `mode` | `infer` \| `declared` \| `mixed` | `infer` | `topology.unknown-mode` (warning; falls back to `infer`) |
| `title` | string | (none — omitted when absent) | (none — a non-string or empty value is silently ignored) |
| `tags` | string list (document-level descriptive chips; a scalar string is accepted as a one-element list) | (none) | tolerant — non-string entries dropped, empties trimmed |
| `focus` | `self` \| `[[Target]]` \| `[[Target\|Alias]]` | (none) | `topology.invalid-focus` (warning, parse); `topology.unresolved-focus` (warning, validate — target is not a known Work System) |
| `radius` | a number \| `inf` \| `{ up, down, lateral }` | `{ up: 1, down: 1, lateral: 1 }` | `topology.unsupported-radius` (unparseable value → defaults to `1`; negative → clamped to `0`; `lateral > 1` flagged at validate) |
| `work-systems` (alias `work_systems`) | list of Work-System objects | `[]` | `topology.invalid-work-system` (a non-object or name-less entry is skipped) |
| `contributes-to` (alias `contributes_to`) | list of edge objects | `[]` | `topology.invalid-edge` (a non-object or `from`/`to`-less entry is skipped) |

**`title`** is an optional human-readable label for the topology as a whole. It
is data-only metadata — used for export filenames and other outside-the-image
uses — and is **not rendered on the diagram**. A non-string or empty value is
silently ignored; when the key is absent the field is **omitted** (never `''`).

**`tags`** are document-level descriptive chips. Note the two scopes: this
top-level `tags:` describes **the topology document**, while the `tags:` on a
`work-systems` entry (below) describes **that Work System**. Topology gained the
document-level one in Phase 27.

`title:` and `tags:` are the **shared document header** — the board and
flight-route DSLs use the same two keys with the same meaning and the same
missing-value convention. Topology did not change here; the other two converged
onto *its* spelling. See [The document header](../document-header.md).

**`focus`** centers the view. `self` focuses the containing Work System (used
in embedded blocks); a `[[wikilink]]` (optionally `[[Target|Alias]]`) focuses a
named Work System. Anything else emits `topology.invalid-focus` and focus is
dropped. At validate time, a wikilink target that is not a known Work System
emits `topology.unresolved-focus`.

**`radius`** bounds the ego-slice. A bare number applies uniformly to `up`,
`down`, and `lateral`. The string `inf` (case-insensitive) means unbounded
(transitive closure). The object form fills any missing key with `1`. Negative
values are clamped to `0` with `topology.unsupported-radius`. Multi-hop `up` /
`down` (and `inf`) are honoured; **`lateral > 1` is not yet supported** and
emits `topology.unsupported-radius` (info) at validate — lateral / `relates-to`
expansion is deferred.

### Work-system fields (each entry of `work-systems`)

| Field | Values / shape | Default | Diagnostic on misuse |
|-------|----------------|---------|----------------------|
| `name` | string (**required**) | — | `topology.invalid-work-system` (entry skipped when `name` is missing or empty) |
| `flightlevel` | `1` \| `2` \| `3` | (none) | `topology.invalid-flightlevel` (warning; the value is omitted) |
| `delta` | `add` \| `change` \| `remove` | (none) | `topology.unknown-delta` (warning; the value is omitted) |
| `tags` | string list (a scalar string is accepted as a one-element list) | (none) | tolerant — non-string entries are dropped, empties trimmed away |
| `anchors` | map of `typeKey → string list` | (none) | `topology.invalid-anchors` (the value is ignored when it is not a map) |

**`name`** is the only required field. **`flightlevel`** places the Work System
in its band (1 = operational, 2 = coordination, 3 = strategy). **`delta`** is
the mixed-mode diff marker. **`tags`** render as colored tag markers in the
Local View. **`anchors`** declares type-keyed entities attached to the Work
System — the key is the entity type (e.g. `agreement`, `meeting`, `individual`,
`unit`, `flight-route`, `interaction`, or a `visualization:board` /
`visualization:route` composite) and the value is a list of entity names. In
the live indexer path these anchors come from the vault; the declared form is
for authoring and the static examples.

### Edge fields (each entry of `contributes-to`)

| Field | Values / shape | Default | Diagnostic on misuse |
|-------|----------------|---------|----------------------|
| `from` | string (**required**) | — | `topology.invalid-edge` (entry skipped when missing) |
| `to` | string (**required**) | — | `topology.invalid-edge` (entry skipped when missing) |
| `delta` | `add` \| `change` \| `remove` | (none) | `topology.unknown-delta` (warning; the value is omitted) |

A `contributes-to` edge points from a contributor to the Work System it feeds.
The value chain flows **upward**: an edge should land on the same or a higher
flight level. An edge to a strictly lower flight level emits
`topology.downward-contribution` (warning) at validate. A repeated `(from, to)`
pair emits `topology.duplicate-edge` (info) and the second occurrence is ignored
at layout time.

## Examples

Each example below appears in **both variants**: the dynamic `dwsd-topology`
fence (the raw DSL a live viewer renders) and the generated static SVG (the
fallback for plain Markdown / GitHub). The examples escalate from the smallest
valid document to the full mixed-mode diff surface.

> For a focused, source-⟷-rendered showcase of just the six examples (generated
> from the single source of truth), see the
> [examples gallery](./topology-examples.md).

### 1. Minimal

The smallest valid topology: a single declared work-system.

```dwsd-topology
# minimal — the smallest valid topology: a single declared work-system.
mode: declared
work-systems:
  - { name: "Squad Aurora", flightlevel: 1 }
```

![Minimal](./topology-examples/minimal.svg)

### 2. Flight Levels

The three Flight Level bands (1 operational, 2 coordination, 3 strategy) across
three work-systems.

```dwsd-topology
# flightlevels — the three Flight Level bands (1 operational, 2 coordination, 3 strategy).
mode: declared
work-systems:
  - { name: "Squad Aurora", flightlevel: 1 }
  - { name: "Reliability Mid", flightlevel: 2 }
  - { name: "Delivery Ops", flightlevel: 3 }
```

![Flight Levels](./topology-examples/flightlevels.svg)

### 3. Contributes-To Network

A contributes-to network: work systems contribute upward to higher-level
targets, and a mid-level system can feed **more than one** target — so the shape
is a network, not a single-rooted tree. Edges point to the same or a higher
flight level (the value chain flows upward).

```dwsd-topology
# contributes-to-network — several sources contribute UP to higher-level targets; one system feeds TWO targets (a network, not a single-rooted tree).
# Edges point same-or-higher flight level (the value chain flows upward).
mode: declared
work-systems:
  - { name: "Squad Aurora", flightlevel: 1 }
  - { name: "Squad Vega", flightlevel: 1 }
  - { name: "Reliability Mid", flightlevel: 2 }
  - { name: "Delivery Ops", flightlevel: 3 }
  - { name: "Platform Strategy", flightlevel: 3 }
contributes-to:
  - { from: "Squad Aurora", to: "Reliability Mid" }
  - { from: "Squad Vega", to: "Reliability Mid" }
  - { from: "Reliability Mid", to: "Delivery Ops" }
  - { from: "Reliability Mid", to: "Platform Strategy" }
```

Here `Reliability Mid` (FL2) contributes to **both** `Delivery Ops` and
`Platform Strategy` (FL3) — the fan-out that makes this a network.

![Contributes-To Network](./topology-examples/contributes-to-network.svg)

### 4. Tags

String tag lists on work-systems, rendered as colored tag markers in the Local
View.

```dwsd-topology
# tags — string lists on work-systems (colored tag markers in the rendered Local View).
mode: declared
work-systems:
  - { name: "Squad Aurora", flightlevel: 1, tags: ["delivery", "core"] }
  - { name: "Delivery Ops", flightlevel: 3, tags: ["strategy"] }
```

![Tags](./topology-examples/tags.svg)

### 5. Focus + Radius

Local-View scoping: `focus:` centers the view on a work-system and `radius:`
bounds how far up/down/lateral the ego-slice reaches.

```dwsd-topology
# focus-radius — Local-View scoping. `focus:` centers the view on a work-system;
# `radius:` bounds how far up/down/lateral the 1-hop ego-slice reaches.
# lateral stays at 1 (lateral > 1 / relates-to expansion is not yet supported).
mode: declared
focus: "[[Reliability Mid]]"
radius: { up: 2, down: 1, lateral: 1 }
work-systems:
  - { name: "Squad Aurora", flightlevel: 1 }
  - { name: "Squad Vega", flightlevel: 1 }
  - { name: "Reliability Mid", flightlevel: 2 }
  - { name: "Delivery Ops", flightlevel: 3 }
contributes-to:
  - { from: "Squad Aurora", to: "Reliability Mid" }
  - { from: "Reliability Mid", to: "Delivery Ops" }
```

![Focus + Radius](./topology-examples/focus-radius.svg)

> **Why "Squad Vega" is not in the rendered view.** It is declared in
> `work-systems` but has no `contributes-to` edge connecting it to the focal
> Work System, so the radius walk never reaches it. Only Work Systems inside
> the focal ego-slice appear in the Local View — declaring a Work System does
> not force it to render if it falls outside `focus` + `radius`.

### 6. Mixed Delta + Anchors

Diff authoring: `mode: mixed` overlays `add`/`change`/`remove` deltas onto the
inferred topology, and `anchors:` declares type-keyed entities attached to a
work-system.

```dwsd-topology
# mixed-delta-anchors — diff authoring. `mode: mixed` overlays declared deltas
# (add/change/remove) onto the inferred topology, and `anchors:` declares the
# type-keyed entities attached to a work-system (agreements, meetings, ...).
mode: mixed
work-systems:
  - name: "Squad Aurora"
    flightlevel: 1
    delta: add
    anchors:
      agreement: ["Team Working Agreement"]
      meeting: ["Weekly Sync"]
  - name: "Reliability Mid"
    flightlevel: 2
    delta: change
  - name: "Legacy Ops"
    flightlevel: 3
    delta: remove
contributes-to:
  - { from: "Squad Aurora", to: "Reliability Mid", delta: add }
```

![Mixed Delta + Anchors](./topology-examples/mixed-delta-anchors.svg)
