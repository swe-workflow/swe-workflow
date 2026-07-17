# Status

Show the status for the current project or issue. **Read-only** — it never writes. On Claude Code this runs as `/swe-workflow:status`; on other agents, invoke the `swe-workflow` skill and ask for status.

1. **In an issue worktree** — when `task_plan.md`, `progress.md`, and `findings.md` exist in the current directory — invoke the **`planning-with-files`** skill's `status` workflow (required; if it's not installed, say so and stop) and report:
   - **Where am I?** — the Current Phase.
   - **Done / remaining?** — phase checkboxes (counts + what's next).
   - **Decisions & errors** — the Decisions table and any rows in the Errors table.
   - **Last activity** — the most recent `progress.md` session entry.
2. **At the project level** — those planning files aren't in the current directory, e.g. the main checkout — roll up every in-flight issue across all live worktrees (see below).

## Project-level rollup (from the main checkout)

When run from the **main checkout** (not inside an issue worktree), enumerate live worktrees with `git worktree list` and, for each issue worktree (one carrying the planning files), report:

- **Progress** — its current phase, phase checkboxes (done / remaining), and most recent session entry, read from that worktree's `task_plan.md` / `progress.md`. One line per issue.
- **Open escalations** — the **`escalated`** entries with no resolution in its `DECISIONS.staged.md`. Aggregate them **sorted by timestamp** — "<n> open escalations across <m> worktrees" — each with its issue context, question, and why it escalated.

Flagged **assumptions** (`Outcome: assumed`) are *not* shown — they're non-blocking and surface in the PR body / journal for review. Status is for what's *in flight* and what's *blocking* on you.

Skip prunable/missing worktree paths.

This is how a returning operator sees every in-flight issue and every parked **ship-all** decision in one place.

## Levels of done

Four levels of "done"; the toolchain verifies three and deliberately stays out of the fourth. Status *reports* them — the writes happen elsewhere (ship's close-out).

| Level | Done when | Recorded in |
|---|---|---|
| Phase | its test-first cycle lands green (or non-code work is logged) | `task_plan.md` checkbox — dies at teardown |
| Issue | all phases ticked **and** the change merged | the tracker's terminal state ([lifecycle](../trackers/README.md#lifecycle-states)) |
| Feature | every child issue of its PRD terminal | `FEATURES.md` strike-through + shipped refs |
| Project | no native concept — a judgment call | — |

**Feature detection**: walk from the PRD to its child issues (via the parent reference each issue body carries) and confirm every child is [terminal](../trackers/README.md#lifecycle-states) — a rejected or duplicate child is *resolved* and doesn't block the feature; a child still in-flight does. GitHub: `gh issue list --search "parent:<PRD#> state:open"` returns empty. Local-markdown: every file in `.scratch/<feature-slug>/issues/` has a terminal `Status:`. Others: the adapter's child-issue query.

**Project completion is policy, not a fact** — it depends on release cadence, commitments, and milestone definitions the toolchain can't see. Layer a hard signal on your tracker (GitHub milestones, Linear cycles, release tags) if you need one; the closest native proxy is a `FEATURES.md` snapshot with zero unstruck `- [ ]` lines — a snapshot, not a guarantee.
