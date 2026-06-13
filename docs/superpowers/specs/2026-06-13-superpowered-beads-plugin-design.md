# Superpowered Beads — Plugin Design Spec

**Date:** 2026-06-13
**Status:** Approved (brainstorming) → ready for writing-plans

> **Correction applied (2026-06-13, post-implementation):** The "contained workspace"
> `~/.superpowered-beads/.beads` was replaced by the **standard home Beads DB `~/.beads`**,
> targeted via `bd -C "$HOME"` (Windows `bd -C "$env:USERPROFILE"`). Reason: `bd` cannot
> create a second isolated embedded Dolt database under the home directory — it walks up,
> finds `~/.beads`, and aborts `init`. Throughout this document, read every
> `~/.superpowered-beads` as `~/.beads` and every `-C <home>/.superpowered-beads` as
> `-C "$HOME"`. `/sb-setup` now reuses an existing `~/.beads` or creates one with
> snapshot-safe cleanup (never deleting a pre-existing `~/.git`). The shipped plugin
> reflects this corrected design.

## Context

A working "Superpowers × Beads" workflow was set up by hand in one user's home
directory (a merged-workflow `CLAUDE.md` section, a `/start-feature` command, a
`SessionStart → bd prime` hook, `bd config no-git-ops`, and a seeded `workflow`
memory). The goal now is to **package that setup so friends can use it out of the
box** — clone/add once and have the same workflow, without hand-editing their config.

The hand setup also taught a hard lesson: `bd init` is invasive — it ran `git init`
on the entire home directory and scattered `AGENTS.md`/`.codex`/`.agents` files. Any
shared installer must avoid inflicting that on friends.

## Goal

A single **Claude Code plugin**, distributed from a **private GitHub repo**, that
delivers the merged Superpowers × Beads workflow non-invasively, with a one-time
visible beads bootstrap. Target audience: friends on **Windows/PowerShell** (design
stays cross-platform where free).

## Decisions (locked)

1. **Shape:** self-contained Claude Code plugin (Approach A), not an `~/.claude`-editing
   install script. Plugins contribute commands + hooks without touching the friend's own
   config files, and update/uninstall cleanly.
2. **Hosting:** private GitHub repo; friends with git access run `claude plugin
   marketplace add`.
3. **Bootstrap visibility:** a visible, idempotent `/sb-setup` command does the beads
   setup — not a silent hook-driven filesystem write (honesty + safety over magic).
4. **Prerequisites, not bundled:** the `bd` binary and the Superpowers plugin remain
   prerequisites; the hook detects them and prints guidance rather than silently
   installing a binary.
5. **Beads is contained:** the global beads workspace lives in a dedicated dir
   (`~/.superpowered-beads/.beads`) so bd's `git init` + agent-file side effects stay
   contained and get cleaned up — never the friend's home root.

## Architecture

The repo *is* the plugin (self-listing marketplace via `source: "./"`).

```
superpowered-beads/                  ← repo == plugin
├─ .claude-plugin/
│  ├─ plugin.json                    manifest: name, version, author, homepage
│  └─ marketplace.json               self-lists this plugin (source "./")
├─ commands/
│  ├─ start-feature.md               lifecycle command: brainstorm→epic→plan→task graph→bd ready
│  └─ sb-setup.md                    one-time, idempotent beads bootstrap (visible)
├─ hooks/
│  ├─ hooks.json                     SessionStart matcher "startup|clear|compact"
│  ├─ run-hook.cmd                   Windows/Unix polyglot shim (pattern reused from Superpowers)
│  └─ session-start                  bash: inject WORKFLOW.md + `bd prime`, else nudge /sb-setup
├─ WORKFLOW.md                       canonical merged rules — SINGLE SOURCE injected by the hook
└─ README.md                         prereqs + 3-command install + uninstall
```

### Components

**`WORKFLOW.md` (single source of the rules).**
The merged-workflow content from the hand setup's CLAUDE.md "Unified Workflow" section:
lifecycle→beads map, the four rules (hybrid task tracking, conservative git, `bd remember`
memory, repo-vs-global doc policy), per-project bootstrap recipe. The hook injects this;
the commands reference it. No separate skill file (YAGNI — hook injection covers
always-present + compaction recovery).

**`hooks/session-start` (the omnipresence + memory mechanism).**
On `startup|clear|compact`, emits `hookSpecificOutput.additionalContext` JSON (mirroring
Superpowers' `session-start` script, including its JSON-escaping and platform-field
selection) containing:
1. `WORKFLOW.md` contents (always-present rules — replaces the role CLAUDE.md played, with
   zero edits to the friend's CLAUDE.md), then
2. output of `bd -C ~/.superpowered-beads prime --hook-json`'s context (command reference +
   their `bd remember` memories).
- If `bd` is missing → inject a short "install bd" notice instead of prime.
- If the beads workspace doesn't exist yet → inject "run `/sb-setup`" nudge instead of prime.
- Reuses `run-hook.cmd` verbatim-in-pattern; requires Git Bash on Windows (a prereq).

**`commands/sb-setup.md` (contained, clean, idempotent bootstrap).**
Steps (all targeting the contained dir, all safe to re-run):
1. Verify `bd` on PATH; if absent, print install guidance and stop.
2. If `~/.superpowered-beads/.beads` absent: `bd -C ~/.superpowered-beads init
   --non-interactive` (prefix e.g. `sb`).
3. Clean bd's side effects inside that dir: remove the contained `.git`, `AGENTS.md`,
   `.agents/`, `.codex/` (reusing the cleanup proven in the hand setup).
4. `bd -C ~/.superpowered-beads config set no-git-ops true`.
5. Seed the `workflow` memory if absent: `bd -C ~/.superpowered-beads remember "<pointer>"
   --key workflow`.
6. Verify: `bd -C ~/.superpowered-beads status` and confirm `bd prime` injects the memory.

**`commands/start-feature.md`.**
The lifecycle command from the hand setup, adjusted to target the contained workspace
(`bd -C ~/.superpowered-beads ...`) for global/home use, and to use a repo-local `.beads`
when run inside a code project.

**`.claude-plugin/plugin.json` + `marketplace.json`.**
Manifest (name `superpowered-beads`, version, author, homepage) and a marketplace that
self-lists the plugin with `source: "./"`, matching the Superpowers reference exactly.

**`README.md`.** Prereqs (Claude Code, Git for Windows, `bd` binary, Superpowers plugin),
the 3-command install, first-run `/sb-setup`, update/uninstall, and the "your issues stay
local" note.

## Friend install flow

Prereqs once: Claude Code · Git for Windows · `bd` (`winget install` / `go install`) ·
Superpowers (`claude plugin marketplace add obra/superpowers-marketplace` + install).
Then:
1. `claude plugin marketplace add <owner>/superpowered-beads`  (private → needs git access)
2. `claude plugin install superpowered-beads@superpowered-beads`
3. Restart Claude Code → run `/sb-setup` once.

Updates: `claude plugin update`. Uninstall: `claude plugin uninstall` (issues/memories in
`~/.superpowered-beads` remain; remove by hand if desired).

## Out of scope / not shared

- The user's own beads DB, issues, and memories (generated locally per machine).
- Auto-installing the `bd` binary or the Superpowers plugin (prereqs with guidance).
- Mac/Linux installers beyond what the polyglot `run-hook.cmd` + bash script already give.

## Error handling

- `bd` missing → hook and `/sb-setup` both detect and print install guidance; hook still
  injects `WORKFLOW.md` so the workflow is usable without beads.
- No Git Bash on Windows → `run-hook.cmd` exits 0 silently (plugin still works, minus
  SessionStart injection); README lists Git for Windows as a prereq.
- Re-running `/sb-setup` → idempotent (checks existence before each mutating step).
- Superpowers not installed → workflow references its skills; README lists it as a prereq
  and the hook can warn if its skills are absent.

## Testing / verification

- `plugin.json` / `marketplace.json` validate; `claude plugin marketplace add <local path>`
  then install succeeds in a clean test profile.
- SessionStart hook returns valid JSON containing WORKFLOW.md text and (post-setup) the
  `workflow` memory — verified with the same `python -m json.tool` check used in the hand
  setup.
- `/sb-setup` on a clean machine creates `~/.superpowered-beads/.beads`, leaves **no**
  `.git`/`AGENTS.md`/`.codex`/`.agents` in that dir or in home, sets `no-git-ops=true`, and
  seeds the memory; re-running is a no-op.
- `/start-feature` triggers brainstorming and creates an epic + child task beads.
- End-to-end dry run in a throwaway Windows user profile (or documented manual checklist).
```
