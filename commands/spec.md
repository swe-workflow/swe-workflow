---
description: Derive PRDs and issues from an idea or feature end to end (swe-workflow stages 1->2->3->4) — grill the domain, enumerate features, write the PRD, slice into tracer-bullet issues. Leaves a ready-for-agent backlog for /ship. Optional arg = a feature slug or a raw idea. Idempotent — resumes from whatever already exists.
argument-hint: [feature-or-idea]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

Invoke the **`swe-workflow`** skill and run its **spec** procedure (`references/spec.md`), following it exactly — kept in the skill so it stays identical for every agent.

The feature or idea to spec: **$ARGUMENTS** (optional — the procedure works from `FEATURES.md` or the conversation if empty).
