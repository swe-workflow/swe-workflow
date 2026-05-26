# Stage 0 — Setup

Bootstrap this repo for the **swe-workflow** suite. One-time, **idempotent** setup — safe to re-run. On Claude Code this runs as `/swe-workflow:setup`; on other agents, invoke the `swe-workflow` skill and ask to bootstrap the repo.

## 1. Ensure prerequisite skills are installed

The workflow depends on external skills this suite doesn't bundle. Detect which are missing and install them.

| Skill(s) | Source (GitHub repo) |
|---|---|
| `planning-with-files` | `OthmanAdi/planning-with-files` |
| `karpathy-guidelines` | `multica-ai/andrej-karpathy-skills` |
| `tdd`, `setup-matt-pocock-skills`, `grill-with-docs`, `to-prd`, `to-issues`, `triage` | `mattpocock/skills` |

1. **Detect** what's already available to your agent and skip those.
2. **Install the missing ones**, announcing each source first (never silently pull third-party code). Use whichever installer your agent supports:
   - **Universal (Agent Skills)** — `npx skills add <github-repo>` (e.g. `npx skills add OthmanAdi/planning-with-files`). Works for Claude Code, Codex, Gemini CLI, Cursor, and other Agent-Skills hosts.
   - **Claude Code plugins** (alternative) — `claude plugin marketplace add <github-repo>` then `claude plugin install <plugin>@<marketplace>`.
3. **Activation may need a restart.** Newly installed skills/plugins are often inactive until the agent restarts. After installing, **report what you installed and tell the user to restart**, then re-run setup to verify. The rule injection in step 3 is a file write — live immediately, no restart needed.

If no skill installer is available here, report it and skip installs (still do steps 2–4).

## 2. Wrap stage-0 setup (mattpocock)

If the `setup-matt-pocock-skills` skill is available, **invoke it** to wire this repo's tracker, triage labels, and doc layout (the `## Agent skills` block in `AGENTS.md`/`CLAUDE.md` + `docs/agents/`). If it's **absent**, **degrade gracefully**: skip it, warn that the mattpocock bootstrap didn't run, and continue — the rule injection below doesn't depend on it.

## 3. Inject the always-on rules

Write the standing rules into the repo's agent-instructions file — `AGENTS.md` if present, else `CLAUDE.md` (match where the repo keeps agent instructions; stage-0 uses the same target). These are the **all-stages** entries in the [always-on rules registry](../REFERENCE.md#always-on-engineering-rules) — two independently sentinel-wrapped blocks.

**Idempotency:** before writing **each** block, `grep` for its sentinel. If present → **no-op** for that block (report "already configured"; never duplicate or overwrite user edits). If absent → append it. The blocks are independent — a repo that has only the decision-logging block from an earlier run gets the engineering-discipline block added on the next.

**3a — Decision logging.** Append if `<!-- swe-workflow:decision-logging -->` is absent:

```markdown
<!-- swe-workflow:decision-logging -->
## Decision logging (swe-workflow)

When you make a decision on the user's behalf they'd want to review — auto-answering a question they'd otherwise be asked, an irreversible/hard-to-undo action, a tradeoff, a deviation from the spec, or resolving a grill question by your own exploration — record it per the `log-decisions` skill: in a swe-workflow ship build (a worktree), stage to `DECISIONS.staged.md` (promoted to `DECISIONS.md` at close-out); otherwise append to `DECISIONS.md` directly. **Look in the repo first** — most "open" questions are already answered there. Decide-and-log when an artifact grounds the call (verify if irreversible); when it's reversible but unsettled, log a best-guess **assumption** (`Outcome: assumed`) and proceed; **escalate** only what's irreversible and needs human context — and always the catastrophic (data loss, migration, spend, public-interface break).
<!-- /swe-workflow:decision-logging -->
```

**3b — Engineering discipline.** Append if `<!-- swe-workflow:engineering-discipline -->` is absent:

```markdown
<!-- swe-workflow:engineering-discipline -->
## Engineering discipline (swe-workflow)

When writing or refactoring code in this repo — in a swe-workflow ship build or an ad-hoc edit — apply the **`andrej-karpathy-skills:karpathy-guidelines`** skill: keep changes surgical; smallest diff that works, no speculative abstractions, surface assumptions instead of guessing silently.

Test-first development via the `tdd` skill is **not** a repo-wide rule — it is scoped to swe-workflow's ship and ship-all builds, where the plan names it in `task_plan.md`. Don't impose red → green → refactor on ad-hoc edits outside those flows.
<!-- /swe-workflow:engineering-discipline -->
```

## 4. Configure the DECISIONS.md privacy policy

`DECISIONS.md` records internal deliberation and is committed by default. Decide whether it's tracked or local:

1. **Detect visibility**: `gh repo view --json visibility -q .visibility` (or infer from the remote).
2. **If the repo is public**, surface the tradeoff: "`DECISIONS.md` will be committed and world-readable in git history; it records internal deliberation — keep it local-only?" — and let the user choose.
3. **Record the choice** in `.swe-workflow.conf` at the repo root — set or **update** the `decisions=` line (`tracked` default, or `local`); don't append a duplicate if one is already present.
4. **On `decisions=local`**, add `DECISIONS.md` to `.gitignore` **if not already listed**.
5. **Always** add `DECISIONS.staged.md` to `.gitignore` **if not already listed** (the ephemeral per-worktree staging file is never committed).

## 5. Choose the landing method

How `/ship` lands a finished branch into the canonical branch ([Landing the work](../REFERENCE.md#landing-the-work)) is a per-repo choice — record it once here.

1. **Check whether `origin` is a fork** — `gh repo view --json isFork -q .isFork` (or: an `upstream` remote exists). This decides the recommendation.
2. **Offer both, and mark the recommendation from that check** — a **fork** can't push to `upstream`, so it needs a PR → recommend **`pr`**; otherwise you own `main` → recommend **`direct`**:
   - **`pr`** — land via a pull request — the reviewable, protected-`main`-safe path. *Also pick this for a non-fork whose `main` is protected/shared.*
   - **`direct`** — merge straight onto `main`, no PR — for a repo you own outright (incl. local-only), where the adversarial review is the only gate.
3. **Record the choice** in `.swe-workflow.conf` — set or **update** the `landing=` line (`pr` or `direct`); don't append a duplicate if one is present. `/ship` reads it at close-out; absent it, the method falls back to topology inference.

## 6. Report

Summarize: prerequisites installed (with the restart reminder), whether stage-0 ran, the rule injections (each added / already present), the privacy choice, and the landing method. If a restart is needed, say so clearly.
