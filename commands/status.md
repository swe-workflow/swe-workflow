---
description: Show swe-workflow planning status for the current issue — phase, progress, decisions, errors — and, from the main checkout, open escalations awaiting a human across all live worktrees.
allowed-tools: Bash, Read
---

Show the planning status for the current worktree / issue.

1. If the `planning-with-files` plugin is installed, invoke **`/planning-with-files:status`** and present its output.
2. Otherwise, read `task_plan.md`, `progress.md`, and `findings.md` in the current directory and report:
   - **Where am I?** — the Current Phase.
   - **Done / remaining?** — phase checkboxes (counts + what's next).
   - **Decisions & errors** — the Decisions table and any rows in the Errors table.
   - **Last activity** — the most recent `progress.md` session entry.

If no planning files exist in the current directory, say so and point the user at `/swe-workflow:ship <issue-slug>` to bootstrap one (or `cd` into the issue's worktree first).

## Repo-wide open escalations (from the main checkout)

When run from the **main checkout** (not inside an issue worktree), also surface what's waiting on a human across all in-flight issues:

- Enumerate live worktrees with `git worktree list`; in each, read `DECISIONS.staged.md` for **`escalated`** entries that have no resolution.
- Report them **aggregated and sorted by timestamp** — "<n> open escalations across <m> worktrees" — each with its issue context, question, and why it escalated.
- Flagged **assumptions** (`Outcome: assumed`) are *not* shown here — they're non-blocking and surface in the PR body / journal for review. `/status` is for what's *blocking* on you.
- Skip prunable/missing worktree paths. This is **read-only** — it never writes.

This is how a returning operator finds every parked `/ship-all` decision in one place.
