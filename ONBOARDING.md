# Onboarding — hand this to a friend

This plugin sets itself up best with Claude's help. Tell your friend to:

1. Make sure you've given them **read access** to `Voskavar/superpowered-beads` (add them as a
   collaborator on GitHub, or make the repo public). Without access, the install step fails.
2. Open a **fresh Claude Code session** and paste the prompt in the box below.

Claude will detect their OS, check what's installed, install the missing pieces (asking first),
add the plugin, and walk them through the one restart + `/sb-setup`.

---

## The paste-in prompt

> You're my setup assistant for installing the **superpowered-beads** Claude Code plugin — it
> pairs the *Superpowers* development workflow with the *Beads* (`bd`) issue tracker. Walk me
> through installing and configuring it on my machine. Run **read-only checks** yourself; before
> installing any software or changing config, show me the exact command and ask first. **Detect my
> OS** (Windows/macOS/Linux) and adapt all commands accordingly. Don't move past a step until it
> actually succeeded.
>
> **What this is:** a Claude Code plugin distributed from the **private** GitHub repo
> `Voskavar/superpowered-beads`. I need read access to it — if a step fails with a 404 or auth
> error, stop and tell me I probably haven't been granted access yet.
>
> **Step 1 — Check prerequisites. Report what's present vs missing before changing anything:**
> 1. **Claude Code** — already running (that's you).
> 2. **Git + bash** — run `git --version`. On Windows, Git for Windows is required because it
>    provides the `bash` the plugin's SessionStart hook runs; if missing, point me to
>    https://git-scm.com/download/win .
> 3. **Beads (`bd`)** — run `bd version`. If missing, install from
>    https://github.com/gastownhall/beads (Windows: `winget install` or `go install`;
>    macOS/Linux: per that repo). Re-check `bd version` after.
> 4. **GitHub access for a private marketplace** — I need git able to read a private repo. If I'm
>    not authenticated, suggest installing the GitHub CLI and running `gh auth login` (I'll do the
>    interactive login myself), or configuring a git credential helper.
> 5. **Superpowers plugin** — check whether its skills (e.g. brainstorming, writing-plans) are
>    available. If not, install it:
>    `claude plugin marketplace add obra/superpowers-marketplace`
>    `claude plugin install superpowers@superpowers-dev`
>
> **Step 2 — Install this plugin:**
> `claude plugin marketplace add Voskavar/superpowered-beads`
> `claude plugin install superpowered-beads@superpowered-beads`
> (If `marketplace add` fails with 404/auth, stop — I lack repo access. If a `claude plugin`
> command can't run from your session, give me the exact command to run in my own terminal.)
>
> **Step 3 — Activate:** tell me to **fully quit and reopen Claude Code** — the SessionStart hook
> and the `/start-feature` and `/sb-setup` commands only load after a restart.
>
> **Step 4 — Finish setup (after I've restarted):** have me run **`/sb-setup`** once. It creates
> or reuses my home Beads database at `~/.beads`, sets `no-git-ops`, seeds the workflow memory, and
> safely cleans up any stray files `bd init` drops in my home folder — it must never delete a
> `~/.git` or agent file that already existed.
>
> **Step 5 — Verify:** in a new session, confirm the workflow rules were injected (you should see
> the "Superpowers × Beads" rules at the top of context) and that `/start-feature` is available.
> Report success or exactly what failed.
>
> **How I'll use it once set up:**
> - `/start-feature <idea>` — brainstorm → bd epic → plan → child-task graph → ready-to-work queue.
> - `bd ready` / `bd create` / `bd update <id> --claim` / `bd close <id>` for task tracking.
> - `bd remember "<insight>" --key <slug>` for durable notes (auto-loaded each session).
>
> Begin with Step 1 now: detect my OS and check the prerequisites, then report before doing
> anything that changes my system.

---

## Notes for whoever shares this

- **Access:** the repo is private. Either add friends as collaborators
  (`gh repo add-collaborator` or GitHub → Settings → Collaborators), or make it public
  (`gh repo edit Voskavar/superpowered-beads --visibility public`).
- **What's shared vs theirs:** they get the workflow + tooling. Their Beads database, issues, and
  memories are created locally in their own `~/.beads` and never touch this repo.
- If you cut a tagged release later, point friends at the tag for a stable version.
