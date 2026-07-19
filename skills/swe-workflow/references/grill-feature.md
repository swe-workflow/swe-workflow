# Stage 3 — Grill a feature → PRD

Spec a **single feature** into a PRD — the **product-manager** stage. Grill that one feature, then synthesize its PRD.

This is **stage 3 of the [spec layer](spec.md)**: `/spec` runs it inline for the feature it's targeting; `/grill-feature` runs it **standalone, once per feature**. It pairs a feature-scoped `grill-with-docs` interview with `to-spec` — the grill supplies the conversation `to-spec` synthesizes from (on its own, `to-spec` doesn't interview).

**The feature** comes from the user's request — a slug from `FEATURES.md` (e.g. `user-can-reset-password`), which carries the detail block `to-features` captured. If none is given, pick an un-specced (not struck-through) feature from `FEATURES.md`; ask which when ambiguous — don't guess.

**Product, not engineering.** Like `to-features`, this is a PM stage: it defines *what* the feature does (the PRD), not *how* it's built. Architecture and the build plan come later, in `/ship`.

## Procedure

1. **Start from the feature's block in `FEATURES.md`** — read the slug's detail block (actor, value, scope, dependencies, open questions, grill notes) that `to-features` captured. That rich seed is your starting context: you pick up where the high-level grill left off, not from a bare slug. Then **invoke `grill-with-docs` scoped to this one feature**, grounded on that block plus `CONTEXT.md` + `docs/adr/` — an **intensive, feature-level interview** (sharper and deeper than `to-features`' high-level grill) that resolves the block's open questions and nails the feature's behavior, edge cases, and boundaries one branch at a time. It may refine `CONTEXT.md`/ADRs. Loop until no questions remain or you call it.
2. **Synthesize the PRD** — at the end of that conversation, invoke `to-spec`. It turns the grill conversation into **one PRD for this feature**, published per the active tracker and labeled `ready-for-agent`. **One feature → one PRD.** Suite constraint on the output: **no file paths or line numbers in the PRD** — they rot; describe interfaces and behavior instead.

**Idempotent** — **skip** if a PRD already exists for this feature (don't create a duplicate parent). Re-run only to refine a PRD whose feature genuinely changed.

**AFK-friendly and pausable** — the spec-layer posture ([spec.md](spec.md)): the grill offers recommended answers and applies the `log-decisions` rules to proceed on determinable/reversible calls (recording them), but an **unsure HITL call pauses** and asks rather than guessing. Bar-crossing feature decisions are journaled via `log-decisions`.

**Next:** slice the PRD into tracer-bullet issues (stage 4 of [spec.md](spec.md)), then `/ship` or `/ship-all`.

**Prerequisites** (not bundled): the `grill-with-docs` and `to-spec` skills (`mattpocock/skills`). If a prerequisite is missing, say so and stop rather than improvising.
