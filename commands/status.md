---
description: Show swe-workflow planning status for the current issue — phase, progress, decisions, and any logged errors from the worktree's planning files.
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
