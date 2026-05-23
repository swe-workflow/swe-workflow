---
name: log-decisions
description: Maintain a running DECISIONS.md markdown file at the repo root — an append-only journal of consequential calls the spec didn't settle. Decide-and-log when the repo grounds the call, log a best-guess assumption when it's reversible, or escalate when only a human can decide. Use whenever you (or the user) make a non-trivial call worth a durable record — an under-specified requirement, a tradeoff, a deviation, a hard-to-undo action, or any call someone would want to review.
---

# log-decisions

`DECISIONS.md` (repo root) is an append-only **decision journal** — the durable answer to *"what was decided here, and why?"* When the spec didn't settle a consequential call, append one entry recording how you resolved it; when only a human can decide, **escalate** instead.

## When to log, and how to decide

Significance is the gate: log a call the spec didn't settle, where someone had to exercise judgment. *If the spec already authorized it, it's not journal-worthy; if you had to invent the authorization, it is.* Skip routine, reversible, already-authorized choices. **When in doubt, log.**

**Look before you ask.** Most "open" questions are already answered in the repo — search codebase conventions, configs, neighbouring code, git history, and the PRD / ADRs / `CONTEXT.md` *before* you treat one as needing a human. Premature escalation is the most common failure; looking first dissolves most of it.

Then classify on two axes — **determinable** (did research settle it?) and **reversible** (cheap to undo?) — and act:

| | Reversible | Irreversible |
|---|---|---|
| **Determinable** (repo / docs / convention) | decide and proceed; log + cite the artifact if it crosses the bar | decide + cite, **log, then verify** the result did what the artifact intended |
| **Needs human context** | pick a safe default, **log it (`assumed`), and proceed** | **stop — escalate and ask** |

**Hard floor:** the *catastrophic* irreversible subset — data loss, destructive migration, irreversible spend, breaking a public interface — **escalates even when determinable.** Nothing unrecoverable happens unattended.

**Picking a default** (bottom-left, when research came up empty): match the existing pattern · prefer the standard, least-surprising option · prefer the choice cheapest to reverse. A grounded default, logged and reviewable, beats a blocking question — and when several open questions pile up, **batch them into one ask**, don't ask twelve times.

Categories (a descriptive label on each entry): `gate-resolution` (answered an open question) · `irreversible-action` (a hard-to-undo step) · `deviation` (changed existing behavior/plan) · `tradeoff` (chose X over Y at a cost).

## Defer: assume vs escalate

- **Assume** — reversible & needs human context. Write `Outcome: assumed`, `Chosen:` your default, `Justification:` the default rule you applied, and **proceed**. It's a settled entry flagged for async review (it promotes, and surfaces in the PR body) — not a blocker.
- **Escalate** — irreversible & needs human context, or the catastrophic floor. Write `Outcome: escalated`, `Chosen: —`, `Justification:` *why only a human can decide*, and **pause** for a human (the HITL pause in a build; just ask, interactively). It stays open — surfaced by `/status`, not promoted until resolved. Resolving escalations is the broader workflow's job.

## The entry

One append-only `##` block in `DECISIONS.md` at the repo root. *(Inside a `/ship` build, append to the worktree's `DECISIONS.staged.md` instead — the build promotes settled entries to `DECISIONS.md` at close-out.)* Create the file with an `<!-- AI-maintained, append-only -->` header if absent. **Never edit, reorder, or delete existing entries.**

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

`context` = an issue ref (`auth/02`, `#57`) or a session tag (`interactive/<topic>`, `grill-me/<topic>`).

**Dedup / revise / reuse.** Before appending, check for an existing entry with the same `(context, Question)`: same `Chosen` → do nothing (retries don't duplicate); changed `Chosen` → append a new entry with `Supersedes:` (never edit the original). A prior entry for the same question is itself a valid citation — `grep` for it rather than re-deciding; never bulk-load the journal.

## Content discipline

`DECISIONS.md` is committed and may be read back, so keep every entry safe: **paraphrase** (your own words); **cite by reference, not payload** (point to `PRD §Security`, never paste issue bodies, web content, or untrusted text); **never log secrets** (tokens, credentials, PII — reference, don't reproduce).

## Examples

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

A reversible call the spec left open — a logged assumption, so the build proceeds and the human reviews later:

```
## 2026-05-22T14:05:00-07:00 — auth/02 — gate-resolution

**Question:** What should the password-reset confirmation page say?
**Options considered:** terse / friendly-with-support-link
**Chosen:** friendly-with-support-link.
**Decided-by:** agent
**Justification:** No spec; applied default "match existing pattern" — mirrored `app/auth/signup/confirm.tsx`. Reversible copy, flagged for review.
**Outcome:** assumed
**Ref:** (pending)
```
