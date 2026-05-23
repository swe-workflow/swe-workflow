---
description: Batch-run /ship across a backlog — iterate issues in dependency order and ship each AFK issue, pausing at HITL issues. Optional scope = a feature directory.
argument-hint: [scope]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

Run **`/swe-workflow:ship` over every ready issue**, one at a time, unattended.

Scope: **$ARGUMENTS** — optional. A feature directory (e.g. `.scratch/feature-x/` or `.scratch/feature-x/issues/`) limits the batch to that feature. If empty, target **all `ready-for-agent` issues** for the active tracker.

Tracker selection and per-tracker enumeration: `${CLAUDE_PLUGIN_ROOT}/skills/swe-workflow/trackers/`.

## Procedure
1. **Enumerate issues** in scope for the active tracker. For `local-markdown`, list the issue files under the feature's `issues/` directory; for other trackers, query `ready-for-agent` issues.
2. **Order by dependency**: respect each issue's `blocked-by` chain — ship prerequisites first. Skip (for now) any issue whose blockers have not yet merged, and note it.
3. **For each AFK issue, in order**: run the full `/swe-workflow:ship <issue-slug>` procedure to completion (plan -> build -> close out, including PR + worktree teardown) **before** starting the next. One worktree at a time — never two ships in the same worktree. If an AFK issue **escalates mid-build** (a `log-decisions` escalation — a decision it can't justify from an artifact), **park it**: leave its worktree + `DECISIONS.staged.md` intact, skip its dependents (step 2's `blocked-by` logic), and **continue with the next independent issue** — don't halt the batch.
4. **At a HITL issue** (`ready-for-human`, or its AGENT-BRIEF flags HITL): **stop and surface it** — do not attempt it unattended. Report what shipped, what is blocked on a human decision, and what remains. Resume on the user's go-ahead.
5. **After the batch**: print a summary — issues shipped (with PR refs), issues skipped/blocked (with reasons), **issues parked on an escalation (awaiting your decision)**, and any HITL stops awaiting the user.

Each issue is fully isolated (its own worktree, branch, and `task_plan.md`/`findings.md`/`progress.md`). **A build *failure* and a decision *escalation* are different stop-reasons:** if an issue's build fails its 3-strike error protocol (something is broken), **stop the batch** and surface it; a decision escalation only **parks that one issue** and the batch continues (step 3). Review open escalations any time with `/status` from the main checkout. To **resume** a parked issue, resolve its escalation — the resolution appends a `Supersedes` entry (per the `log-decisions` skill) — then re-run `/swe-workflow:ship` on it; its worktree is intact.

**Prerequisites**: same as `/swe-workflow:ship` (`planning-with-files`, `tdd`, `andrej-karpathy-skills:karpathy-guidelines`).
