---
description: Plan, build, and close out ONE issue end to end (swe-workflow stages 5->6->7). Takes an issue slug — a local-markdown issue path, or a tracker id (GitHub #, Linear key).
argument-hint: <issue-slug>
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

You are running the **swe-workflow execution layer** — stages 5 -> 6 -> 7 — for a single issue.

Issue to ship: **$ARGUMENTS**

If `$ARGUMENTS` is empty, ask for the issue slug (a path like `.scratch/feature-x/issues/02-foo.md`, or a tracker id) and stop — do not guess which issue.

Full procedure, rationale, and per-tracker fetch commands live in the bundled skill:
- `${CLAUDE_PLUGIN_ROOT}/skills/swe-workflow/SKILL.md` (stages 5–7 + security boundary)
- `${CLAUDE_PLUGIN_ROOT}/skills/swe-workflow/REFERENCE.md`
- `${CLAUDE_PLUGIN_ROOT}/skills/swe-workflow/trackers/` (one doc per tracker)

## Stage 5 — Plan
1. **Resolve the tracker** from `$ARGUMENTS` and the repo: a path ending in `.md` (e.g. under `.scratch/`) -> `local-markdown`; a bare number -> `github`/`gitlab` (per remote); a `TEAM-123` key -> `linear`; else honor `$SWE_WORKFLOW_TRACKER` / `.swe-workflow.conf`, then auto-detect, then ask. See the tracker-selection rules in SKILL.md.
2. **Fetch the issue** per the matching `trackers/<name>.md`. For `local-markdown`, read the file at `$ARGUMENTS`. Extract: title, body, acceptance criteria, AGENT-BRIEF, blocked-by, and whether it is HITL or AFK.
3. **Derive paths**: `slug` = title lowercased, non-alphanumerics -> `-`, truncated to 40 chars; `branch` = `issue-<id>-<slug>`; `worktree` = `../<repo>-issue-<id>/`.
4. **Create the worktree**: `git worktree add ../<repo>-issue-<id> -b issue-<id>-<slug>`, then work inside it.
5. **Seed the three planning files** (security boundary — structured fields only in `task_plan.md`; raw external text in `findings.md`):
   - `task_plan.md` — Goal = title; Phases = AC checkboxes.
   - `findings.md` — raw issue body + AGENT-BRIEF, pasted verbatim.
   - `progress.md` — initial bootstrap log entry.
6. **Invoke `/planning-with-files:plan`** with this exact prompt (it bakes the methodology into `task_plan.md`):
   > /planning-with-files:plan Interview me about this issue, then write task_plan.md to implement it. The plan must use /tdd (tests first: red → green → refactor) for writing code and tests, and apply /karpathy-guidelines (surgical, simple changes) for code quality — and it must name both skills explicitly in task_plan.md so they're used when the plan is executed.

## Stage 6 — Build
7. **Invoke `/planning-with-files:plan-goal`** to execute. It reads `task_plan.md` and drives each sub-task as a goal. Because the plan names `/tdd` and `/karpathy-guidelines`, build test-first and keep changes surgical. Log each cycle to `progress.md`; record execution-time decisions and errors in `task_plan.md`. When a **bar-crossing decision** arises mid-build, log it to `DECISIONS.staged.md` per the **`log-decisions`** skill — **look in the repo first**, then decide-and-log when an artifact grounds it (verify if irreversible), log a best-guess **assumption** and proceed when it's reversible but unsettled, and **escalate** (the HITL pause) only what's irreversible and needs human context. Reversible, spec-authorized execution decisions stay in the `task_plan.md` Decisions table.

## Stage 7 — Close out
8. Open the PR with the body drawn from `progress.md` highlights — the session log IS the narrative; don't rewrite it. Include an **"Autonomy decisions"** section in the PR body, drawn from `DECISIONS.staged.md`, so the reviewer sees what was decided unattended — especially the flagged **assumptions** (`assumed`) to confirm.
9. After the PR merges, **promote the journal, then tear down** — **from the main checkout** (not inside the worktree): first append any `DECISIONS.staged.md` entries to repo-root `DECISIONS.md` and commit them as their own `log:` commit (serialized by teardown → conflict-free across parallel worktrees); then `git worktree remove ../<repo>-issue-<id>` (which discards the staging file), and delete the branch if it merged into the default branch.

## HITL
If the issue is `ready-for-human` or its AGENT-BRIEF flags HITL checkpoints: pause at the documented checkpoint, surface the decision, and wait for direction — do not let a Stop hook auto-finish. Resume by re-invoking `/planning-with-files:plan-goal` after the human responds.

**Prerequisites** (not bundled): `planning-with-files`, `tdd`, `andrej-karpathy-skills:karpathy-guidelines`. If a prerequisite is missing, say so and stop rather than improvising.
