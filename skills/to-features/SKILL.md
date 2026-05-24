---
name: to-features
description: Enumerate a product's user-facing features into FEATURES.md at the repo root. Run it in the same conversation as /grill-with-docs (stage 1), because CONTEXT.md and docs/adr/ don't capture every feature — to-features rides the live grill context and runs its own intensive interview to surface the full set. It offers a recommended answer to every question and follows the log-decisions AFK rules — decide, assume, or escalate — recording feature-scope calls in the decision journal. Use after the domain grill and before /to-prd specs each feature.
---

# to-features

Enumerate a product's **user-facing features** into `FEATURES.md` at the repo root — the seam between domain understanding (`/grill-with-docs`, upstream) and per-feature specs (`/to-prd`, downstream, one feature → one PRD). The output is a list, in user language, of what the product does: `user-can-…`, `admin-can-…`, `guest-can-…`.

## Run it in the same conversation as the grill (stage 1)

`/to-features` is **not** a cold read of two files. `CONTEXT.md` is a glossary — "totally devoid of implementation details" by design — and `docs/adr/` records only the sparse, hard-to-reverse architectural calls. **Neither captures the full feature set.** The understanding of *what features the product needs* is built during the stage-1 grill and lives in that **conversation**, not in the artifacts it leaves behind.

So invoke `/to-features` **right after `/grill-with-docs`, in the same conversation**, while that context is live. The chain runs the two back-to-back (`/swe-workflow:spec` stages 1→2) for exactly this reason.

> Running cold — a fresh conversation, only the docs to read? Say so. You've lost the grill context and will recover less from the artifacts alone; lean harder on the interview below, and consider re-running `/grill-with-docs` first to rebuild shared understanding.

The mental model — project-wide grounding feeds a per-feature flow, and `to-features` is the hinge:

```
/grill-with-docs ─► CONTEXT.md + docs/adr/     (project-wide, durable)
       │
       ▼  same conversation
/to-features     ─► FEATURES.md                (the feature set)
       │
       ▼  per feature, one line at a time
/to-prd ─► PRD ─► /to-issues ─► issues
```

The three stages interview about different things: `/grill-with-docs` about **domain language**, `/to-features` about **the feature set**, and `/to-prd` about **nothing** — it doesn't interview, it synthesizes one feature's PRD from the conversation. The elicitation `/to-prd` skips has to happen before it: grill (language) **+ to-features (features)**.

## Process

1. **Ground yourself, in order** — the live grill conversation first, then `CONTEXT.md` + `docs/adr/*.md`, then an existing `FEATURES.md` if present (this run is then a refinement — see below).
2. **Draft** the user-facing features the domain implies — one per line, kebab-case slug + one-line description, actor-first (`user-can-…`), behavioral not implementation (`user-can-reset-password`, never `add-bcrypt-hashing`).
3. **Interview intensively** to complete and sharpen the draft — see [The interview](#the-interview). Stay in your lane: the *feature set*, not domain language (the grill's job) and not any one feature's spec (`/to-prd`'s job).
4. **Surface the final list** for a last look, then **write `FEATURES.md`** at the repo root.
5. *(Optional, local-markdown teams)* `mkdir -p .scratch/<feature-slug>/issues` per pending feature to stub the layout downstream stages fill.

**Idempotent** — if `FEATURES.md` already exists, this is a **refinement**: propose additions and strikethroughs against the latest understanding; never overwrite shipped (struck-through) lines.

## The interview

`/to-features` is an **AFK-friendly** spec interview: it draws the feature set out of you by asking, but offers a recommended answer to every question and keeps moving via the `log-decisions` rules when you're away — pausing only on an unsure call that needs you (see below). Walk the feature tree one question at a time — the `/grill-with-docs` cadence, aimed at features instead of vocabulary. Cover, at least:

- **Coverage** — is every actor in `CONTEXT.md` accounted for? For each, are the core flows present (create / read / update / delete-shaped gaps, auth, onboarding, settings, the unhappy paths)?
- **Boundaries** — is a given line one feature or several? Split when a slug bundles unrelated value; merge when two lines are really the same slice.
- **In or out** — which candidates are *not* v1? Rejected ones go to `.out-of-scope/`; deferred ones to a "Later / vNext" note.
- **Ordering** — does any feature have to ship before another can (foundational vs dependent)?

This is **enumeration grounded in the grill**, not product discovery: you draw the feature set out of the shared understanding already built (plus whatever product knowledge the user brings to the conversation). Discovering *new* product direction — market research, user studies, strategy — still lives outside the toolchain.

### Recommended answers, and the `log-decisions` rules

The interview is **AFK-friendly** — it neither stalls nor over-asks, and it can run without you in the room — held by two mechanisms:

- **A recommended answer with every question** — accept it fast, or override it.
- **The `log-decisions` decide / assume / escalate rules** (the same discipline `/tdd` uses unattended in execution) decide how each call is handled:
  - **Determinable** (the grill, `CONTEXT.md`, ADRs, or convention settle it) → don't ask; decide and proceed.
  - **Reversible but open** → take the recommended default, **log an assumption** (`Outcome: assumed`), and proceed — flagged for async review.
  - **Unsure and HITL** — irreversible, needs your context (e.g. committing the v1 scope a stakeholder owns) → **pause and escalate**, and wait for you (batch several into one ask). *This is the spec/execution line:* in `/ship` and `/ship-all` the same escalation is instead **persisted for batch review** and the run continues — here, in spec, it pauses.

The full 2×2 (determinable × reversible) and the catastrophic floor live in the **[`log-decisions` skill](../log-decisions/SKILL.md)** — apply them; don't restate them.

### Record feature-scope decisions

A feature-set call someone would want to review is journal-worthy. Record it via the **[`log-decisions` skill](../log-decisions/SKILL.md)** as you make it:

- **including or dropping** a feature from v1, **splitting or merging**, or **deferring to vNext** → a `gate-resolution` or `tradeoff` entry; cite the grill / `CONTEXT.md` / ADR that grounded it.
- **rejecting** a feature → write the reason to `.out-of-scope/<concept>.md` *and* log a `deviation` / `tradeoff` entry pointing at it.

In the spec layer you're not inside a ship worktree, so entries append to `DECISIONS.md` directly (in a worktree they'd stage to `DECISIONS.staged.md`). Use `log-decisions`' entry schema and dedup rules — don't reinvent them here.

## File format

```markdown
# Features

<!-- One feature per line. Strike through completed features; don't delete. -->

- [ ] user-can-sign-up — A user can create an account with email + password
- [ ] user-can-log-in — A user can sign in to their account
- [x] ~~user-can-reset-password~~ — ~~A user can reset a forgotten password~~ (shipped: #42)
- [ ] user-can-post-photo — A user can upload and share a photo

## Out of scope (rejected)

- ~~user-can-stream-video~~ — Rejected for v1 (see `.out-of-scope/video-streaming.md`)
```

## Discipline: when a feature ships, strike it through (don't delete)

Mark its line with strikethrough + a shipped reference (issue number, PR, or tracker ID); never delete it. Strikethrough preserves:

- **Institutional memory** — what HAS shipped, not just what's pending.
- **Traceability** — link from `FEATURES.md` to the issue/PR that completed it.
- **Drift resistance** — you can't quietly drop a feature; dropping requires explicitly marking it out of scope (and writing the reason to `.out-of-scope/`).

```
- [ ] user-can-reset-password — A user can reset a forgotten password
                              ↓
- [x] ~~user-can-reset-password~~ — ~~A user can reset a forgotten password~~ (shipped: #42)
```

## Why this skill exists

mattpocock's chain (`/to-prd`, `/to-issues`, `/triage`) is engineering-side — it assumes the feature set already exists. `/to-prd` deliberately **doesn't interview**; it synthesizes one feature from the conversation. So the work of figuring out *what the features are* has no home in that chain. `/to-features` is that home: an interview, run in the grill's conversation, that turns domain understanding into an enumerated, user-language backlog `/to-prd` can then take one line at a time.

It's **engineering-adjacent** — it reads engineering artifacts (`CONTEXT.md`, ADRs) and rides the grill conversation, but its output (`FEATURES.md`) is in user-facing language. The deliberate seam where product reasoning enters the chain.

## When to skip

If your team enumerates features in an external tool — Linear projects, Notion, Productboard, a spreadsheet, anything — skip `/to-features`, keep that as the source of truth, and pass features straight to `/to-prd`. This skill fills the seam for teams without a product-backlog convention, or solo devs who'd rather keep feature tracking in the repo alongside the code.
