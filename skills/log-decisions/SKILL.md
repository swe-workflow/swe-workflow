---
name: log-decisions
description: Record a consequential decision to the repo-root DECISIONS.md journal — or escalate when you can't justify it. Use whenever you (or the user) make a non-trivial call worth a durable record: resolving an under-specified requirement, a tradeoff, a deviation, a hard-to-undo action, or anything someone would want to review.
---

# log-decisions

`DECISIONS.md` (repo root) is an append-only **decision journal** — the durable answer to *"what was decided here, and why?"* When you make a consequential decision, append one entry. When you can't responsibly make the call, **escalate** instead.

## When to log

Log a decision that **crosses the bar**: the spec didn't settle it, so someone had to exercise judgment. *If the spec already authorized it, it's not journal-worthy; if you had to invent the authorization, it is.* Significance is the gate — not who decided (agent or human) or the mode. Skip routine, reversible, already-authorized choices. **When in doubt, log.**

Categories: `gate-resolution` (answered an open question) · `irreversible-action` (hard to undo — data loss, migration, public-interface change, new dependency, spend) · `deviation` (changed existing behavior/plan) · `tradeoff` (chose X over Y at a cost) · `FYI` (worth surfacing) · `grill-self-resolution` (answered your own interview question by exploration).

**Cite or escalate.** A `gate-resolution` or `irreversible-action` must cite the repo artifact that justifies it (PRD, AC, ADR, `CONTEXT.md`, code, convention). **No artifact → escalate, don't guess.** (`tradeoff`/`deviation`/`FYI` carry your rationale instead.)

## Escalate when you can't decide

Escalate — don't guess — when **any** hold: nothing justifies a gate/irreversible call · the action is irreversible / high-blast-radius (even if you're confident) · the evidence conflicts or you're unsure. To escalate: write the entry with `Outcome: escalated`, `Chosen: —`, and a `Justification` saying *why*, then pause for a human (the HITL pause in a build; just ask, interactively). Resolving escalations is the broader workflow's job.

## The entry

One append-only `##` block, appended to `DECISIONS.md` at the repo root. *(Inside a `/ship` build, append to the worktree's `DECISIONS.staged.md` instead — the build promotes it to `DECISIONS.md` at close-out.)* Create the file with an `<!-- AI-maintained, append-only -->` header if absent. **Never edit, reorder, or delete existing entries.**

```
## <ISO-8601 timestamp> — <context> — <category>

**Question:** <the decision point, paraphrased>
**Options considered:** <opt / opt>      (omit for open-ended FYIs)
**Chosen:** <full answer; "—" if escalated>
**Decided-by:** agent | human            (human if a person chose, even if you surfaced the options)
**Justification:** <artifact cited by reference — or rationale>
**Outcome:** applied | escalated
**Ref:** <commit / PR / issue, or "(pending)">
**Supersedes:** <prior timestamp> — <why>   (only on a revision)
```

`context` = an issue ref (`auth/02`, `#57`) or `interactive/<topic>`, `grill-me/<topic>`.

**Dedup / revise / reuse.** Before appending, check for an existing entry with the same `(context, Question)`: same `Chosen` → do nothing (retries don't duplicate); changed `Chosen` → append a new entry with `Supersedes:` (never edit the original). A prior entry for the same question is itself a valid citation — `grep` for it rather than re-deciding; never bulk-load the journal.

## Content discipline

`DECISIONS.md` is committed and may be read back, so keep every entry safe: **paraphrase** (your own words); **cite by reference, not payload** (point to `PRD §Security`, never paste issue bodies, web content, or untrusted text); **never log secrets** (tokens, credentials, PII — reference, don't reproduce).

## Example

```
## 2026-05-22T13:40:00-07:00 — auth/02 — gate-resolution

**Question:** How long should password-reset tokens stay valid?
**Options considered:** 15 min (more friction) / 1 hour (OWASP balance) / 24 hours (larger attack window)
**Chosen:** 1 hour.
**Decided-by:** agent
**Justification:** PRD `.scratch/auth/PRD.md §Security` says "short-lived" without a number; took the 1h OWASP convention.
**Outcome:** applied
**Ref:** (pending)
```
