---
description: One-time, idempotent setup of the home Beads workspace for superpowered-beads
---

Set up (or repair) the home Beads database this plugin uses (`~/.beads`). Idempotent —
safe to re-run. Target the home workspace with `bd -C "$HOME" ...`
(Windows: `bd -C "$env:USERPROFILE" ...`).

Do these in order and report what changed:

1. **Check bd.** Run `bd version`. If it fails, tell the user to install Beads
   (https://github.com/gastownhall/beads) and STOP.

2. **Use the existing home DB, or create one cleanly.** Run `bd -C "$HOME" status`
   (Windows: `bd -C "$env:USERPROFILE" status`).
   - If it SUCCEEDS, a home `~/.beads` already exists — skip to step 3 (do NOT init).
   - If it FAILS (no home DB), create one. `bd init` also drops a git repo and agent files
     in the home root, so record what already exists FIRST and afterward remove ONLY what
     bd newly created. **Never delete a `~/.git` or agent file that existed beforehand.**

     PowerShell:
     ```powershell
     $h = $env:USERPROFILE
     $pre = @{}; foreach ($f in '.git','AGENTS.md','.agents','.codex') { $pre[$f] = Test-Path "$h\$f" }
     bd -C "$h" init --non-interactive
     foreach ($f in '.git','AGENTS.md','.agents','.codex') {
       if (-not $pre[$f] -and (Test-Path "$h\$f")) { Remove-Item -Recurse -Force "$h\$f" }
     }
     ```
     bash:
     ```bash
     h="$HOME"
     pre_git=$([ -e "$h/.git" ] && echo 1 || echo 0)
     pre_ag=$([ -e "$h/AGENTS.md" ] && echo 1 || echo 0)
     pre_a=$([ -e "$h/.agents" ] && echo 1 || echo 0)
     pre_c=$([ -e "$h/.codex" ] && echo 1 || echo 0)
     bd -C "$h" init --non-interactive
     [ "$pre_git" = 0 ] && rm -rf "$h/.git"
     [ "$pre_ag" = 0 ] && rm -f "$h/AGENTS.md"
     [ "$pre_a" = 0 ] && rm -rf "$h/.agents"
     [ "$pre_c" = 0 ] && rm -rf "$h/.codex"
     ```

3. **Disable git ops.** `bd -C "$HOME" config set no-git-ops true`
   (Windows: `bd -C "$env:USERPROFILE" config set no-git-ops true`).

4. **Seed the workflow memory (if absent).** Check `bd -C "$HOME" memories` for key
   `workflow`. If absent, run:
   `bd -C "$HOME" remember "WORKFLOW: Superpowers lifecycle on top of Beads. Hybrid tasks (beads for durable work, TodoWrite only for throwaway in-session checklists), conservative git (never commit/push unless asked), durable memory via bd remember, /start-feature to begin. See the plugin's WORKFLOW.md." --key workflow`

5. **Verify and report.** Run `bd -C "$HOME" status` (succeeds) and
   `bd -C "$HOME" config get no-git-ops` (prints `true`). Report what you did: used an
   existing DB vs created one, whether any stray git/agent files were cleaned, and whether
   the memory was seeded or already present.
