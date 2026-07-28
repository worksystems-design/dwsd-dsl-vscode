---
name: dwsd-dsl
description: >-
  Read and write the DWSD board, flight-route, and topology DSLs. Use whenever
  creating, editing, or reviewing *.dwsd-board / *.dwsd-flightroute /
  *.dwsd-topology files or embedded dwsd.board / dwsd.flightroute / wsd-topology
  fenced blocks.
---

# DWSD DSL — authoring skill

This skill **orchestrates** authoring the three DWSD DSLs (board, flight route,
topology). The *facts* live in the compact per-DSL cards under
`${CLAUDE_PLUGIN_ROOT}/skills/dwsd-dsl/cards/` (and, in fuller depth, in the
bundled human reference under `${CLAUDE_PLUGIN_ROOT}/docs/`). This file tells you
which cards to read and how to turn a natural-language request into valid DSL.

All three DSLs are **plain YAML** with **tolerant parsers**: they never throw.
Malformed input yields a `board.*` / `route.*` / `topology.*` **diagnostic** and a
skip/clamp — the rest of the document still parses. That means "it parsed" is
**not** "it is correct": you must check the diagnostics. The closed, documented key
schema + the per-construct diagnostics→fix tables are what let you author and
**repair** a document without a UI.

## The NL→DSL workflow

1. **Pick the DSL.** A kanban work system → **board**; how a work *type* flows
   across systems/levels → **route**; one focal work system + its 1-hop neighborhood
   → **topology**.

   If the request is *not* a document but something that **points into** one — an
   observation, an interpretation, a need, a proposed response, a recorded decision,
   an agreement, a recurring touchpoint — it is an **annotation entity**: a plain
   Markdown file with a frontmatter `type:`, authored by hand. Read
   `cards/shared.md` § *The annotation entities*. Do not invent a DSL key for it;
   the DSL document declares nothing about it.
2. **Read the cards (reading order).** Always read the shared card first, then the
   matching per-DSL card:
   - authoring a **board** → read `cards/shared.md` + `cards/board.md`
   - authoring a **route** → read `cards/shared.md` + `cards/route.md`
   - authoring a **topology** → read `cards/shared.md` + `cards/topology.md`

   `cards/shared.md` is the single source for the **concepts primer**, the
   **Inside-Out / Outside-In** anchoring mechanic, the **locator grammar** (incl. the
   `.` document-root), the **annotation entities** that anchor into a document
   (`signal` · `insight` · `driver` · `proposal` · `organizational-decision-record`
   alongside `interaction` / `agreement` / `ai-agent`, with `observes:` /
   `for-domain:` / `derived-from:`), the cross-DSL **`notes:`** annotation surface,
   and the **shared `title:` + `tags:` document header**. The per-DSL cards point back
   to it rather than restating it.
3. **Author against the closed schema.** Use only the documented keys and closed
   value sets on the card. Every construct is a spelled-out YAML key — there is no
   hidden inline string-sugar (no `#status`, no `[wip]` bracket). Honor the
   **not-keys** list on each card (e.g. board `policy:`/`refs:` → `agreements:`;
   route `bind:`/`flow:` → `bound-to:`; retired `board:`/`route:` → `title:`).
4. **Validate.** The parser is tolerant, so a clean-looking parse can still carry
   diagnostics. Check for any `*.*` diagnostic on the result; treat every one as a
   real problem to fix (errors block, warnings are teaching nudges but usually mean
   you wrote the wrong construct).
5. **Repair via the diagnostics→fix tables.** Each card carries a diagnostics→fix
   table. Map the emitted code to its fix, apply it, re-validate. Iterate until the
   document is 0-diagnostic (or every remaining diagnostic is a deliberate,
   understood teaching warning).

## Fence tags & files

Standalone files use `.dwsd-board` / `.dwsd-flightroute` / `.dwsd-topology`. In
Markdown, the **preview-renderable** fence is the dot form `dwsd.board` /
`dwsd.flightroute`; topology accepts all four spellings (`wsd-topology` /
`dwsd-topology` / `wsd.topology` / `dwsd.topology`). Use the dot form for a board
or route fence you want the extension preview to render.

## Depth

For the full human reference (every construct, worked examples, the complete
diagnostics families, the rendered galleries), read the bundled docs under
`${CLAUDE_PLUGIN_ROOT}/docs/` — start at `docs/concepts.md` for the *why*, then the
per-DSL reference. **When prose and code disagree, the code wins** — the shipped
parser is always the authority.
