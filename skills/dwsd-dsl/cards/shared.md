# Shared card — concepts, anchoring, locator grammar, document header

> **When prose and code disagree, the code wins.** These facts are distilled from
> the human reference (`docs/reference/`) and the shipped parsers. Fuller depth:
> `docs/concepts.md`, `docs/reference/locator-grammar.md`,
> `docs/reference/document-header.md`.

This is the ONE place the **concepts primer**, the **Inside-Out / Outside-In**
anchoring mechanic, the **locator grammar**, and the **shared document header**
live for agents. The per-DSL cards (`board.md`, `route.md`, `topology.md`) POINT
here — they never restate this story.

---

## Concepts — what the three DSLs mean

**DWSD** (*Dynamic Worksystems Design*) describes how an organization works, in
plain text. Three building blocks, all diagram-as-text (you write the structure,
the tooling renders it):

- **Board** — a kanban-style work system: the **columns** work flows through,
  the **lanes** (parallel swimlanes) it flows in, the **groups** that band
  columns together, **splits** (parallel sub-strips of one column), and **WIP
  limits** (the core kanban constraint). One work system at one altitude.
- **Flight Route** — how *one type of work* flows through the **Flight Levels**,
  end-to-end across the systems it touches. Authored as an abstract design
  (**bands** → **stages** → glyph-typed **edges**), then optionally **bound** to
  concrete boards.
- **Topology** — a **Work-System Local View**: one focal work system plus the
  1-hop ego-slice around it (what contributes up to it, what it contributes to).
  Scoped and focal, not a global org map. The only edge is `contributes-to`.

**Flight Levels** is a *Denk- und Gestaltungsmodell* (a thinking-and-design
model — not a framework or method) with three altitudes: **FL1 Operational**
(where work is delivered), **FL2 Coordination** (flow across FL1 systems toward
an end-to-end outcome), **FL3 Strategic** (which bets the organization pursues).
Boards, routes, and topologies together give you the inside of a system, the
journey across systems, and the neighborhood around a system — connecting
strategy to operations in plain text.

---

## Inside-Out / Outside-In anchoring

An external entity (a meeting, an interaction, an agreement note) points *into* a
board position, and the board renders a marker there **without the board file
declaring anything**. Discovery is *outside-in*: the entities declare the
positions, the indexer builds a reverse index, and the board renders what points
at it. A board author may **also** declare *internal* `agreements:` /
`interactions:` directly on a column/lane/group (*inside-out*); when both target
the same position, the renderer blends them into one surface. The connective
tissue is the **locator** below.

---

## The locator grammar

A **locator** is a string that addresses a **position on a board** — a column, a
lane, a group, a split section, and the slot *before* / *in* / *after* it. Parser:
`src/common/parse-locator.ts` (pure, tolerant, no filesystem access).

```
locator := head ('#' boardId)? ('/' path)? ('::' position)?
path    := segment ('/' segment)*
segment := kind ':' name        (kind ∈ lane | group | col | split | stage)
position := 'before' | 'in' | 'after'
```

**Head forms** identify the file (resolved by **basename, case-insensitive**):

| Head form | Example | Meaning |
|-----------|---------|---------|
| `[[file]]` (canonical wikilink) | `[[Sprint Board]]` | The file `Sprint Board`. **Canonical form.** |
| `[[file\|label]]` | `[[sprint-board\|Sprint]]` | Only the part **before** `\|` is the file. |
| *(no head)* | `#col:Doing::in` | **Relative** — inherits file/board from the container entity's frontmatter. |

**Typed path segments** drill into structure (chain with `/`; both kind and name
match case-insensitively):

| Segment | Addresses | Example |
|---------|-----------|---------|
| `col:Name` | a column | `col:Done` |
| `lane:Name` | a lane (swimlane) | `lane:Alice/col:Doing` |
| `group:Name` | a group | `group:Delivery/lane:Bob/col:Review` |
| `split:Name` | a split section | `col:Build/split:QA` |
| `stage:Name` | a **route stage** (not a board) — `Name` is the full `itemType @ Band` pair | `stage:epic @ Coordination` |

**Position qualifier** (trailing) selects the slot: `::before`, `::in` (the
default — bare = `in`), `::after`. Peeled from the end before the path is split,
so `col:Review::before` is the column `Review`, not `Review::before`.

**Value-polymorphic keys.** `interaction-of:` / `agreement-of:` accept **either**
a WorkSystem wikilink (`[[Delivery Squad]]` — no `#`/path → a WS anchor) **or** a
board-position locator (has a `#boardId` and/or `kind:` path segment → a chip on
that board position). At a glance: a `#` boardId or any `kind:` segment ⇒
board-position locator; otherwise ⇒ WorkSystem wikilink.

**Graceful degradation.** The locator surface **never crashes and never blanks
the board**. An unresolved locator renders no marker there; a malformed one
degrades to whatever the tolerant parser recovered. *"Show as much as possible,
never blank, never break."*

The route DSL's `bound-to:` stage refinements and route refs reuse this **same**
locator grammar — the `stage:` segment (landed Phase 26) is its one route
extension: `[[Route]]/stage:epic @ Coordination::before`, with `#Route Name`
disambiguating a file that hosts several routes (exactly like `#boardId`). The
route-side marker/drawer delta lives in `docs/reference/route/anchors.md`.

---

## The document header — `title:` + `tags:` (all three DSLs)

All three DSLs carry the same document-level header: an optional **`title:`** and
an optional descriptive **`tags:`** list. Same keys, same meaning, same shape.

```dwsd.board
title: Team Board — Q3
tags: [delivery, squad-aurora]
columns:
  - ToDo
  - Doing
  - Done
```

```dwsd.flightroute
title: Feature Delivery
tags: [strategy, flsa]
for: idee
```

```dwsd.topology
title: Bakery Operations
tags: [ops, pilot]
mode: declared
```

- **`title:`** names the document; it does **not** declare its kind (the file
  extension + fence tag already carry that). The old type-named keys `board:` /
  `route:` are **retired** — a hard error, no alias. Topology always used
  `title:`; the other two converged onto its spelling.
- **Missing values are omitted, never faked.** When `title:` (or `tags:`) is
  absent, the field is **omitted** from the AST — never `''`, never `[]`.
  `'title' in ast` is `false`; `ast.title` is `undefined`.
- **`tags:`** are descriptive chips at **document level** in all three DSLs
  (route + topology gained them in Phase 27). They carry **no** color tint (on
  boards the tint is the separate column-scoped `status-category:` concern). A
  scalar string is accepted as a one-element list. Route tags are
  **document-level only** — no band-level or stage-level tags (*"colour and
  information belong only where they carry meaning"*).

One accessor reads any of the three:
`getDocumentTitle(ast)` / `getDocumentTags(ast)`.
