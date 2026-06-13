# Onboarding — hand this to a friend

This plugin sets itself up best with Claude's help. Tell your friend to open a **fresh Claude
Code session** and paste the prompt below. Claude will detect their OS, check what's installed,
install the missing pieces (asking first), wire up the plugin, and walk them through the one
restart + `/sb-setup`.

The repo is **public**, so no GitHub access or auth is needed.

---

## The paste-in prompt

> You're my setup assistant for installing the **superpowered-beads** Claude Code plugin — it
> pairs the *Superpowers* development workflow with the *Beads* (`bd`) issue tracker. Walk me
> through installing and configuring it. Run **read-only checks** yourself; before installing any
> software or changing config, show me the exact command and ask first. **Detect my OS**
> (Windows/macOS/Linux) and adapt commands. Don't move past a step until it actually succeeded.
>
> **What this is:** a Claude Code plugin from the **public** GitHub repo
> `Voskavar/superpowered-beads` (no auth needed to fetch it).
>
> **Step 1 — Check prerequisites; report present vs missing before changing anything:**
> 1. **Claude Code** — already running (that's you).
> 2. **Git + bash** — `git --version`. On Windows, Git for Windows is required (it provides the
>    `bash` the plugin's SessionStart hook runs); if missing: https://git-scm.com/download/win .
> 3. **Beads (`bd`)** — `bd version`. If missing, install from
>    https://github.com/gastownhall/beads (Windows: `winget install` or `go install`). Re-check.
> 4. **Superpowers plugin** — check whether its skills (brainstorming, writing-plans) are
>    available; if not, install marketplace `obra/superpowers-marketplace`, plugin `superpowers`.
>
> **Step 2 — Install this plugin. Use whichever fits my client:**
> - **If I have the `/plugin` command or the `claude` CLI:**
>     claude plugin marketplace add Voskavar/superpowered-beads
>     claude plugin install superpowered-beads@superpowered-beads
> - **If `/plugin` is NOT available (e.g. the desktop app):** add these to my
>   `~/.claude/settings.json` (Windows `C:\Users\<me>\.claude\settings.json`), merging into what's
>   already there:
>     "enabledPlugins": { "superpowered-beads@superpowered-beads": true },
>     "extraKnownMarketplaces": {
>       "superpowered-beads": { "source": { "repo": "Voskavar/superpowered-beads", "source": "github" } }
>     }
>   If a stale/broken `superpowered-beads` entry already exists, remove it first to bust the cache.
>
> **Step 3 — Activate:** tell me to **fully quit and reopen Claude Code** — the SessionStart hook
> and the `/start-feature` and `/sb-setup` commands only load after a restart.
>
> **Step 4 — Finish (after I restart):** have me run **`/sb-setup`** once. It creates or reuses my
> home Beads DB at `~/.beads`, sets `no-git-ops`, seeds the workflow memory, and safely cleans up
> any stray files `bd init` drops in my home folder — it must never delete a `~/.git` or agent file
> that already existed.
>
> **Step 5 — Verify:** in a new session, confirm `/start-feature` and `/sb-setup` are recognized
> and the "Superpowers × Beads" rules appear at the top of context. If a command is "unknown," the
> plugin didn't load — check the plugin load log and report what it says before changing more.
>
> **How I'll use it once set up:**
> - `/start-feature <idea>` — brainstorm → bd epic → plan → child-task graph → ready queue.
> - `bd ready` / `bd create` / `bd update <id> --claim` / `bd close <id>` for tasks.
> - `bd remember "<insight>" --key <slug>` for durable notes (auto-loaded each session).
>
> Begin with Step 1: detect my OS and check prerequisites, then report before changing anything.

---

## Notes for whoever shares this

- **It's public** — anyone can install; nothing sensitive is in the repo (each user's Beads
  issues and memories live locally in their own `~/.beads`, never in the repo).
- **Known gotcha:** the plugin's `marketplace.json` must source the plugin via a **url object**
  (`{ "source": "url", "url": "https://github.com/Voskavar/superpowered-beads.git" }`), not `"./"`
  — the desktop app shows 0 plugins for `"./"`.
- **Install on the desktop app** is file-based: the client clones the repo into its plugin cache,
  records it in `installed_plugins.json`, and enables it via `settings.json` `enabledPlugins`. The
  settings.json snippet in the prompt is the reliable route there.
- The `v0.1.0` release is a stable snapshot if you'd rather point friends at a tag than `main`.
