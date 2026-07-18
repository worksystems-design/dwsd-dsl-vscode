# DWSD DSL — VS Code

Editor tooling for the three DWSD DSLs — **Flight Route**, **Board**, and
**Topology** — in VS Code: syntax highlighting, validation, and a live diagram
preview, for both Markdown fenced blocks (`dwsd.flightroute` / `dwsd.board` /
`wsd-topology`) and standalone `*.dwsd-flightroute` / `*.dwsd-board` /
`*.dwsd-topology` files.

## Why this exists

DWSD — *Dynamic Worksystems Design* — describes how an organization works in plain text.
Three of its building blocks are small domain-specific languages: a **Board** DSL for
kanban-style work systems, a **Flight Route** DSL for how work flows through the flight
levels, and a **Topology** DSL for a focal work system and its 1-hop neighborhood (a
Work-System Local View). This extension makes those DSLs first-class in the editor — you
see a live, lane-aware diagram update as you type, and the syntax reference plus a Claude
Code teaching plugin ship alongside so both humans and AI agents can author the DSLs
without guesswork.

## Install from VSIX

This extension is distributed as a `.vsix` GitHub Release asset (not via the VS Code
Marketplace). To install:

1. Download the latest `dwsd-vscode-<version>.vsix` from this repository's
   **[Releases](../../releases)** page.
2. In VS Code, open the **Extensions** view (`Ctrl+Shift+X` / `Cmd+Shift+X`).
3. Click the **`...`** menu at the top of the Extensions view and choose
   **Install from VSIX…**.
4. Select the downloaded `.vsix` file.

VS Code installs the extension and activates it on the first `*.dwsd-flightroute`,
`*.dwsd-board`, `*.dwsd-topology`, or Markdown file with a DWSD DSL fenced block.

## Claude Code plugin

This repo also ships a thin Claude Code teaching plugin, **`dwsd-dsl`**, that points an
agent at the bundled syntax docs and examples. Add this repo as a plugin marketplace and
install `dwsd-dsl`; the skill references `docs/concepts.md`, `docs/board-syntax.md`,
`docs/route-syntax.md`, `docs/topology-syntax.md`, and `examples/` so there is nothing to
keep in sync.

## Concepts & syntax reference

- **[`docs/concepts.md`](docs/concepts.md)** — what boards, flight routes, topologies, and the Flight Levels *mean* (start here).
- **[`docs/board-syntax.md`](docs/board-syntax.md)** — every board construct + a minimal example.
- **[`docs/route-syntax.md`](docs/route-syntax.md)** — every route construct + a minimal example.
- **[`docs/topology-syntax.md`](docs/topology-syntax.md)** — the topology (Work-System Local View) syntax reference.
- **[`examples/`](examples/)** — one minimal file per construct.

## License

[MIT](LICENSE) © Thomas Krause.
