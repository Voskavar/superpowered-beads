# Superpowered Beads Plugin — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.
>
> **Beads integration:** At execution start, create a tracking epic and one child bead per Task in this repo's own `.beads` (created in Task 1). Map: `bd create --type=epic`, then `bd create --type=task --parent=<epic>` per task, ordered with `bd dep add <next> <prev> -t blocks`. Conservative-git still applies EXCEPT this repo is the explicit deliverable, so its own `git commit`s (the "Commit" steps below) are authorized; pushing to GitHub is done in Task 7 with the user.

> **Correction applied (2026-06-13, post-implementation):** Tasks 2–5 originally used a
> contained `~/.superpowered-beads` workspace. That was replaced during the build by the
> **standard home Beads DB `~/.beads`**, targeted via `bd -C "$HOME"`, because `bd` cannot
> isolate a second embedded Dolt DB under home (it finds `~/.beads` and aborts `init`). Read
> every `~/.superpowered-beads` below as `~/.beads` and `-C <home>/.superpowered-beads` as
> `-C "$HOME"`. `/sb-setup` reuses an existing `~/.beads` or creates one with snapshot-safe
> cleanup. The committed plugin reflects the corrected design (commit `619031f`).

**Goal:** Package the Superpowers × Beads workflow as a single, non-invasive Claude Code plugin in a private GitHub repo that friends add, install, and finish with one `/sb-setup`.

**Architecture:** The repo *is* the plugin (self-listing marketplace, `source: "./"`). A SessionStart hook injects `WORKFLOW.md` + `bd prime` every session (the non-invasive replacement for editing CLAUDE.md). `/sb-setup` does a contained, idempotent beads bootstrap in `~/.superpowered-beads`. `/start-feature` runs the lifecycle. `bd` + Superpowers are guided prerequisites.

**Tech Stack:** Claude Code plugin (JSON manifests, command markdown, JSON hooks), a Windows/Unix polyglot `.cmd` shim + bash `session-start` script, Beads (`bd`) CLI.

**Reference files (read before starting):**
- Spec: `docs/superpowers/specs/2026-06-13-superpowered-beads-plugin-design.md`
- Proven plugin pattern: `C:\Users\jeans\.claude\plugins\cache\superpowers-marketplace\superpowers\5.1.0\` — `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `hooks/hooks.json`, `hooks/run-hook.cmd`, `hooks/session-start`.
- Source content to port: `C:\Users\jeans\CLAUDE.md` ("Unified Workflow" section) and `C:\Users\jeans\.claude\commands\start-feature.md`.

**Repo root for all paths below:** `C:\Users\jeans\superpowered-beads\`

---

## File Structure

| File | Responsibility |
|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest (name, version, author) |
| `.claude-plugin/marketplace.json` | Self-listing marketplace so `marketplace add` works |
| `WORKFLOW.md` | Single source of the merged rules; injected by the hook |
| `hooks/hooks.json` | Registers the SessionStart hook |
| `hooks/run-hook.cmd` | Windows/Unix polyglot shim that locates bash |
| `hooks/session-start` | Bash: build + emit the `additionalContext` JSON |
| `commands/sb-setup.md` | One-time, idempotent contained beads bootstrap |
| `commands/start-feature.md` | Lifecycle command (workspace-aware) |
| `README.md` | Prereqs, install, uninstall, scope |
| `.gitignore` | Ignore local cruft |

---

## Task 1: Repo scaffold, git init, manifests, beads tracking

**Files:**
- Create: `.gitignore`, `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`

> **User config:** Replace `OWNER`, `Your Name`, `you@example.com` in both JSON files with the user's GitHub username and identity. These are required real values, not placeholders to leave blank — confirm them with the user before committing.

- [ ] **Step 1: Initialize the repo and beads tracking**

```powershell
cd C:\Users\jeans\superpowered-beads
git init
bd init --prefix sb --non-interactive   # repo-local .beads for tracking THIS build
bd config set no-git-ops true
```
Then create the epic + one task bead per Task in this plan (see header). Claim Task 1.

- [ ] **Step 2: Write `.gitignore`**

```gitignore
# OS / editor
Thumbs.db
.DS_Store
*.swp

# Beads working data (issues sync via Dolt, not committed here)
.beads/embeddeddolt/
```

- [ ] **Step 3: Write `.claude-plugin/plugin.json`**

```json
{
  "name": "superpowered-beads",
  "description": "Superpowers lifecycle on top of Beads issue tracking: /start-feature, always-on workflow injection, and one-time bd bootstrap.",
  "version": "0.1.0",
  "author": { "name": "Your Name", "email": "you@example.com" },
  "homepage": "https://github.com/OWNER/superpowered-beads",
  "repository": "https://github.com/OWNER/superpowered-beads",
  "license": "MIT",
  "keywords": ["beads", "superpowers", "workflow", "issue-tracking"]
}
```

- [ ] **Step 4: Write `.claude-plugin/marketplace.json`**

```json
{
  "name": "superpowered-beads",
  "description": "Superpowers × Beads unified workflow plugin",
  "owner": { "name": "Your Name", "email": "you@example.com" },
  "plugins": [
    {
      "name": "superpowered-beads",
      "description": "Superpowers lifecycle on top of Beads issue tracking",
      "version": "0.1.0",
      "source": "./",
      "author": { "name": "Your Name", "email": "you@example.com" }
    }
  ]
}
```

- [ ] **Step 5: Validate JSON**

```powershell
Get-Content .claude-plugin\plugin.json -Raw | python -m json.tool > $null; if ($?) { "plugin.json OK" }
Get-Content .claude-plugin\marketplace.json -Raw | python -m json.tool > $null; if ($?) { "marketplace.json OK" }
```
Expected: both print "OK".

- [ ] **Step 6: Commit**

```powershell
git add .gitignore .claude-plugin
git commit -m "scaffold: plugin manifest and self-listing marketplace"
```

---

## Task 2: WORKFLOW.md (the injected rules)

**Files:**
- Create: `WORKFLOW.md`

Port the "Unified Workflow: Superpowers × Beads" section from `C:\Users\jeans\CLAUDE.md`, reworded so it is plugin-context (not "this home dir") and references the contained workspace.

- [ ] **Step 1: Write `WORKFLOW.md`**

````markdown
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
````

- [ ] **Step 2: Verify content**

```powershell
Select-String -Path WORKFLOW.md -Pattern "HYBRID","CONSERVATIVE","bd remember","/start-feature" | Measure-Object | % Count
```
Expected: 4 (all anchors present).

- [ ] **Step 3: Commit**

```powershell
git add WORKFLOW.md
git commit -m "content: canonical merged workflow rules"
```

---

## Task 3: SessionStart hook (injection mechanism)

**Files:**
- Create: `hooks/hooks.json`, `hooks/run-hook.cmd`, `hooks/session-start`

- [ ] **Step 1: Write `hooks/hooks.json`**

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 2: Write `hooks/run-hook.cmd`** (Windows/Unix polyglot shim — reproduce the Superpowers pattern verbatim)

```bat
: << 'CMDBLOCK'
@echo off
REM Cross-platform polyglot wrapper for hook scripts.
REM On Windows: cmd.exe runs the batch portion, which finds and calls bash.
REM On Unix: the shell interprets this as a script (: is a no-op in bash).
REM Usage: run-hook.cmd <script-name> [args...]

if "%~1"=="" (
    echo run-hook.cmd: missing script name >&2
    exit /b 1
)

set "HOOK_DIR=%~dp0"

if exist "C:\Program Files\Git\bin\bash.exe" (
    "C:\Program Files\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)
if exist "C:\Program Files (x86)\Git\bin\bash.exe" (
    "C:\Program Files (x86)\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)

where bash >nul 2>nul
if %ERRORLEVEL% equ 0 (
    bash "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)

REM No bash found - exit silently rather than error
exit /b 0
CMDBLOCK

# Unix: run the named script directly
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPT_NAME="$1"
shift
exec bash "${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

- [ ] **Step 3: Write `hooks/session-start`** (bash; injects WORKFLOW.md + bd prime, handles bd-missing and Windows home path)

```bash
#!/usr/bin/env bash
# SessionStart hook for superpowered-beads
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"

# Resolve home for the bd Windows binary: prefer Windows %USERPROFILE%, normalize slashes.
HOME_DIR="${USERPROFILE:-$HOME}"
HOME_DIR="${HOME_DIR//\\//}"
BEADS_WS="${HOME_DIR}/.superpowered-beads"

# 1. Always-present workflow rules
workflow_content=$(cat "${PLUGIN_ROOT}/WORKFLOW.md" 2>/dev/null || echo "Error reading WORKFLOW.md")

# 2. Beads context, or guidance if not ready
if ! command -v bd >/dev/null 2>&1; then
  beads_block=$'\n\n[superpowered-beads] The `bd` binary is not on PATH. Install Beads, then run /sb-setup. The workflow rules above still apply.'
elif [ ! -d "${BEADS_WS}/.beads" ]; then
  beads_block=$'\n\n[superpowered-beads] Beads workspace not initialized. Run /sb-setup once to create it.'
else
  prime_out=$(bd -C "${BEADS_WS}" prime --full 2>/dev/null || echo "")
  beads_block=$'\n\n'"${prime_out}"
fi

context="${workflow_content}${beads_block}"

# JSON-escape (same approach as the Superpowers session-start script)
escape_for_json() {
  local s="$1"
  s="${s//\\/\\\\}"
  s="${s//\"/\\\"}"
  s="${s//$'\n'/\\n}"
  s="${s//$'\r'/\\r}"
  s="${s//$'\t'/\\t}"
  printf '%s' "$s"
}
context_escaped=$(escape_for_json "$context")

if [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -z "${COPILOT_CLI:-}" ]; then
  printf '{\n  "hookSpecificOutput": {\n    "hookEventName": "SessionStart",\n    "additionalContext": "%s"\n  }\n}\n' "$context_escaped"
else
  printf '{\n  "additionalContext": "%s"\n}\n' "$context_escaped"
fi
exit 0
```

- [ ] **Step 4: Validate hooks.json**

```powershell
Get-Content hooks\hooks.json -Raw | python -m json.tool > $null; if ($?) { "hooks.json OK" }
```
Expected: "hooks.json OK".

- [ ] **Step 5: Smoke-test the hook emits valid JSON containing the rules**

```powershell
$env:CLAUDE_PLUGIN_ROOT = (Resolve-Path .).Path
& "C:\Program Files\Git\bin\bash.exe" hooks/session-start | python -c "import sys,json; d=json.load(sys.stdin); c=d['hookSpecificOutput']['additionalContext']; print('JSON OK; has rules:', 'Superpowers' in c)"
```
Expected: `JSON OK; has rules: True` (beads block will say "Run /sb-setup" until Task 4 setup is run — that's correct).

- [ ] **Step 6: Commit**

```powershell
git add hooks
git commit -m "hooks: SessionStart injection of workflow + bd prime"
```

---

## Task 4: `/sb-setup` command (contained, idempotent bootstrap)

**Files:**
- Create: `commands/sb-setup.md`

- [ ] **Step 1: Write `commands/sb-setup.md`**

````markdown
---
description: One-time, idempotent setup of the contained global Beads workspace for superpowered-beads
---

Set up (or repair) the contained global Beads workspace this plugin uses. Idempotent —
safe to re-run. Workspace: `<HOME>/.superpowered-beads` (Windows `C:\Users\<you>\.superpowered-beads`,
macOS/Linux `~/.superpowered-beads`). On Windows use `$env:USERPROFILE`; on Unix use `$HOME`.

Do these in order and report what changed:

1. **Check bd.** Run `bd version`. If it fails, tell the user to install Beads
   (https://github.com/gastownhall/beads) and STOP.

2. **Init if missing.** If `<HOME>/.superpowered-beads/.beads` does not exist:
   - PowerShell: `bd -C "$env:USERPROFILE\.superpowered-beads" init --prefix sb --non-interactive`
   - bash: `bd -C "$HOME/.superpowered-beads" init --prefix sb --non-interactive`

3. **Contain bd's side effects.** `bd init` creates a git repo and agent files in that
   folder; remove them (not needed; no-git-ops is used). Delete if present inside
   `<HOME>/.superpowered-beads/`: `.git`, `AGENTS.md`, `.agents`, `.codex`.
   - PowerShell: `Remove-Item -Recurse -Force "$env:USERPROFILE\.superpowered-beads\.git","$env:USERPROFILE\.superpowered-beads\AGENTS.md","$env:USERPROFILE\.superpowered-beads\.agents","$env:USERPROFILE\.superpowered-beads\.codex" -ErrorAction SilentlyContinue`
   - bash: `rm -rf "$HOME/.superpowered-beads/.git" "$HOME/.superpowered-beads/AGENTS.md" "$HOME/.superpowered-beads/.agents" "$HOME/.superpowered-beads/.codex"`

4. **Disable git ops.** `bd -C "<HOME>/.superpowered-beads" config set no-git-ops true`

5. **Seed the workflow memory (if absent).** Check
   `bd -C "<HOME>/.superpowered-beads" memories` for key `workflow`. If absent, run:
   `bd -C "<HOME>/.superpowered-beads" remember "WORKFLOW: Superpowers lifecycle on top of Beads. Hybrid tasks (beads for durable work, TodoWrite only for throwaway in-session checklists), conservative git (never commit/push unless asked), durable memory via bd remember, /start-feature to begin. See the plugin's WORKFLOW.md." --key workflow`

6. **Verify and report.** Run `bd -C "<HOME>/.superpowered-beads" status` (succeeds) and
   `bd -C "<HOME>/.superpowered-beads" config get no-git-ops` (true). Confirm no
   `.git`/`AGENTS.md`/`.codex`/`.agents` remain in the workspace folder. Report success.
````

- [ ] **Step 2: Manually run the setup once (this machine already has it; verify idempotency on the contained path)**

```powershell
bd -C "$env:USERPROFILE\.superpowered-beads" init --prefix sb --non-interactive
Remove-Item -Recurse -Force "$env:USERPROFILE\.superpowered-beads\.git","$env:USERPROFILE\.superpowered-beads\AGENTS.md","$env:USERPROFILE\.superpowered-beads\.agents","$env:USERPROFILE\.superpowered-beads\.codex" -ErrorAction SilentlyContinue
bd -C "$env:USERPROFILE\.superpowered-beads" config set no-git-ops true
bd -C "$env:USERPROFILE\.superpowered-beads" config get no-git-ops
```
Expected: ends with `true`; the workspace folder contains `.beads` and none of `.git`/`AGENTS.md`/`.codex`/`.agents`.

- [ ] **Step 3: Re-run the hook smoke test (now prime should appear)**

```powershell
$env:CLAUDE_PLUGIN_ROOT = (Resolve-Path .).Path
& "C:\Program Files\Git\bin\bash.exe" hooks/session-start | python -c "import sys,json; d=json.load(sys.stdin); c=d['hookSpecificOutput']['additionalContext']; print('has prime:', 'Persistent Memories' in c or 'Essential' in c or 'bd ready' in c)"
```
Expected: `has prime: True`.

- [ ] **Step 4: Commit**

```powershell
git add commands/sb-setup.md
git commit -m "command: idempotent contained beads bootstrap (/sb-setup)"
```

---

## Task 5: `/start-feature` command (workspace-aware)

**Files:**
- Create: `commands/start-feature.md`

Port `C:\Users\jeans\.claude\commands\start-feature.md`, adding workspace targeting.

- [ ] **Step 1: Write `commands/start-feature.md`** — copy the existing file's body, prepend this workspace note, and replace bare `bd` calls in the body with workspace-aware guidance:

````markdown
---
description: Start a feature via the Superpowers lifecycle backed by a Beads epic + task graph
argument-hint: [short feature description]
---

You are starting a new feature: **$ARGUMENTS**

**Beads workspace:** if the current directory is inside a code repo that has its own
`.beads` (check `bd status`), use it. Otherwise operate against the contained global
workspace by prefixing bd commands with `-C "<HOME>/.superpowered-beads"` (Windows:
`$env:USERPROFILE\.superpowered-beads`). If `bd status` fails, tell the user to run
`/sb-setup` first.

If the workspace rules aren't loaded this session, the plugin's SessionStart hook injects
them; you can also read `WORKFLOW.md` from the plugin.

## 1. Brainstorm (REQUIRED gate)
Invoke `superpowers:brainstorming` to turn "$ARGUMENTS" into an approved design. Do NOT
create beads or code until the user approves. The skill's own checklist may use TodoWrite.

## 2. Create the tracking epic
After approval:
- In a code repo, also save the spec (`docs/superpowers/specs/...`), no commit unless asked.
- `bd [-C <workspace>] create --type=epic --title="<feature>" --description="<design summary>"`; record the epic id.

## 3. Write the plan
Invoke `superpowers:writing-plans`. In a repo, save the plan doc; for global work the bead
graph is the plan.

## 4. Explode the plan into a bead graph
For each plan Task N, in order:
- `bd [-C <workspace>] create --type=task --parent=<epic-id> --title="Task N: <name>" --description="<files, approach, verify>"`
- Order: `bd [-C <workspace>] dep add <this-task> <previous-task> -t blocks`
- Verify: `bd [-C <workspace>] dep cycles` (none) and `bd [-C <workspace>] ready` (Task 1 only).

## 5. Hand off to execution
- `bd ready` → `bd update <id> --claim` → implement (`superpowers:subagent-driven-development`
  or `superpowers:executing-plans`) → `bd close <id> --reason="<verification>"`.
- Discoveries: `bd create --title="..." --deps 'discovered-from:<current-task>'`.
- Progress: `bd epic status`.

## Workspace rules (see WORKFLOW.md)
- Conservative git: never commit/push/`bd dolt push` unless asked.
- Memory: `bd remember "<insight>" --key <slug>`.
- Hybrid tasks: durable = beads; throwaway in-session checklists = TodoWrite.
````

- [ ] **Step 2: Verify**

```powershell
Select-String -Path commands\start-feature.md -Pattern "brainstorming","--type=epic","-t blocks","sb-setup" | Measure-Object | % Count
```
Expected: ≥4.

- [ ] **Step 3: Commit**

```powershell
git add commands/start-feature.md
git commit -m "command: workspace-aware /start-feature"
```

---

## Task 6: README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write `README.md`** (replace `OWNER` with the user's GitHub username)

````markdown
# Superpowered Beads

A Claude Code plugin that pairs the **Superpowers** development lifecycle with the
**Beads** (`bd`) issue tracker. Ships `/start-feature`, an always-on workflow injection,
and a one-time `/sb-setup` that creates a clean, contained Beads workspace.

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
claude plugin marketplace add OWNER/superpowered-beads
claude plugin install superpowered-beads@superpowered-beads
```
(Private repo: you need git access to `OWNER/superpowered-beads`.)

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
and memories — they live in `~/.superpowered-beads` and are never part of this repo.

## Update / uninstall

```
claude plugin update superpowered-beads
claude plugin uninstall superpowered-beads
```
Uninstalling leaves `~/.superpowered-beads` intact; delete it by hand to remove your local issues.
````

- [ ] **Step 2: Commit**

```powershell
git add README.md
git commit -m "docs: README with prereqs, install, scope"
```

---

## Task 7: End-to-end verification + publish

**Files:** none (verification + remote)

- [ ] **Step 1: Local plugin-load test in a clean check**

```powershell
claude plugin marketplace add "C:\Users\jeans\superpowered-beads"
claude plugin install superpowered-beads@superpowered-beads
```
Expected: install succeeds; `/start-feature` and `/sb-setup` appear as commands. (If testing in the live session isn't possible, document the manual check.)

- [ ] **Step 2: Confirm full hook output post-setup**

```powershell
$env:CLAUDE_PLUGIN_ROOT = "C:\Users\jeans\superpowered-beads"
& "C:\Program Files\Git\bin\bash.exe" "C:\Users\jeans\superpowered-beads\hooks\session-start" | python -c "import sys,json; d=json.load(sys.stdin); c=d['hookSpecificOutput']['additionalContext']; print('rules+prime:', ('Superpowers' in c) and ('bd ready' in c or 'Persistent Memories' in c))"
```
Expected: `rules+prime: True`.

- [ ] **Step 3: Create the private GitHub repo and push (with the user)**

```powershell
gh repo create OWNER/superpowered-beads --private --source . --remote origin --push
```
Or, if the repo exists: `git remote add origin git@github.com:OWNER/superpowered-beads.git; git push -u origin main`.
Confirm the user wants to push before running (this is the one outward action).

- [ ] **Step 4: Verify a friend's path (documented dry-run)**

On a second machine / clean profile: install prereqs → `claude plugin marketplace add OWNER/superpowered-beads` → install → restart → `/sb-setup` → open a session and confirm the workflow rules appear and `/start-feature` works. Record results.

- [ ] **Step 5: Close the build epic**

```powershell
bd close <epic-id> --reason "Plugin built, verified, and published"
```

---

## Self-Review

**Spec coverage:** plugin manifest + marketplace (T1), WORKFLOW.md single source (T2), hook injection with bd-detect + Windows path handling (T3), contained idempotent bootstrap with side-effect cleanup (T4), workspace-aware /start-feature (T5), README prereqs/install/scope (T6), end-to-end verification + private publish (T7). All spec sections covered.

**Known risks addressed:** Windows `~`/MSYS home vs `%USERPROFILE%` — handled in T3 (`${USERPROFILE:-$HOME}` + slash normalization) and T4/T5 (explicit `$env:USERPROFILE` vs `$HOME`). No-bash-on-Windows — `run-hook.cmd` exits 0 silently; README lists Git for Windows. bd-missing — hook + `/sb-setup` both detect and guide.

**Config values to fill (not placeholders to leave blank):** `OWNER`, `Your Name`, `you@example.com` in `plugin.json`, `marketplace.json`, `README.md` — confirm with the user in Task 1.
