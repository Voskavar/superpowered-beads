# Superpowers × Beads — Workflow

This session runs the **Superpowers lifecycle** (how to work) on top of **Beads**
(`bd`, the tracking + memory substrate). Invoke Superpowers skills via the Skill tool.

## Lifecycle → Beads map

| Superpowers stage | Beads action |
|---|---|
| brainstorming → design approved | `bd create --type=epic --title="<feature>" --description="<design>"` |
| writing-plans → each Task N | `bd create --type=task --parent=<epic> --title="<task>"`; order with `bd dep add <next> <prev> -t blocks` |
| executing / subagent-driven | `bd ready` → `bd update <id> --claim` → work → `bd close <id> --reason="<verification>"` |
| TDD / debugging discovery | `bd create --title="..." --deps 'discovered-from:<current>'` |
| code review findings | `bd create ...` linked `-t related` / `-t blocks` |
| finishing-a-development-branch | close beads, then STOP before git — report status + proposed commands |

Run `/start-feature <description>` for stages 1–4 (brainstorm → epic → plan → task graph).

## The four rules

1. **Task tracking — HYBRID.** Beads for any work item that outlives the session
   (features, plan tasks, bugs, discoveries, review findings). **TodoWrite only for
   throwaway in-session step checklists** (e.g. a Superpowers skill's own checklist).
   When in doubt, use beads.
2. **Git — CONSERVATIVE.** Never `git commit`/`push` or `bd dolt push` unless the user
   explicitly asks. The beads workspace uses `no-git-ops=true`. At handoff, report
   `git status` + proposed commands and wait.
3. **Memory — `bd remember`.** Durable knowledge via `bd remember "<insight>" --key <slug>`
   (auto-injected each session). Not MEMORY.md, not Claude file-memory. Retrieve with
   `bd memories <keyword>` / `bd recall <key>`.
4. **Spec/plan docs — context-dependent.** Inside a code repo: keep
   `docs/superpowers/{specs,plans}/*.md` (commit only on approval). For global/general
   tasks: skip the docs — the epic + child-task graph IS the plan.

## Beads workspace

This plugin uses a contained global workspace at `~/.superpowered-beads/.beads`
(created by `/sb-setup`). Commands target it with `bd -C <home>/.superpowered-beads ...`
unless you are inside a code repo that has its own `.beads`.

Per-project bootstrap (when starting work in a real repo):
```
bd init --non-interactive
bd config set no-git-ops true
```
Note: `bd init` runs `git init` and writes agent files; in a real repo that's usually
fine, in a scratch dir review/remove the stray git repo afterward.
