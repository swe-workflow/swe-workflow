---
description: Show swe-workflow status for the current project or issue — phase, progress, decisions, errors — and, from the main checkout, every in-flight issue's progress plus open escalations across all live worktrees.
allowed-tools: Bash, Read
---

Invoke the **`swe-workflow`** skill and run its **status** procedure (`references/status.md`), following it exactly — kept in the skill so it stays identical for every agent. It reports status for the current issue worktree, or rolls up every in-flight issue and open escalation from the main checkout. Read-only.
