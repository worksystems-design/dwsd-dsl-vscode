# DWSD DSL — VS Code

Editor tooling for the three DWSD DSLs — **Flight Route**, **Board**, and
**Topology** — in VS Code: syntax highlighting, validation, and a live diagram
preview, for both Markdown fenced blocks (`dwsd.flightroute` / `dwsd.board` /
`wsd-topology`) and standalone `*.dwsd-flightroute` / `*.dwsd-board` /
`*.dwsd-topology` files.

## What is in this repository

**This is a distribution repository, not the extension's source code.** What you see here is
generated from a private monorepo on every release, so there is no build to run and no
`src/` to read:

| Path | What it is |
|---|---|
| **[Releases](../../releases)** | the `.vsix` — the actual VS Code extension, attached as a release asset |
| `.claude-plugin/` | the Claude Code plugin manifest (this repo doubles as a plugin marketplace) |
| `skills/` | the authoring skill an agent loads — an NL→DSL workflow plus one card per DSL |
| `docs/` | the syntax reference for all three DSLs |
| `examples/` | one minimal, parse-clean file per construct |
| `LICENSE` | MIT |

So there are two things to install, and they are independent: the **extension** (from the
release asset, for the editor) and the **Claude Code plugin** (from this repo as a
marketplace, for agents). Take one, the other, or both.

Issues and pull requests against generated files here will be overwritten by the next
release — the source lives in the private monorepo.

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

## Install the Claude Code plugin

This repo doubles as a **Claude Code plugin marketplace**. The plugin, **`dwsd-dsl`**, teaches
an agent to read and write the three DSLs — it is a skill plus one card per DSL, pointing at
the `docs/` and `examples/` in this same repo, so there is nothing to keep in sync.

In Claude Code, run:

```
/plugin marketplace add worksystems-design/dwsd-dsl-vscode
/plugin install dwsd-dsl@worksystems-design-dwsd-dsl-vscode
```

The first command registers this repo as a marketplace; the second installs the plugin from
it. The long name after the `@` is the *marketplace* name (from `.claude-plugin/marketplace.json`),
not the repo path — they differ on purpose.

The install command opens a details view where you pick the installation scope. Then activate it
in the running session:

```
/reload-plugins
```

From that point an agent can author `*.dwsd-board`, `*.dwsd-flightroute` and `*.dwsd-topology`
files, and the matching Markdown fenced blocks, without being handed the syntax each time.

To pick up a later release:

```
/plugin marketplace update worksystems-design-dwsd-dsl-vscode
```

> **If the `add` step fails on authentication:** GitHub `owner/repo` shorthand clones over SSH
> by default. Either use the full HTTPS URL
> (`/plugin marketplace add https://github.com/worksystems-design/dwsd-dsl-vscode.git`) or set
> `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1`.

**The plugin and the VS Code extension are separate installs.** The plugin gives an agent the
language; the extension gives a human the live preview. Neither requires the other.

## Concepts & syntax reference

- **[`docs/concepts.md`](docs/concepts.md)** — what boards, flight routes, topologies, and the Flight Levels *mean* (start here).
- **[`docs/board-syntax.md`](docs/board-syntax.md)** — every board construct + a minimal example.
- **[`docs/route-syntax.md`](docs/route-syntax.md)** — every route construct + a minimal example.
- **[`docs/topology-syntax.md`](docs/topology-syntax.md)** — the topology (Work-System Local View) syntax reference.
- **[`examples/`](examples/)** — one minimal file per construct.

## License

[MIT](LICENSE) © Thomas Krause.
