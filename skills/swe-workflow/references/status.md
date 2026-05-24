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
