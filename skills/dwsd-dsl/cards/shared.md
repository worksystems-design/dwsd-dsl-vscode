# Shared card — concepts, anchoring, locator grammar, document header

> **When prose and code disagree, the code wins.** These facts are distilled from
> the human reference (`docs/reference/`) and the shipped parsers. Fuller depth:
> `docs/concepts.md`, `docs/reference/locator-grammar.md`,
> `docs/reference/document-header.md`.

This is the ONE place the **concepts primer**, the **Inside-Out / Outside-In**
anchoring mechanic, the **locator grammar**, the **annotation entities** that use
it, and the **shared document header** live for agents. The per-DSL cards
(`board.md`, `route.md`, `topology.md`) POINT here — they never restate this story.

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
tissue is the **locator** below; the full roster of entities that may point in is
**The annotation entities**, further down.

---

## The locator grammar

A **locator** is a string that addresses a **position on a board** — a column, a
lane, a group, a split section, and the slot *before* / *in* / *after* it. Parser:
`src/common/parse-locator.ts` (pure, tolerant, no filesystem access).

```
locator := head ('#' boardId)? ('/' path)? ('::' position)?
path    := '.' | '.' '/' segment ('/' segment)* | segment ('/' segment)*
segment := kind ':' name        (kind ∈ lane | group | col | split | stage)
position := 'before' | 'in' | 'after'
```

**Document-root `.`** (XPath/filesystem convention). A leading `.` addresses the
**whole document**: `.` alone = the entire board/route/topology (an empty path);
`./<path>` is a path **from the root**, exactly equivalent to the bare path
(`./col:Doing` ≡ `col:Doing`). `/` stays the path *separator*, so `.` is the free
root token (no YAML quoting needed, though `on: "."` is the convention). Used by
**notes** (below) to anchor at document scope.

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

## The annotation entities — what points *into* a document

The entities that anchor **into** board positions, route stages and topology nodes
are plain Markdown files: **one file = one entity**, its kind decided by the
frontmatter **`type:`** key. The DSL documents never declare them; the entities
declare where they land, and the renderer paints what points at it (outside-in,
above).

Two families matter here beyond the marker types, and until now this card knew
neither.

| `type:` | Anchor field | What it is | Paints |
|---|---|---|---|
| `interaction` · `meeting` | `interaction-of:` (+ `for-domain:`) | A recurring touchpoint | FL-tinted marker in the bottom marker row |
| `agreement` | `agreement-of:` (+ `for-domain:`) | A rule the system agreed on | FL-tinted marker in the bottom marker row |
| `ai-agent` | `interaction-of:` | A non-human participant | FL-tinted marker in the bottom marker row |
| `signal` | **`observes:`** | A selected datum, **before** interpretation | type-colored **location pin**, node top-left |
| `insight` | **`observes:`** | An interpretation drawn from signals | location pin |
| `driver` | **`for-domain:`** | The motive to act — situation · effect · need | location pin |
| `proposal` | **`for-domain:`** | A proposed response, before consent | location pin |
| `organizational-decision-record` | **`for-domain:`** | The decision once taken (ODR) | its own weightier **record marker** |

**`observes:` vs. `for-domain:`.** `observes:` points at *what was seen or
interpreted* — an individual, a unit, a work system, or a board position; it takes
the full value-polymorphic **locator** above. `for-domain:` points **up** at the
**work system (domain)** that has the need — a bare `[[Work System]]` wikilink, never
a board or route position. Getting this backwards is the commonest authoring error
in this family: a `driver` does not observe a column.

### The chain

```
signal  →  insight  →  driver  →  proposal  →  ODR  →  agreement(s)
(data)     (meaning)   (motive)   (response)  (decided)  (operational)
```

The first four are still **in flux**; the ODR is the decision **recorded**; the
`agreement`s carry that decision out. An ODR is pinned at the **domain**
(`for-domain:`), its agreements **where the work happens** (`agreement-of:`) — one
decision up at the system, its consequences down in the flow. That is why the two
anchor fields differ, and why an ODR and an agreement are different rungs rather
than the same rung about different subjects.

> **`organizational-decision-record` is spelled out on purpose.** The
> `organizational` qualifier is a **threshold, not decoration**: plenty gets decided
> about a product, a tool, or a team's own practice, and none of that is an ODR.
> Write one when the decision concerns the organization's **structure, process,
> people, or incentives**. The short label *Decision Records* appears only in the
> preview's drawer and filter, where the context already says which scope is meant.

### `derived-from:` — the shared emergence key

`derived-from:` says **what an entity emerged from**. It is a **shared key on any
entity**, not a per-type field — do not look for it on a type page, and do not omit
it just because a given type's description does not mention it.

- Each step may name its origin: an insight the signal(s) it interprets, a driver the
  insight(s), a proposal the driver, an **ODR** the proposal, and an **`agreement`
  the ODR it operationalizes**.
- That last link is what makes a decision's **bundle** expressible with no bundle
  field: the agreements belonging to one ODR are exactly those whose `derived-from:`
  names it. It also answers *why does this rule apply?* long after the people who
  agreed it have moved on.
- **Optional everywhere**, **single value or list**, both spellings accepted
  (`derived-from` = `derived_from`).
- **Metadata only.** There is no `derived-from` relationship *type*: it is preserved
  and readable, but it resolves to no marker, no pin and no typed edge.

### Worked frontmatter

```markdown
---
type: signal
observes:
  - "[[Emma Design Lead]]"
---
# Schmidt kommt um halb sechs

Schmidt kommt um 17:30 Uhr vorbei.
```

The body of a `signal` is the selected datum, often one line — **no interpretation,
no suggested actions**. The moment meaning is added it has become an `insight`.

```markdown
---
type: driver
for-domain: "[[Strategic Direction]]"
status: new
derived-from:
  - "[[Cross-Team Alignment Gap]]"
---
# Standard-order hand-off has no owner

## Current Situation
## Effect
## Need
## Consequences
```

```markdown
---
type: organizational-decision-record
for-domain: "[[Strategic Direction]]"
status: accepted            # proposed | accepted | superseded | deprecated
date: "2026-05-11"
decision-makers:
  - "[[COO]]"
derived-from: "[[Merge the coordination systems]]"
---
## Context
## Decision
## Consequences
```

The ODR body is the Nygard-ADR minimal shape — **Context · Decision ·
Consequences** — and nothing more is required. `status:` is drawer content only; the
marker looks the same for every value.

### How they group in the preview

The filter menu and the drawer group these by **affordance**, not by where they sit
in the chain: **Change conversation** holds the four still in flux (`signal` ·
`insight` · `driver` · `proposal`, one layer, one drawer band), **Decision records**
holds the ODR. Agreements / Interactions / AI Agents keep their own buckets. Do not
expect a per-type filter row for the in-flux four — there is one row for all of them.

---

## Notes — `notes:` (all three DSLs)

A **note** is an author-written annotation that lives **inside** the document it
annotates — unlike anchors, which arrive from *external* entities. `notes:` is a
**document-level list**; each entry is `{ on, text, resolved? }`. Fuller reference:
`docs/reference/{board,route,topology}/notes.md`.

| Field | Shape | Meaning |
|-------|-------|---------|
| `on` | a **position-only locator** (string) | Where the note anchors. **No `[[file]]#` head** — a note addresses a position in its OWN document (same-doc), never across files. |
| `text` | string | The note body. Supports a **safe minimal-markdown** subset — `**bold**`, `*italic*`, `` `code` `` — and nothing else (**no links**). |
| `resolved` | bool (default `false`) | `true` = done: the marker **recedes** (faded) and the note folds into the drawer's **"Resolved"** subsection. |

- **`on:` per DSL** — board: `col:` / `lane:` / `group:` / `split:` (and a `lane/col`
  cell); route: `stage:<itemType> @ <band>`; topology: `node:<name>` (a bare name or
  `[[node]]` resolve too, including a **purely-inferred** node in `mode: infer`).
- **`on: "."` = the whole document** (doc-root, see the locator grammar above). It
  renders as ONE comment bubble in the document **header**, left of the doc-tag chips.
  **N≥2 doc-notes collapse** to one header bubble with a count badge.
- **Rendering.** A resolving `on:` paints a **comment-bubble marker** at the position
  and adds the note to a shared **"Notes"** drawer bucket (click the marker to open,
  scoped to that unit).
- **Graceful degradation.** An `on:` that no longer resolves is an **orphan**: NO
  canvas marker (never blanks the document), the text is preserved in the drawer's
  **"Orphaned notes"** tray, and an editor **squiggle** lands on the entry
  (`board.note-orphan` / `route.note-orphan` / `topology.note-orphan`, warning).

```dwsd.board
title: Sprint Board
tags: [q3]
notes:
  - on: "."                     # whole-board note → ONE header bubble (left of the tags)
    text: "**Under review** this sprint."
  - on: "col:Doing"             # column note → a bubble on the Doing header
    text: Blocked on infra — escalate today
  - on: "col:Review"
    text: Waiting on the steering decision
    resolved: true
columns:
  - Backlog
  - Doing
  - Review
  - Done
```

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
