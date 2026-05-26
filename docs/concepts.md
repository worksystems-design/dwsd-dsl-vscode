# DWSD DSL — Concepts

> Read this first. It explains *what* boards and flight routes represent — the
> meaning behind the syntax. For *how* to write each construct, see
> [`board-syntax.md`](board-syntax.md) and [`route-syntax.md`](route-syntax.md);
> for runnable minimal files, see [`examples/`](../examples/).

**DWSD** — *Dynamic Worksystems Design* — is a way to describe how an
organization works, in plain text. Two of its building blocks are small
domain-specific languages: a **Board** DSL for kanban-style work systems, and a
**Flight Route** DSL for how a type of work flows through the Flight Levels.
Both are diagram-as-text: you write the structure, the tooling renders it.

## Boards

A **board** is a kanban-style work system — a left-to-right picture of how work
moves from idea to done.

- **Columns** are the workflow stages work passes through (e.g. `Backlog`,
  `Doing`, `Done`). The simplest board is just a list of columns.
- A **lane** is a parallel horizontal swimlane — a way to split the same
  workflow across teams, work types, or classes of service, each with its own
  row of columns.
- A **group** is a named cluster of adjacent columns under one heading (e.g.
  grouping `In Dev` + `Code Review` under `Development`) — structure without a
  full lane.
- A **WIP limit** (`[3]`, `[1..2]`) caps how much work sits in a column at once
  — the core kanban constraint that surfaces bottlenecks.
- A **split** divides one column into parallel sub-strips (e.g. `Develop` /
  `Verify` happening side-by-side inside one stage).
- **Board settings** (flightlevel, timebox, aging, …) and column modifiers
  (aging, returns, policy, refs) tune behaviour and surface explicit working
  agreements.

Reach for a **lane** when the same flow runs in parallel tracks; a **group**
when you only need to cluster a few columns; a **split** when one stage holds
genuinely parallel sub-work.

## Flight Routes

A **flight route** describes how one type of work flows through the **Flight
Levels** — end-to-end, across the systems and teams it touches, rather than
inside a single board.

- **`for:`** names the item type the route is about (e.g. `@feature`).
- **`triggers:`** are the events that start the route (e.g. a new idea, a
  customer request).
- **`path:`** is the sequence of flight levels the work travels through (see
  below).
- **`flow:`** shows the concrete movement between systems/boards, with
  **activity arrows** annotating each transition (`-[generate]->`,
  `-[deliver]`).
- A flow node can reference a board location directly (`@board#column` or a
  typed `@board#lane:Name/col:Name`), tying the route back to the boards above.

## Flight Levels — the model behind routes

**Flight Levels** is a Denk- und Gestaltungsmodell (a thinking-and-design model,
not a framework or method) for making an organization's work — and its
connection from strategy to delivery — visible and improvable. It distinguishes
three levels of altitude:

- **FL1 — Operational.** Where the work is actually delivered: individual teams
  and their work systems.
- **FL2 — Coordination.** Coordinating flow and dependencies across several FL1
  systems toward an end-to-end outcome (a value stream / cross-team level).
- **FL3 — Strategic.** The strategic level: which initiatives and bets the
  organization pursues, connecting strategy down to coordination and operations.

A route's `path:` names the levels the work travels, e.g.
`FL3: Strategy -> FL2: Coordination -> FL1: Delivery`. The point of a route is to
make that strategy-to-delivery connection explicit.

## How boards and routes connect

Boards describe *a* work system at one altitude; routes describe how a type of
work travels *across* systems and levels. A flight route's typed flow nodes can
point at specific board columns, so a high-level route and the concrete boards
that realize it stay linked — strategy connected to operations, in plain text.
