# Stages 5–7 — Ship

Run the **execution layer** — stages 5 → 6 → 7 — for a single issue: plan it, build it test-first, open the PR, tear down. On Claude Code this runs as `/swe-workflow:ship`; on other agents, invoke the `swe-workflow` skill and ask to ship an issue.

**The issue to ship** comes from the user's request — an issue slug such as a local-markdown path (`.scratch/feature-x/issues/02-foo.md`) or a tracker id (a GitHub `#number`, a Linear `TEAM-NN` key). If no issue is identified, ask which one and stop — don't guess.

**Unattended, not unsupervised.** For an **AFK** issue, ship runs end-to-end without interviewing you — `/tdd`'s mid-build calls go through the `log-decisions` rules (decide / assume / escalate), not to a prompt. It doesn't decide blindly: a reversible best-guess is logged as an **assumption** and surfaced in the PR's *Autonomy decisions* section to confirm, and an **unsure HITL call** (irreversible, needs you) does **not** pause the run — it's **persisted for your batch review**, parked and surfaced by [`/status`](status.md), while shipping continues. (A **pre-declared HITL** issue is separate — it pauses at its documented checkpoints; see [HITL](#hitl).) `/ship-all` ([`ship-all.md`](ship-all.md)) is this same per-issue posture, looped over the backlog.

Rationale, the security boundary, and per-tracker fetch commands live alongside this file: [REFERENCE.md](../REFERENCE.md), [trackers/](../trackers/).

## Stage 5 — Plan
1. **Resolve the tracker** per the [tracker contract](../trackers/README.md#selection--which-adapter) — the issue slug is the rung-1 arg form (a `.md` path, a `TEAM-NN` key, or a bare number).
2. **Fetch the issue** per the matching `trackers/<name>.md`. For `local-markdown`, read the file at that path. Extract: title, body, acceptance criteria, AGENT-BRIEF, blocked-by, and whether it is HITL or AFK.
3. **Derive paths**: `slug` = title lowercased, non-alphanumerics → `-`, truncated to 40 chars; `branch` = `issue-<id>-<slug>`; `worktree` = `../<repo>-issue-<id>/`.
4. **Create the worktree — re-run check first (idempotent).**
   - **Closed/merged, worktree still present** → a prior run merged but didn't finish: **complete the teardown** (Stage 7's journal promote + `git worktree remove` + branch delete), then stop.
   - **Closed/merged, no worktree** → report "already shipped" and stop; don't redo it.
   - **Worktree exists, issue still open** → resuming an interrupted or parked ship: `cd` in and continue from `task_plan.md`'s recorded state; never re-bootstrap or clobber its planning files.
   - **Otherwise** → `git worktree add ../<repo>-issue-<id> -b issue-<id>-<slug>` and work inside it.
5. **Seed the three planning files** (security boundary — structured fields only in `task_plan.md`; raw external text in `findings.md`):
   - `task_plan.md` — Goal = title; Phases = AC checkboxes.
   - `findings.md` — raw issue body + AGENT-BRIEF, pasted verbatim.
   - `progress.md` — initial bootstrap log entry.
6. **Invoke the `planning-with-files` skill to plan** (its `plan` workflow) with this exact prompt — this is the **single source** for the planner prompt; it bakes the methodology into `task_plan.md`:
   > Interview me about this issue, then write task_plan.md to implement it. Write these two rules into task_plan.md, naming each skill explicitly so it activates when the plan is executed: (1) Use the tdd skill for all code and tests — tests first: red → green → refactor. (2) When writing or refactoring code, apply the andrej-karpathy-skills:karpathy-guidelines skill for code quality — surgical, simple changes.

## Stage 6 — Build
7. **Invoke `planning-with-files` to execute the plan** (its `plan-goal` workflow) — it reads `task_plan.md` and drives each phase to completion. Because the plan names the `tdd` and `karpathy-guidelines` skills, build test-first and keep changes surgical. Log each cycle to `progress.md`; record execution-time decisions and errors in `task_plan.md`. When a **bar-crossing decision** arises mid-build, resolve it per the **`log-decisions`** rules (look first → decide / assume / escalate) and stage it to `DECISIONS.staged.md`. The build-layer delta — an **escalation persists rather than pauses**: it parks the issue for batch review (surfaced by [`/status`](status.md)) while the run keeps going. Reversible, spec-authorized execution decisions stay in the `task_plan.md` Decisions table. **Out-of-scope discoveries** (a new issue or feature) get captured to a durable home — the tracker or `FEATURES.md` — never folded into this worktree; see *Discovered scope* in [REFERENCE.md](../REFERENCE.md#discovered-scope).

## Stage 7 — Close out
8. Open the PR with the body drawn from `progress.md` highlights — the session log IS the narrative; don't rewrite it. Include an **"Autonomy decisions"** section in the PR body, drawn from `DECISIONS.staged.md`, so the reviewer sees what was decided unattended — especially the flagged **assumptions** (`assumed`) to confirm.
9. After the PR merges, **promote the journal, then tear down** — **from the main checkout** (not inside the worktree): first append any `DECISIONS.staged.md` entries to repo-root `DECISIONS.md` and commit them as their own `log:` commit (serialized by teardown → conflict-free across parallel worktrees); then `git worktree remove ../<repo>-issue-<id>` (which discards the staging file), and delete the branch if it merged into the default branch.

## HITL
If the issue is `ready-for-human` or its AGENT-BRIEF flags HITL checkpoints: pause at the documented checkpoint, surface the decision, and wait for direction — don't let `planning-with-files`' stop/auto-finish mechanism end the session. Resume by re-invoking `planning-with-files`' `plan-goal` after the human responds.

**Prerequisites** (not bundled): the `planning-with-files`, `tdd`, and `andrej-karpathy-skills:karpathy-guidelines` skills. If a prerequisite is missing, say so and stop rather than improvising.
