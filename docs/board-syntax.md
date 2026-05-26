# Board DSL — Syntax Reference

Every board construct, with a minimal example. Each example is a complete, parse-clean `*.dwsd-board` file — the same bytes shipped under `examples/board/`.

## board header

Optional board title and attachment point for board-level settings: `board "Title"` (or bare `board`).

```dwsd.board
board "Delivery Board"
Backlog
Doing
Done
```

## board settings

Board-level modifiers indented under the header: flightlevel, timebox, aging, focus-fit, labels, policy, refs.

```dwsd.board
board "Strategy Board"
  flightlevel: FL2
  timebox: 2w
Backlog
Doing
Done
```

## column

A workflow column. The simplest board is a sequence of column-name lines, one per line.

```dwsd.board
Backlog
Doing
Done
```

## group

A named cluster of adjacent columns under a shared heading: `group Development` with indented columns.

```dwsd.board
group Development
  In Dev
  Code Review
```

## hashtags

Trailing `#tag` markers on a column, lane, or group that surface a visual convention: `Waiting #blocked`.

```dwsd.board
Backlog
Waiting #blocked
Done
```

## indented column modifier

A modifier on an indented line below a column: `aging:`, `returns:`, `check:`, `policy:`, `refs:`.

```dwsd.board
Backlog
Doing
  returns: 2
  policy: "Pull only when capacity is free"
Done
```

## inline column modifier

A modifier on the same line as a column: `aging:`, `returns:`/`max-returns:`, `policy:`, `refs:`.

```dwsd.board
Backlog
Doing aging: 3d
Done
```

## lane

A named horizontal swimlane that groups its own columns: `lane Team Alpha` with indented columns.

```dwsd.board
lane Team Alpha
  Backlog
  Doing
  Done
```

## split column

Splits a column into parallel sub-strips via the `split` keyword, one indented section per strip.

```dwsd.board
In Flight
  split
    Develop
    Verify
```

## WIP limit

A work-in-progress limit on a column via `[N]` (max) or `[min..max]` (range): `Doing [3]`.

```dwsd.board
Backlog
Doing [3]
Review [1..2]
Done
```
