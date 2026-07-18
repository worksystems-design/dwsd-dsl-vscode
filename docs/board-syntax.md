# Board DSL — Syntax Reference

Every board construct, with a minimal example. Each example is a complete, parse-clean `*.dwsd-board` file — the same bytes shipped under `examples/board/`.

## board header

The board title via `title:`, plus header keys `flightlevel:` (FL1|FL2|FL3 badge), `timebox:` (deadline chip), and `aging:` (board-level aging chip).

```dwsd.board
title: Delivery Board
flightlevel: FL2
timebox: 2w
aging: 7d

# The board-level header keys render in the board header: `board:` is the title,
# `flightlevel:` a pastel FL badge (FL1 | FL2 | FL3), `timebox:` a deadline
# countdown chip, `aging:` a board-level aging chip (⏳ Nd).
columns:
  - Backlog
  - Doing
  - Done
```

## board settings

Board-scope settings beyond the title. Board-level `tags:` render as a labels legend in the header; live header keys are `flightlevel:`, `timebox:`, `aging:`.

```dwsd.board
title: Strategy Board
flightlevel: FL3

# Board-level settings beyond the title. `tags:` here are board-scope descriptive
# chips (→ AST `labels`), rendered as a labels legend in the header. The
# removed/dormant keys `focus-fit:`, `labels:`, `policy:`, `refs:` are NOT
# authored — the parser silently ignores them; the live keys are shown here.
tags: [strategy, quarterly]
columns:
  - Backlog
  - Doing
  - Done
```

## column

A workflow column — the atomic unit of board flow. Each `columns:` entry is either a bare name string or an object with `name:` plus modifiers.

```dwsd.board
title: Columns
flightlevel: FL2

# A column is the atomic unit of board flow. Each `columns:` entry is either a
# bare string (name-only column) or an object carrying `name:` plus modifiers.
# This minimal example shows both entry forms side by side — just the flow, no
# modifiers.
columns:
  - Backlog
  - name: Doing
  - Done
```

## descriptive tags

Descriptive `tags:` chips on a column, rendered as label chips beside the name. They do NOT tint the header — header tinting is the separate `status-category:` key.

```dwsd.board
title: Descriptive Tags
flightlevel: FL2

# `tags:` are DESCRIPTIVE chips (→ AST `labels`), rendered as label chips beside
# the column name. They do NOT tint the header — header tinting is the separate
# `status-category:` key. The column names here are deliberately neutral
# (no status name-match) so the ONLY visual is the descriptive chips.
columns:
  - Backlog
  - name: Build
    tags: [frontend, urgent]
  - Ship
```

## group

A named cluster of adjacent columns via `groups:`, painted as a band over the columns it names.

```dwsd.board
title: Groups
flightlevel: FL2

# A board-level `groups:` block bands a set of columns under a named group bar.
# Columns-only groups flatten into the board flow (carrying their group path)
# and emit a group sidecar that paints the band over those columns.
groups:
  - name: Development
    columns:
      - In Dev
      - Code Review
  - name: Delivery
    columns:
      - Staging
      - Done
```

## indented column modifier

The same spelled-out modifier keys in YAML block (indented) object form — `name:` on its own line with `aging:` / `max-returns:` indented beneath it.

```dwsd.board
title: Indented Modifiers
flightlevel: FL2

# The same spelled-out modifier keys, written as YAML BLOCK (indented) object
# entries — `name:` on its own line with each modifier key indented beneath it.
# `max-returns:` (→ AST `returns`) renders a `↻N` rework chip; `aging:` renders
# an aging threshold chip. (The legacy `policy:` key was REMOVED — use
# `agreements:`; it is not authored here.)
columns:
  - Backlog
  - name: Doing
    max-returns: 2
    aging: 5d
  - Done
```

## inline column modifier

A modifier attached to a column in YAML flow (inline) object form — `{ name: Doing, aging: 3d }`. Every modifier is a spelled-out key (e.g. `aging:`, `status-category:`).

```dwsd.board
title: Inline Modifiers
flightlevel: FL2

# Every modifier is a spelled-out YAML key (the legacy `Doing aging: 3d` sugar
# is gone). This example uses YAML FLOW (inline) object entries —
# `{ name: …, key: … }` on a single line — to attach modifiers. `aging:` is a
# duration threshold; `status-category:` (closed set) tints the header.
columns:
  - Backlog
  - { name: Doing, aging: 3d }
  - { name: Review, status-category: assess }
  - Done
```

## lane

A named horizontal swimlane via `lanes:`. Each entry carries an optional `type:` (named|assignee|expedite) and a lane-local `columns:` list that replaces the global flow for that lane.

```dwsd.board
title: Swimlanes
flightlevel: FL2

# `lanes:` are swimlanes. Each entry is a bare string (name-only) or an object
# with `name:` + optional `type:` (named | assignee | expedite, default named)
# and a lane-local `columns:` list that replaces the global flow for that lane.
lanes:
  - name: Team Alpha
    type: named
    columns:
      - Backlog
      - Doing
      - Done
  - name: Team Beta
    columns:
      - Backlog
      - Doing
      - Done
```

## split column

Sub-divides a column into sections via the `split:` key: a `direction:` (horizontal|vertical) plus an `of:` list of named sections with optional per-section modifiers.

```dwsd.board
title: Split Columns
flightlevel: FL2

# A `split:` sub-divides a column into sections. `direction:` sets orientation
# (horizontal = side-by-side, vertical = stacked; default vertical). Each `of:`
# entry is a section with a required `name:` plus optional per-section modifiers
# (here `status-category:` tints the section name band).
columns:
  - Backlog
  - name: In Flight
    split:
      direction: horizontal
      of:
        - { name: Develop, status-category: in-progress }
        - { name: Verify, status-category: assess }
  - Done
```

## WIP limit

A work-in-progress cap on a column via the `wip:` key: a single number is the max, a `[min, max]` list renders as a range (`min..max`).

```dwsd.board
title: WIP Limits
flightlevel: FL2

# `wip:` caps work-in-progress per column. A single number is the MAX only
# (renders as a bare count, e.g. `3`); a `[min, max]` list is a range (renders
# `min..max`, e.g. `1..2`). Both forms appear below.
columns:
  - Backlog
  - name: Doing
    wip: 3
  - name: Review
    wip: [1, 2]
  - Done
```
