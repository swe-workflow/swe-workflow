---
description: Take an idea or feature from domain grill to a ready-for-agent backlog (swe-workflow stages 1–4) — grill, features, PRD, issues. Optional arg = a feature slug or raw idea. Idempotent.
argument-hint: [feature-or-idea]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

Invoke the **`swe-workflow`** skill and run its **spec** procedure (`references/spec.md`), following it exactly — kept in the skill so it stays identical for every agent.

The feature or idea to spec: **$ARGUMENTS** (optional — the procedure works from `FEATURES.md` or the conversation if empty).
