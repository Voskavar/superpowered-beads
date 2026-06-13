# Superpowered Beads

A Claude Code plugin that pairs the **Superpowers** development lifecycle with the
**Beads** (`bd`) issue tracker. Ships `/start-feature`, an always-on workflow injection,
and a one-time `/sb-setup` that prepares a clean home Beads database.

## Prerequisites (one-time)

1. **Claude Code** (desktop, CLI, or IDE).
2. **Git for Windows** — provides the `bash` the SessionStart hook needs. (macOS/Linux: bash is built in.)
3. **Beads (`bd`)** — install per https://github.com/gastownhall/beads (e.g. `winget install` or `go install`). Verify with `bd version`.
4. **Superpowers plugin:**
   ```
   claude plugin marketplace add obra/superpowers-marketplace
   claude plugin install superpowers@superpowers-dev
   ```

## Install

```
claude plugin marketplace add Voskavar/superpowered-beads
claude plugin install superpowered-beads@superpowered-beads
```
(Private repo: you need git access to `Voskavar/superpowered-beads`.)

Then **restart Claude Code** and run once:
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
