# Superpowered Beads

A Claude Code plugin that pairs the **Superpowers** development lifecycle with the
**Beads** (`bd`) issue tracker. Ships `/start-feature`, an always-on workflow injection,
and a one-time `/sb-setup` that prepares a clean home Beads database.

## Prerequisites (one-time)

1. **Claude Code** (desktop app, CLI, or IDE extension).
2. **Git for Windows** — provides the `bash` the SessionStart hook needs. (macOS/Linux: bash is built in.)
3. **Beads (`bd`)** — install per https://github.com/gastownhall/beads (e.g. `winget install` or `go install`). Verify with `bd version`.
4. **Superpowers plugin** — marketplace `obra/superpowers-marketplace`, plugin `superpowers`.

## Install

This repo is **public**, so no GitHub auth is needed. Use whichever path matches your client.

### A. Terminal Claude Code (`/plugin` command or `claude` CLI)
```
claude plugin marketplace add Voskavar/superpowered-beads
claude plugin install superpowered-beads@superpowered-beads
```
(Or `/plugin` → add marketplace `Voskavar/superpowered-beads` → install.)

### B. Desktop app / clients without `/plugin` (settings.json)
Some clients don't expose the `/plugin` command. Add these to your `~/.claude/settings.json`
(Windows: `C:\Users\<you>\.claude\settings.json`), merging into anything already present:
```json
"enabledPlugins": {
  "superpowered-beads@superpowered-beads": true
},
"extraKnownMarketplaces": {
  "superpowered-beads": {
    "source": { "repo": "Voskavar/superpowered-beads", "source": "github" }
  }
}
```
(If a stale/broken `superpowered-beads` entry already exists, remove it first to bust the cache.)

Then **fully restart Claude Code** and run once:
```
/sb-setup
```

## Use

- `/start-feature <description>` — brainstorm → epic → plan → task graph → `bd ready`.
- Track ad-hoc work with `bd create` / `bd ready` / `bd close`; save insights with `bd remember`.
- The workflow rules (`WORKFLOW.md`) and your `bd` memories load automatically each session.

## What's shared vs local

Shared: the workflow + tooling. **Local to each machine:** your Beads database, issues,
and memories — they live in `~/.beads` and are never part of this repo.

## Update / uninstall

```
claude plugin update superpowered-beads
claude plugin uninstall superpowered-beads
```
Uninstalling leaves `~/.beads` intact; your issues and memories remain.

## Notes for maintainers

- The plugin in `marketplace.json` is sourced via a **url object**
  (`{ "source": "url", "url": "https://github.com/Voskavar/superpowered-beads.git" }`), **not**
  `"./"`. The Claude desktop app lists **0 plugins** for a `"./"` source even though the terminal
  CLI accepts it — always use the url-object form.
- The repo must stay **public** (or each user's git must be authenticated) or clients can't fetch it.
