---
description: Start a feature via the Superpowers lifecycle backed by a Beads epic + task graph
argument-hint: [short feature description]
---

You are starting a new feature: **$ARGUMENTS**

**Beads workspace:** if the current directory is inside a code repo that has its own
`.beads` (check `bd status`), use it. Otherwise operate against the home Beads database by
prefixing bd commands with `bd -C "$HOME" ...` (Windows: `bd -C "$env:USERPROFILE" ...`).
Below, `<ws>` means that prefix (omit it when a repo-local `.beads` is present). If both
`bd status` and `bd -C "$HOME" status` fail, tell the user to run `/sb-setup` first.

The plugin's SessionStart hook injects the workflow rules (`WORKFLOW.md`) each session.

## 1. Brainstorm (REQUIRED gate)
Invoke `superpowers:brainstorming` to turn "$ARGUMENTS" into an approved design. Do NOT
create beads or code until the user approves. The skill's own checklist may use TodoWrite.

## 2. Create the tracking epic
After approval:
- In a code repo, also save the spec (`docs/superpowers/specs/...`), no commit unless asked.
- `bd <ws> create --type=epic --title="<feature>" --description="<design summary>"`; record the epic id.

## 3. Write the plan
Invoke `superpowers:writing-plans`. In a repo, save the plan doc; for global work the bead
graph is the plan.

## 4. Explode the plan into a bead graph
For each plan Task N, in order:
- `bd <ws> create --type=task --parent=<epic-id> --title="Task N: <name>" --description="<files, approach, verify>"`
- Order: `bd <ws> dep add <this-task> <previous-task> -t blocks`
- Verify: `bd <ws> dep cycles` (none) and `bd <ws> ready` (Task 1 only).

## 5. Hand off to execution
- `bd ready` → `bd update <id> --claim` → implement (`superpowers:subagent-driven-development`
  or `superpowers:executing-plans`) → `bd close <id> --reason="<verification>"`.
- Discoveries: `bd create --title="..." --deps 'discovered-from:<current-task>'`.
- Progress: `bd epic status`.

## Workspace rules (see WORKFLOW.md)
- Conservative git: never commit/push/`bd dolt push` unless asked.
- Memory: `bd remember "<insight>" --key <slug>`.
- Hybrid tasks: durable = beads; throwaway in-session checklists = TodoWrite.
