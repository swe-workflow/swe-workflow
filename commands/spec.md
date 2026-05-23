---
description: Spec ONE feature end to end (swe-workflow stages 1->2->3->4): grill the domain, enumerate features, write the PRD, slice into tracer-bullet issues. Leaves a ready-for-agent backlog for /ship. Optional arg = a feature slug or a raw idea. Idempotent — resumes from whatever already exists.
argument-hint: [feature-or-idea]
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

You are running the **swe-workflow spec layer** — stages 1 -> 2 -> 3 -> 4 — turning an idea into a `ready-for-agent` backlog. The mirror image of `/swe-workflow:ship` (stages 5-7): **`spec` produces the issues, `ship` builds them.**

Focus: **$ARGUMENTS** — optional. A feature slug from `FEATURES.md` specs that one; a raw idea/topic starts from scratch; empty works from the conversation, or from the un-specced features already in `FEATURES.md`.

Full procedure and per-stage detail (including when to skip each) live in the bundled skill:
- `${CLAUDE_PLUGIN_ROOT}/skills/swe-workflow/SKILL.md` + `${CLAUDE_PLUGIN_ROOT}/skills/swe-workflow/REFERENCE.md`

**This command is interactive, not AFK** — `/grill-with-docs` and `/to-issues` interview you. Expect to answer questions. (The unattended part is `/swe-workflow:ship-all` afterward.)

## Idempotent by design — do only what's missing

Detect where the feature already is in the spec layer and resume; never clobber or duplicate. Each stage states its skip condition; run only the stages whose artifact is absent or stale.

## Stage 1 — Grill the domain (`/grill-with-docs`)
Resolve the domain language → `CONTEXT.md` (+ ADRs under `docs/adr/` for substantive decisions).
- **Skip if** terminology is settled: `CONTEXT.md` exists and a grill pass surfaces no new questions.
- Otherwise invoke `/grill-with-docs`. It's re-runnable — each pass **sharpens** `CONTEXT.md` (and may add ADRs); it refines, never duplicates. Loop until no questions remain or the user calls it.

## Stage 2 — Enumerate features (`/swe-workflow:to-features`)
Read `CONTEXT.md` + ADRs → user-facing features in `FEATURES.md`.
- **Idempotent**: if `FEATURES.md` exists, this is a **refinement** — propose additions / strikethroughs; never overwrite shipped (struck-through) lines.

## Stage 3 — Write the PRD (`/to-prd`)
For the target feature, synthesize a PRD (Problem / Solution / User Stories / Implementation + Testing Decisions / Out of Scope), published as a parent issue labeled `ready-for-agent`.
- **Skip if** a PRD already exists for this feature — don't create a duplicate parent (re-run `/to-prd` only to refine a PRD whose feature genuinely changed).
- If no feature is targeted yet, pick from `FEATURES.md`; ask the user which when it's ambiguous.

## Stage 4 — Slice into issues (`/to-issues`)
Break the PRD into **tracer-bullet** issues (thin vertical slices through every layer), each labeled `ready-for-agent`, with `blocked-by` chains and HITL/AFK tags.
- **Skip if** the PRD is already sliced (issues exist with it as parent) — re-slicing forks state. To refine an existing slice, send it back through `/triage`; don't re-run `/to-issues` on a started PRD.

## Done
You now have a `ready-for-agent` backlog. Hand off to **`/swe-workflow:ship <issue>`** (one) or **`/swe-workflow:ship-all`** (the batch).

**Prerequisites** (not bundled): `grill-with-docs`, `to-prd`, `to-issues` (`mattpocock/skills`). `to-features` is bundled. If a prerequisite is missing, say so and stop rather than improvising.
