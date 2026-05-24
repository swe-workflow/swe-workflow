---
description: Enumerate the project's coarse-grained user-facing features into FEATURES.md (swe-workflow stage 2) — the product-manager step that invokes grill-with-docs at a high level, capturing a rich detail block per feature so /grill-feature starts informed. Runs after the stage-1 domain grill, before /grill-feature. Idempotent — refines an existing FEATURES.md rather than overwriting.
argument-hint: [idea-or-scope]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

Invoke the **`swe-workflow`** skill and run its **to-features** procedure (`references/to-features.md`), following it exactly — kept in the skill so it stays identical for every agent.

The project or scope to enumerate features for: **$ARGUMENTS** (optional — the procedure grills from `CONTEXT.md` + the live conversation, and refines an existing `FEATURES.md` if present).
