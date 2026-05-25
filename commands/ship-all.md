---
description: Batch-run /ship across a backlog — ship each AFK issue sequentially in dependency order, one worktree at a time, each with a fresh-context review before close-out, pausing at HITL issues. Optional scope = a feature directory.
argument-hint: [scope]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

Invoke the **`swe-workflow`** skill and run its **ship-all** procedure (`references/ship-all.md`), following it exactly — kept in the skill so it stays identical for every agent.

The scope (an optional feature directory): **$ARGUMENTS** (empty = all `ready-for-agent` issues for the active tracker).
