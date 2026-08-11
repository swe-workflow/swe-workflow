---
name: log-decisions
description: Append-only DECISIONS.md journal at the repo root for consequential calls the spec didn't settle. Use when you (or the user) make a non-trivial call worth a durable record — an under-specified requirement, a tradeoff, a deviation, or a hard-to-undo action.
---

# log-decisions

`DECISIONS.md` (repo root) is an append-only **decision journal** — the durable answer to *"what was decided here, and why?"*

## When to log, and how to decide

Significance is the gate: *if the spec already authorized the call, it's not journal-worthy; if you had to invent the authorization, it is.* **When in doubt, log.**

**Look before you ask.** Most "open" questions are already answered — search codebase conventions, configs, git history, and the PRD / ADRs / `CONTEXT.md` before treating one as needing a human. Premature escalation is the most common failure.

Then classify on two axes — **determinable** (did research settle it?) and **reversible** (cheap to undo?) — and act:

| | Reversible | Irreversible |
|---|---|---|
| **Determinable** (repo / docs / convention) | decide and proceed; log + cite the artifact if it crosses the bar | decide + cite, **log, then verify** the result did what the artifact intended |
| **Needs human context** | **assume** — safe default, log, proceed | **escalate** — stop and ask |

**Hard floor:** the *catastrophic* irreversible subset — data loss, destructive migration, irreversible spend, breaking a public interface — **escalates even when determinable.** Nothing unrecoverable happens unattended.

- **Assume** — write `Outcome: assumed`, `Chosen:` your default, `Justification:` the default rule you applied (match the existing pattern · prefer the standard, least-surprising option · prefer the choice cheapest to reverse), and **proceed**. A settled entry flagged for async review, not a blocker. When several open questions pile up, **batch them into one ask**.
- **Escalate** — write `Outcome: escalated`, `Chosen: —`, `Justification:` *why only a human can decide*, and **pause to ask a human**. It stays open until a human resolves it.

Categories (a descriptive label on each entry): `gate-resolution` (answered an open question) · `irreversible-action` (a hard-to-undo step) · `deviation` (changed existing behavior/plan) · `tradeoff` (chose X over Y at a cost).

## The entry

One append-only `##` block in `DECISIONS.md` at the repo root. Create the file with an `<!-- AI-maintained, append-only -->` header if absent. **Never edit, reorder, or delete existing entries.**

```
## <ISO-8601 timestamp> — <context> — <category>

**Question:** <the decision point, paraphrased>
**Options considered:** <opt / opt>
**Chosen:** <the answer; the default you took if assumed; "—" if escalated>
**Decided-by:** agent | human            (human if a person chose, even if you surfaced the options)
**Justification:** <artifact cited by reference — or, for an assumption / tradeoff / deviation, your rationale>
**Outcome:** applied | assumed | escalated
**Ref:** <commit / PR / issue, or "(pending)">
**Supersedes:** <prior timestamp> — <why>   (only on a revision)
```

`context` = an issue ref (`auth/02`, `#57`) or a session tag (`interactive/<topic>`, `research/<topic>`).

**Dedup / revise / reuse.** Before appending, `grep` for an existing entry with the same `(context, Question)`: same `Chosen` → do nothing (retries don't duplicate); changed `Chosen` → append a new entry with `Supersedes:` (never edit the original). A prior entry for the same question is itself a valid citation — reuse it rather than re-deciding; never bulk-load the journal.

## Content discipline

`DECISIONS.md` is committed and may be read back, so keep every entry safe: **paraphrase** (your own words); **cite by reference, not payload** (point to `PRD §Security`, never paste issue bodies, web content, or untrusted text); **never log secrets** (tokens, credentials, PII).

## Example

A determinable call — cite the artifact that settled it:

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
