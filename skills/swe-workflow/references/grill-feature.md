# Stage 3 — Grill a feature → PRD

Spec a **single feature** into a PRD — the **product-manager** stage. Grill that one feature, then synthesize its PRD. On Claude Code this runs as `/swe-workflow:grill-feature`; on other agents, invoke the `swe-workflow` skill and ask to grill a feature into its PRD.

This is **stage 3 of the [spec layer](spec.md)**: `/spec` runs it inline for the feature it's targeting; `/grill-feature` runs it **standalone, once per feature**. It pairs a feature-scoped `grill-with-docs` interview with `to-prd` — the grill supplies the conversation `to-prd` synthesizes from (on its own, `to-prd` doesn't interview).

**The feature** comes from the user's request — a slug from `FEATURES.md` (e.g. `user-can-reset-password`), which carries the detail block `to-features` captured. If none is given, pick an un-specced (not struck-through) feature from `FEATURES.md`; ask which when ambiguous — don't guess.

**Product, not engineering.** Like `to-features`, this is a PM stage: it defines *what* the feature does (the PRD), not *how* it's built. Architecture and the build plan come later, in `/ship` (`planning-with-files` + `tdd`).

## Procedure

1. **Start from the feature's block in `FEATURES.md`** — read the slug's detail block (actor, value, scope, dependencies, open questions, grill notes) that `to-features` captured. That rich seed is your starting context: you pick up where the high-level grill left off, not from a bare slug. Then **invoke `grill-with-docs` scoped to this one feature**, grounded on that block plus `CONTEXT.md` + `docs/adr/` — an **intensive, feature-level interview** (sharper and deeper than `to-features`' high-level grill) that resolves the block's open questions and nails the feature's behavior, edge cases, and boundaries one branch at a time. It may refine `CONTEXT.md`/ADRs. Loop until no questions remain or you call it.
2. **Synthesize the PRD** — at the end of that conversation, invoke `to-prd`. It turns the grill conversation into **one PRD for this feature** (Problem / Solution / User Stories / Implementation + Testing Decisions / Out of Scope), published per the active tracker and labeled `ready-for-agent`. **One feature → one PRD.**

**Idempotent** — **skip** if a PRD already exists for this feature (don't create a duplicate parent). Re-run only to refine a PRD whose feature genuinely changed.

**AFK-friendly and pausable** — the spec-layer posture ([spec.md](spec.md)): the grill offers recommended answers and applies the `log-decisions` rules to proceed on determinable/reversible calls (recording them), but an **unsure HITL call pauses** and asks rather than guessing. Bar-crossing feature decisions are journaled via `log-decisions`.

**Runs per feature, not necessarily one conversation each.** Each feature gets its own PRD; whether you spec several in one sitting or one at a time is up to you.

**Next:** slice the PRD into tracer-bullet issues with `to-issues` (stage 4, separate), then `/ship` or `/ship-all`.

**Prerequisites** (not bundled): the `grill-with-docs` and `to-prd` skills (`mattpocock/skills`). If a prerequisite is missing, say so and stop rather than improvising.
