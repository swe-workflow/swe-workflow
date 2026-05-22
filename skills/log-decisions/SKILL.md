---
name: log-decisions
description: Record a consequential decision to the repo-root DECISIONS.md journal. Use whenever you (or the user) make a non-trivial decision worth a durable record — resolving an under-specified requirement, choosing between meaningful alternatives, deviating from the plan, or making a call someone would want to review — appending a structured, append-only entry of what was decided, by whom, and why.
---

# log-decisions

Maintain `DECISIONS.md` at the repo root — an append-only **decision journal** of consequential decisions made during work on this repo, by agent or human. When such a decision is made, append one entry recording the question, the options, the choice, who decided, and the justification.

It is the durable, reviewable answer to *"what was decided here, and why?"* — a record that would otherwise vanish when the work is done.

## When to log

Log a **consequential** decision — one the spec didn't already settle, where someone had to exercise judgment: an under-specified requirement resolved, a tradeoff made, a deviation from the plan, or a call worth reviewing. Do **not** log routine, reversible, already-authorized choices (variable names, obvious refactors) — those are noise.

When in doubt, log: an entry is cheap; a missing record is not.

> The precise bar for "consequential," and how to **escalate** a decision you cannot justify, are layered on by the broader workflow. This skill establishes the entry format and the write path; here, every logged decision was made and acted on (`Outcome: applied`).

## Entry schema

One append-only `##` block per decision:

```
## <ISO-8601 timestamp> — <context> — <category>

**Question:** <the decision point, in your own words>
**Options considered:**
- <option 1>
- <option 2>
**Chosen:** <the full chosen answer>
**Decided-by:** agent | human
**Justification:** <the artifact that justifies it, cited by reference — or the rationale>
**Outcome:** applied
**Ref:** <commit / PR / issue, or "(pending)">
```

- **timestamp** — ISO 8601 with offset, e.g. `2026-05-22T13:40:00-07:00`.
- **context** — where it happened: an issue ref (`auth/02`, `#57`), or `interactive/<topic>`, `grill-me/<topic>`.
- **category** — one of `gate-resolution`, `irreversible-action`, `deviation`, `tradeoff`, `FYI`, `grill-self-resolution`. (`escalated`/`resolved` are *outcomes*, not categories — they arrive with escalation handling.)
- **Options considered** — omit for open-ended notes (e.g. an `FYI`).
- **Decided-by** — `human` if a person made the call (even when you surfaced the options); `agent` if you decided on your own.
- **Justification** — paraphrase and **cite by reference** (point to the artifact; don't paste it); never include secrets.
- **Outcome** — `applied`: the decision was made and acted on.

## Writing the entry (direct-append)

1. If `DECISIONS.md` does not exist at the repo root, **create it** with this header as the first line, followed by a blank line:
   ```
   <!-- Decision journal — consequential decisions made on the user's behalf. AI-maintained, append-only. -->
   ```
2. **Append** the entry block to the end of the file, separated from the previous content by one blank line.
3. **Append only.** Never edit, reorder, or delete existing entries — the journal is immutable history.

## Example

```
<!-- Decision journal — consequential decisions made on the user's behalf. AI-maintained, append-only. -->

## 2026-05-22T13:40:00-07:00 — auth/02 — gate-resolution

**Question:** How long should password-reset tokens stay valid?
**Options considered:**
- 15 minutes (more reset friction)
- 1 hour (OWASP-recommended balance)
- 24 hours (larger attack window)
**Chosen:** 1 hour.
**Decided-by:** agent
**Justification:** PRD `.scratch/auth/PRD.md §Security` says "short-lived" without a number; took the 1h OWASP convention.
**Outcome:** applied
**Ref:** (pending)
```

See `scenarios.md` for the behavioral cases this skill must satisfy.
