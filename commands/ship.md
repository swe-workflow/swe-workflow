---
description: Plan, build, and close out ONE issue end to end (swe-workflow stages 5->6->7). Takes an issue slug — a local-markdown issue path, or a tracker id (GitHub #, Linear key).
argument-hint: <issue-slug>
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

Invoke the **`swe-workflow`** skill and run its **ship** procedure (`references/ship.md`, which holds the single-source planner prompt), following it exactly — kept in the skill so it stays identical for every agent.

The issue to ship: **$ARGUMENTS** (if empty, the procedure asks which issue — don't guess).
