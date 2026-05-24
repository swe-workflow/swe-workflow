# Stages 1–4 — Spec

Run the **spec layer** — stages 1 → 2 → 3 → 4 — turning an idea into a `ready-for-agent` backlog. The mirror image of the **ship** stage ([`ship.md`](ship.md), stages 5–7): **spec produces the issues, ship builds them.** On Claude Code this runs as `/swe-workflow:spec`; on other agents, invoke the `swe-workflow` skill and ask to spec a feature.

**Focus** comes from the user's request — optional. A feature slug from `FEATURES.md` specs that one; a raw idea/topic starts from scratch; nothing given works from the conversation, or from the un-specced features already in `FEATURES.md`.

Per-stage detail (including when to skip each) is in [REFERENCE.md](../REFERENCE.md).

**AFK-friendly — but it pauses for you.** `grill-with-docs`, `to-features`, and `to-issues` interview you intensively, yet each is AFK-friendly: when you're away it offers recommended answers and applies the `log-decisions` rules to decide or assume the determinable and reversible calls, recording them in the journal. What it **won't** settle alone is an **unsure HITL call** (irreversible, needs your context) — there it **pauses and escalates**, and waits. That's the line to execution: **spec pauses on an unsure HITL call; `/ship` and `/ship-all` ([`ship.md`](ship.md), [`ship-all.md`](ship-all.md)) persist it for your batch review and keep going** — same `log-decisions` rules in both, opposite reaction to the one call only you can make.

## Idempotent by design — do only what's missing
Detect where the feature already is in the spec layer and resume; never clobber or duplicate. Each stage states its skip condition; run only the stages whose artifact is absent or stale.

## Stage 1 — Grill the domain (`grill-with-docs` skill)
Resolve the domain language → `CONTEXT.md` (+ ADRs under `docs/adr/` for substantive decisions).
- **Skip if** terminology is settled — `CONTEXT.md` exists and a grill pass surfaces no new questions.
- Otherwise invoke `grill-with-docs`. It's re-runnable — each pass **sharpens** `CONTEXT.md` (and may add ADRs); it refines, never duplicates. Loop until no questions remain or the user calls it.

## Stage 2 — Enumerate features (`to-features` skill)
**Run in the same conversation as stage 1** — `CONTEXT.md` + ADRs don't capture every feature, so `to-features` rides the live grill context and **interviews** you to complete the set, then writes `FEATURES.md`. It offers recommended answers, follows `log-decisions`' AFK rules (proceed on safe defaults, escalate what needs you), and journals feature-scope calls.
- **Idempotent**: if `FEATURES.md` exists, this is a **refinement** — propose additions / strikethroughs; never overwrite shipped (struck-through) lines.

## Stage 3 — Write the PRD (`to-prd` skill)
For the target feature, synthesize a PRD (Problem / Solution / User Stories / Implementation + Testing Decisions / Out of Scope), published as a parent issue labeled `ready-for-agent`.
- **Skip if** a PRD already exists for this feature — don't create a duplicate parent (re-run `to-prd` only to refine a PRD whose feature genuinely changed).
- If no feature is targeted yet, pick from `FEATURES.md`; ask the user which when it's ambiguous.

## Stage 4 — Slice into issues (`to-issues` skill)
Break the PRD into **tracer-bullet** issues (thin vertical slices through every layer), each labeled `ready-for-agent`, with `blocked-by` chains and HITL/AFK tags.
- **Skip if** the PRD is already sliced (issues exist with it as parent) — re-slicing forks state. To refine an existing slice, send it back through `triage`; don't re-run `to-issues` on a started PRD.

## Done
You now have a `ready-for-agent` backlog. Hand off to **ship** ([`ship.md`](ship.md), one issue) or **ship-all** ([`ship-all.md`](ship-all.md), the batch).

**Prerequisites** (not bundled): the `grill-with-docs`, `to-prd`, `to-issues` skills (`mattpocock/skills`). `to-features` is part of this suite. If a prerequisite is missing, say so and stop rather than improvising.
