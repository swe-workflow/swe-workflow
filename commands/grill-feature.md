---
description: Spec one feature into a PRD (swe-workflow stage 3) — grill it, then synthesize the PRD. Pass a feature slug from FEATURES.md. Idempotent — skips a feature that already has a PRD.
argument-hint: [feature-slug]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

Invoke the **`swe-workflow`** skill and run its **grill-feature** procedure (`references/grill-feature.md`), following it exactly — kept in the skill so it stays identical for every agent.

The feature to spec into a PRD: **$ARGUMENTS** (a feature slug from `FEATURES.md` — optional; the procedure picks an un-specced feature or asks if empty).
