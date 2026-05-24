# Stage 2 — Enumerate features → FEATURES.md

Split the project into **coarse-grained, user-facing features** → `FEATURES.md`. This is the **product-manager** stage of the spec layer: it answers *what should the product do?* at the project level — the big capabilities, in user language (`user-can-…`, `admin-can-…`) — not *how* any one of them is built. It sits **after** the stage-1 domain grill (which grounds `CONTEXT.md`/ADRs) and **before** [`/grill-feature`](grill-feature.md), which grills and specs each feature in depth. Each feature is recorded as a **rich detail block, not a bare slug**, so `/grill-feature` starts from real context.

This is **stage 2 of the [spec layer](spec.md)**: `/spec` runs it inline as part of stages 1–4. On Claude Code it also runs standalone as `/swe-workflow:to-features`; on other agents, invoke the `swe-workflow` skill and ask to enumerate the project's features.

## How it works — a high-level grill, then coarse features

`to-features` is a PM, not an engineer, and its mechanism is to **invoke `/grill-with-docs` at a high (product) level** — the big "who are the actors, what are the major jobs and capabilities, where does v1 start and stop?" questions — then turn the answers into a **coarse** feature list. It grills rather than just reading files because:

- `CONTEXT.md` is a glossary and `docs/adr/` are sparse architectural calls — **neither enumerates features.** The feature set has to be *elicited*, not read off.
- The distinct **stage-1 project grill** that runs first grounds *domain language*; `to-features`' grill builds on it (ideally continuing that conversation, for context) but aims at *features*.

Keep the *granularity* **coarse** — one big user-facing capability per feature (`user-can-post-photo`), never the fine detail of any one (that's `/grill-feature`'s job). But keep the *context* **rich** — capture what the grill surfaced about each feature (the [File format](#file-format) fields). Granularity sharpens downstream:

```
stage 1   /grill-with-docs ─► CONTEXT.md + docs/adr/        (domain grounding)
              │
              ▼
stage 2   /to-features (PM — high-level grill) ─► FEATURES.md   (coarse features, each a rich block)
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
3. **Capture each coarse feature as a block** — a `- [ ]` header line (kebab-case slug + one-line summary, actor-first, behavioral not implementation — `user-can-reset-password`, never `add-bcrypt-hashing`) **plus an indented detail block** recording what the grill surfaced: actor, value, rough scope, dependencies, open questions, constraints/conventions. Coarse *granularity*, rich *context* — see [File format](#file-format).
4. **Surface the list** for a last look, then **write `FEATURES.md`** at the repo root.
5. *(Optional, local-markdown teams)* `mkdir -p .scratch/<feature-slug>/issues` per pending feature to stub the layout downstream stages fill.

**Idempotent** — if `FEATURES.md` already exists, this is a **refinement**: add features and enrich existing blocks against the latest understanding; never overwrite shipped (struck-through) features or discard captured context.

## AFK-friendly and pausable

Like the rest of the spec layer, the high-level grill is **AFK-friendly**: it offers a recommended answer to each question and applies the **[`log-decisions`](../../log-decisions/SKILL.md)** rules (decide / assume) to keep moving when you're away, but **pauses and escalates** on an unsure HITL call (e.g. committing the v1 scope a stakeholder owns) rather than guess.

**Record feature-scope decisions.** A coarse-feature call someone would want to review — **including or dropping** a feature, **splitting or merging**, **deferring to vNext** — is journal-worthy: log it via `log-decisions` (`gate-resolution` / `tradeoff`), citing the grill / `CONTEXT.md` / ADR that grounded it. **Rejecting** a feature → write the reason to `.out-of-scope/<concept>.md` *and* log a `deviation` / `tradeoff` pointing at it.

## File format

Each feature is a `- [ ]` **header line** + an **indented detail block** capturing what the high-level grill surfaced — the seed `/grill-feature` reads. The fields below are a **guide**: fill what the grill produced, skip what doesn't apply, but don't thin a feature back to a one-liner.

```markdown
# Features

<!-- One `- [ ]` header line per feature + an indented detail block (filled from the high-level grill). On ship: strike the header line, add a shipped ref, and KEEP the block. Never delete a feature. -->

- [ ] **user-can-reset-password** — a user can reset a forgotten password
  - **Actor:** a registered user who has forgotten their password
  - **Value:** regain account access without contacting support
  - **Behavior:** request a reset by email → time-limited link → set a new password
  - **Scope (v1):** email reset link; single-use, short-lived token
  - **Out / later:** SMS reset, security questions
  - **Depends on:** `user-can-sign-up` (accounts must exist first)
  - **Open questions:** token TTL (15 min vs 1 h)? reset-request rate limit?
  - **Grill notes:** OWASP short-lived-token convention; mirror the existing `signup/confirm` copy; see `docs/adr/0007-auth.md`

- [ ] **user-can-post-photo** — a user can upload and share a photo
  - **Actor:** a signed-in user
  - **Value:** the core sharing loop
  - **Behavior:** pick an image → add a caption → publish to the feed
  - **Depends on:** `user-can-sign-up`
  - **Open questions:** max file size? allowed formats? moderation in v1?

## Out of scope (rejected)

- ~~**user-can-stream-video**~~ — rejected for v1 (see `.out-of-scope/video-streaming.md`)
```

The header line stays terse and actor-first (it's what `/status` and the chain scan); the block holds the depth. Slugs are kebab-case and unique — they're how `/grill-feature` and issues reference a feature.

## Discipline: when a feature ships, strike it through (don't delete)

Mark the **header line** with strikethrough + a shipped reference (issue number, PR, or tracker ID), and **keep the detail block** beneath as the shipped record — never delete the feature. Strikethrough + retained block preserves:

- **Institutional memory** — what HAS shipped *and the context behind it*, not just what's pending.
- **Traceability** — link from `FEATURES.md` to the issue/PR that completed it.
- **Drift resistance** — you can't quietly drop a feature; dropping requires explicitly marking it out of scope (and writing the reason to `.out-of-scope/`).

```
- [ ] **user-can-reset-password** — a user can reset a forgotten password
  - **Actor:** … (detail block)
                              ↓   strike the header on ship; keep the block
- [x] ~~**user-can-reset-password**~~ — ~~a user can reset a forgotten password~~ (shipped: #42)
  - **Actor:** … (block preserved as the record)
```

## Why this stage exists

mattpocock's chain (`/to-prd`, `/to-issues`, `/triage`) is engineering-side — it assumes the feature set already exists. `/to-prd` deliberately **doesn't interview**; it synthesizes from a conversation. So the work of *eliciting what the features are* has no home in that chain. `to-features` is that home: the **PM stage** that grills the project at a high level and writes the coarse backlog that `/grill-feature` then specs one at a time.

It reads engineering artifacts (`CONTEXT.md`, ADRs) but its output (`FEATURES.md`) is user-facing product language — the deliberate seam where product reasoning enters the chain.

## When to skip

If your team enumerates features in an external tool — Linear projects, Notion, Productboard, a spreadsheet, anything — skip this stage, keep that as the source of truth, and pass features straight to `/grill-feature`. It fills the seam for teams without a product-backlog convention, or solo devs who'd rather keep feature tracking in the repo alongside the code.

## Prerequisites

Not bundled: the `grill-with-docs` skill (`mattpocock/skills`), which this stage drives at a high (product) level. If it's missing, say so and stop rather than improvising.
