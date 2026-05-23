---
description: Bootstrap a repo for swe-workflow — auto-install missing prerequisite skills, wrap stage-0 setup, inject the always-on decision-logging rule, and configure the DECISIONS.md privacy policy. Safe to re-run (idempotent).
allowed-tools: Bash, Read, Write, Edit, Grep
---

You are bootstrapping this repo for the **swe-workflow** plugin. One-time, **idempotent** setup — safe to re-run.

## 1. Ensure prerequisite skills are installed

The workflow depends on external skills this plugin doesn't bundle. Detect which are missing and install them.

| Skill(s) | Source (GitHub repo) |
|---|---|
| `planning-with-files` | `OthmanAdi/planning-with-files` |
| `karpathy-guidelines` | `multica-ai/andrej-karpathy-skills` |
| `tdd`, `setup-matt-pocock-skills`, `grill-with-docs`, `to-prd`, `to-issues`, `triage` | `mattpocock/skills` |

1. **Detect** what's installed: read `~/.claude/plugins/installed_plugins.json` and `enabledPlugins` in `~/.claude/settings.json`. Skip anything already present.
2. **Install the missing ones**, announcing each source first (never silently pull third-party code):
   ```bash
   claude plugin marketplace add <github-repo>
   claude plugin install <plugin>@<marketplace>
   ```
3. **Two-phase / restart.** Newly installed plugins are **not active until Claude Code restarts** (the CLI's own `update` says "restart required to apply"). After installing, **report what you installed and tell the user to restart**, then re-run `/swe-workflow:setup` to verify. The rule injected in step 3 is a file write — live immediately, no restart needed.

If `claude plugin` is unavailable here, report it and skip installs (still do steps 2–4).

## 2. Wrap stage-0 setup (mattpocock)

If `/setup-matt-pocock-skills` is installed, **invoke it** to wire this repo's tracker, triage labels, and doc layout. If it's **absent**, **degrade gracefully**: skip it, warn that the mattpocock bootstrap didn't run, and continue — the rule injection below doesn't depend on it.

## 3. Inject the always-on decision-logging rule

Write the rule into the repo's agent-instructions file — `AGENTS.md` if present, else `CLAUDE.md` (match where the repo keeps agent instructions; stage-0 uses the same target).

**Idempotency:** the section is sentinel-wrapped. Before writing, `grep` for `<!-- swe-workflow:decision-logging -->`. If present → **no-op** (report "already configured"; never duplicate or overwrite user edits). If absent → append:

```markdown
<!-- swe-workflow:decision-logging -->
## Decision logging (swe-workflow)

When you make a decision on the user's behalf they'd want to review — auto-answering a question they'd otherwise be asked, an irreversible/hard-to-undo action, a tradeoff, a deviation from the spec, or resolving a grill question by your own exploration — record it per the `log-decisions` skill: in a `/ship` worktree build, stage to `DECISIONS.staged.md` (promoted to `DECISIONS.md` at close-out); otherwise append to `DECISIONS.md` directly. **Look in the repo first** — most "open" questions are already answered there. Decide-and-log when an artifact grounds the call (verify if irreversible); when it's reversible but unsettled, log a best-guess **assumption** (`Outcome: assumed`) and proceed; **escalate** only what's irreversible and needs human context — and always the catastrophic (data loss, migration, spend, public-interface break).
<!-- /swe-workflow:decision-logging -->
```

## 4. Configure the DECISIONS.md privacy policy

`DECISIONS.md` records internal deliberation and is committed by default. Decide whether it's tracked or local:

1. **Detect visibility**: `gh repo view --json visibility -q .visibility` (or infer from the remote).
2. **If the repo is public**, surface the tradeoff: "`DECISIONS.md` will be committed and world-readable in git history; it records internal deliberation — keep it local-only?" — and let the user choose.
3. **Record the choice** in `.swe-workflow.conf` at the repo root — set or **update** the `decisions=` line (`tracked` default, or `local`); don't append a duplicate if one is already present.
4. **On `decisions=local`**, add `DECISIONS.md` to `.gitignore` **if not already listed**.
5. **Always** add `DECISIONS.staged.md` to `.gitignore` **if not already listed** (the ephemeral per-worktree staging file is never committed).

## 5. Report

Summarize: prerequisites installed (with the restart reminder), whether stage-0 ran, the rule injection (added / already present), and the privacy choice. If a restart is needed, say so clearly.
