---
name: to-features
description: Split the project into coarse-grained, user-facing features → FEATURES.md. This is the product-manager step — to-features invokes /grill-with-docs at a high level to elicit the major capabilities, not read them off CONTEXT.md/ADRs (which don't enumerate features). Runs after the stage-1 domain grill and before /grill-feature, which grills and specs each feature in depth. AFK-friendly and pausable; records feature-scope calls via log-decisions.
---

# to-features

Split the project into **coarse-grained, user-facing features** → `FEATURES.md`. This is the **product-manager** step of the spec layer: it answers *what should the product do?* at the project level — the big capabilities, in user language (`user-can-…`, `admin-can-…`) — not *how* any one of them is built. It sits **after** the stage-1 domain grill (which grounds `CONTEXT.md`/ADRs) and **before** [`/grill-feature`](../swe-workflow/references/grill-feature.md), which grills and specs each feature in depth.

## How it works — a high-level grill, then coarse features

`to-features` is a PM, not an engineer, and its mechanism is to **invoke `/grill-with-docs` at a high (product) level** — the big "who are the actors, what are the major jobs and capabilities, where does v1 start and stop?" questions — then turn the answers into a **coarse** feature list. It grills rather than just reading files because:

- `CONTEXT.md` is a glossary and `docs/adr/` are sparse architectural calls — **neither enumerates features.** The feature set has to be *elicited*, not read off.
- The distinct **stage-1 project grill** that runs first grounds *domain language*; `to-features`' grill builds on it (ideally continuing that conversation, for context) but aims at *features*.

Keep features **coarse-grained** — one big user-facing capability per line (`user-can-post-photo`), never the fine detail of any one (that's `/grill-feature`'s job, per feature). Granularity sharpens downstream:

```
stage 1   /grill-with-docs ─► CONTEXT.md + docs/adr/        (domain grounding)
              │
              ▼
stage 2   /to-features (PM — high-level grill) ─► FEATURES.md   (coarse features)
              │
              ▼  per feature
stage 3   /grill-feature (PM — feature grill + /to-prd) ─► one PRD
              │
              ▼
stage 4   /to-issues ─► tracer-bullet issues
```

## Process

1. **Ground on stage 1** — read the existing `CONTEXT.md` + `docs/adr/`, and pick up the stage-1 grill conversation if it's still live.
2. **Invoke `/grill-with-docs` at a high level** — drive the product-level interview that surfaces the major capabilities, the actors, and where v1 begins and ends. This *is* the interview; `to-features` doesn't run a separate bespoke one.
3. **Enumerate coarse features** from what the grill surfaced — one per line, kebab-case slug + one-line description, actor-first (`user-can-…`), behavioral not implementation (`user-can-reset-password`, never `add-bcrypt-hashing`), and **coarse** (the fine detail is split out later, in `/grill-feature`).
4. **Surface the list** for a last look, then **write `FEATURES.md`** at the repo root.
5. *(Optional, local-markdown teams)* `mkdir -p .scratch/<feature-slug>/issues` per pending feature to stub the layout downstream stages fill.

**Idempotent** — if `FEATURES.md` already exists, this is a **refinement**: propose additions and strikethroughs against the latest understanding; never overwrite shipped (struck-through) lines.

## AFK-friendly and pausable

Like the rest of the spec layer, the high-level grill is **AFK-friendly**: it offers a recommended answer to each question and applies the **[`log-decisions`](../log-decisions/SKILL.md)** rules to keep moving when you're away — decide what's determinable, take a reversible default and log it as an **assumption** — but it **pauses and escalates** on an unsure HITL call (e.g. committing the v1 scope a stakeholder owns) rather than guess. (In execution that same escalation would *persist for batch review* instead of pausing — see [`spec.md`](../swe-workflow/references/spec.md).)

**Record feature-scope decisions.** A coarse-feature call someone would want to review — **including or dropping** a feature, **splitting or merging**, **deferring to vNext** — is journal-worthy: log it via `log-decisions` (`gate-resolution` / `tradeoff`), citing the grill / `CONTEXT.md` / ADR that grounded it. **Rejecting** a feature → write the reason to `.out-of-scope/<concept>.md` *and* log a `deviation` / `tradeoff` pointing at it.

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

mattpocock's chain (`/to-prd`, `/to-issues`, `/triage`) is engineering-side — it assumes the feature set already exists. `/to-prd` deliberately **doesn't interview**; it synthesizes from a conversation. So the work of *eliciting what the features are* has no home in that chain. `to-features` is that home: the **PM step** that grills the project at a high level and writes the coarse backlog `/grill-feature` then specs one feature at a time.

It reads engineering artifacts (`CONTEXT.md`, ADRs) but its output (`FEATURES.md`) is user-facing product language — the deliberate seam where product reasoning enters the chain.

## When to skip

If your team enumerates features in an external tool — Linear projects, Notion, Productboard, a spreadsheet, anything — skip `to-features`, keep that as the source of truth, and pass features straight to `/grill-feature`. This skill fills the seam for teams without a product-backlog convention, or solo devs who'd rather keep feature tracking in the repo alongside the code.
