---
description: Batch-run /ship across a backlog — iterate issues in dependency order and ship each AFK issue, pausing at HITL issues. Optional scope = a feature directory.
argument-hint: [scope]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

Invoke the **`swe-workflow`** skill and run its **ship-all** procedure (`references/ship-all.md`), following it exactly — kept in the skill so it stays identical for every agent.

The scope (an optional feature directory): **$ARGUMENTS** (empty = all `ready-for-agent` issues for the active tracker).
