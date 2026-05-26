---
name: dwsd-dsl
description: >-
  Read and write the DWSD board and flight-route DSLs. Use whenever creating,
  editing, or reviewing *.dwsd-board / *.dwsd-flightroute files or embedded
  dwsd.board / dwsd.flightroute fenced blocks.
---

# DWSD DSL

This skill is a thin pointer; the full reference ships with this plugin. Before
authoring board or route DSL, read:

- **`${CLAUDE_PLUGIN_ROOT}/docs/concepts.md`** — what boards, flight routes, and the Flight Levels *mean* (read first for the why).
- **`${CLAUDE_PLUGIN_ROOT}/docs/board-syntax.md`** — every board construct + a minimal example.
- **`${CLAUDE_PLUGIN_ROOT}/docs/route-syntax.md`** — every route construct + a minimal example.
- **`${CLAUDE_PLUGIN_ROOT}/examples/`** — one minimal file per construct.

When in doubt, the bundled docs win — this skill never overrides them.
