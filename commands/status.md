---
description: Show swe-workflow status — the current issue's phase, progress, decisions, and errors; or from the main checkout, every in-flight issue plus open escalations across worktrees.
allowed-tools: Bash, Read
---

Invoke the **`swe-workflow`** skill and run its **status** procedure (`references/status.md`), following it exactly — kept in the skill so it stays identical for every agent. It reports status for the current issue worktree, or rolls up every in-flight issue and open escalation from the main checkout. Read-only.
