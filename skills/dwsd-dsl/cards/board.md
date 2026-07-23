# Board card — `dwsd.board` / `.dwsd-board`

> **When prose and code disagree, the code wins.** Facts from `src/board/parser.ts`
> + `src/board/types.ts`. Fuller human reference: `docs/reference/board/index.md`.
> For anchors, the locator grammar, and the shared `title:` + `tags:` header, see
> **`shared.md`** — this card does not restate them.

A **board** is a Flight-Levels kanban work system. Plain **YAML**. The
preview-renderable Markdown fence is the dot form **`dwsd.board`** (the language
id for standalone `.dwsd-board` files is `dwsd-board`). The parser is **tolerant**:
it never throws — malformed input yields a `board.*` diagnostic and a clamp/skip,
the rest of the document still parses. **There is no inline string-sugar** (no
`#status`, no `[wip]` bracket) — every modifier is a spelled-out YAML key.

## Header + structure keys (closed schema)

| Key | Values / shape | Diagnostic on misuse |
|-----|----------------|----------------------|
| `title` | string — the board name (shared header; see `shared.md`) | (none — non-string/empty silently ignored) |
| `flightlevel` | **`FL1` \| `FL2` \| `FL3`** (closed set) | `board.invalid-flightlevel` (warn; omitted) |
| `timebox` | string (e.g. `5d`, `12w`) | (none) |
| `aging` | string (board-level aging, e.g. `3d`) | (none) |
| `tags` | string list — descriptive chips (shared header) | tolerant — non-strings dropped |
| `agreements` | string OR string list — working agreements (`[[wikilink]]` = linked entity; plain string = inline policy) | tolerant |
| `interactions` | string OR string list — meeting / interaction pointers | tolerant |
| `columns` | list of column entries (bare string OR object with `name` + modifiers) | `board.invalid-column` |
| `lanes` | list of lane entries | `board.invalid-lane` |
| `groups` | list of board-level group blocks (nesting capped at depth 3) | `board.invalid-group` / `board.nested-too-deep` |
| `notes` | list of `{ on, text, resolved? }` — inline annotations; `on:` is a position-only locator (`col:`/`lane:`/`group:`/`split:`, or `.` = whole board). See **`shared.md` › Notes** | `board.note-orphan` (warn) |

**Column modifiers** (on an object column entry): `name`, `wip` (a number or a
`[min, max]` pair → `board.invalid-wip`), `aging`, `max-returns`,
`status-category`, `tags`, `split`.

**Lane** entry: `name` + optional `type` (**`named` \| `assignee` \| `expedite`**,
default `named` → `board.unknown-lane-type`) + a lane-local `columns` list.

## Closed value sets

- **`flightlevel`**: `FL1` · `FL2` · `FL3`.
- **`status-category`** (column tint): **`to-do` · `in-progress` · `done` ·
  `waiting` · `assess`** — anything else → `board.invalid-status-category`
  (dropped, no tint). Tint is a `jira-default`-theme feature only.
- **lane `type`**: `named` · `assignee` · `expedite`.

## Diagnostics → fix (`board.*`)

| Code | Severity | Fix |
|------|----------|-----|
| `board.syntax-error` | error | Fix the YAML (an executable `!!`-tag is rejected, never run). |
| `board.retired-key` | **error** | Rename root `board:` → `title:`. (A nested `board:` in a route binding is legitimate.) |
| `board.invalid-flightlevel` | warn | Use `FL1`/`FL2`/`FL3`. |
| `board.invalid-status-category` | warn | Use the closed status set above. |
| `board.invalid-wip` | warn | `wip:` must be a number or `[min, max]`. |
| `board.unknown-lane-type` | warn | lane `type:` ∈ `named`/`assignee`/`expedite`. |
| `board.invalid-column` / `-lane` / `-split` / `-group` | warn | Malformed entry skipped — fix its shape. |
| `board.nested-too-deep` | — | Groups nest at most 3 deep; flatten deeper levels. |

## Not keys — do NOT write them

- **`policy:` / `refs:`** → the single **`agreements:`** key (linked entities +
  inline rules both live there).
- **`labels:`** → descriptive chips are **`tags:`**.
- **`focus-fit:`** (alias `focus_fit`) — the header carries no FIT-type label.
- **`statuses:`** — no Jira-status key exists.
- **`board:`** — **retired**; the title is `title:` (hard `board.retired-key`).

## Parse-clean examples

```dwsd.board
# Header keys: flightlevel badge (FL1|FL2|FL3), timebox countdown, board aging.
title: Delivery Board
flightlevel: FL2
timebox: 2w
aging: 7d
columns:
  - Backlog
  - Doing
  - Done
```

```dwsd.board
# Descriptive `tags:` on a column render as label chips (no tint).
title: Descriptive Tags
flightlevel: FL2
columns:
  - Backlog
  - name: Build
    tags: [frontend, urgent]
  - Ship
```

```dwsd.board
# `agreements:` folds former refs: + policy: into one bucket per scope —
# a `[[wikilink]]` links an entity, a plain string is inline policy.
title: Working Agreements
flightlevel: FL2
columns:
  - ToDo
  - name: Doing
    agreements:
      - "[[example-wip-agreement]]"
      - "Pull a new card only when this column is below its WIP cap."
  - Done
agreements:
  - "[[example-definition-of-ready]]"
  - "[[example-definition-of-done]]"
  - "Daily standup at 09:30. Keep the board honest; respect the WIP caps."
```

```dwsd.board
# `notes:` — inline annotations. `on: "."` is a whole-board note (header bubble);
# `on: col:…` anchors a column; `resolved: true` recedes + folds into "Resolved".
title: Notes
tags: [q3]
notes:
  - on: "."
    text: "**Under review** this sprint."
  - on: "col:Doing"
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
