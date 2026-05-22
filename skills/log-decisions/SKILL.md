---
name: log-decisions
description: Record a consequential decision to the repo-root DECISIONS.md journal, or escalate when you can't justify it. Use whenever you (or the user) make a non-trivial decision worth a durable record — resolving an under-specified requirement, choosing between alternatives, deviating from the plan, making a hard-to-undo call, or a call someone would want to review — appending a structured, append-only entry of what was decided, by whom, and why.
---

# log-decisions

Maintain `DECISIONS.md` at the repo root — an append-only **decision journal** of consequential decisions made during work on this repo, by agent or human. When such a decision is made, append one entry recording the question, the options, the choice, who decided, and the justification. When you *can't* responsibly make the call, **escalate** instead.

It is the durable, reviewable answer to *"what was decided here, and why?"* — a record that would otherwise vanish when the work is done.

## When to log

Log a decision that **crosses the bar** — the spec didn't already settle it, so someone had to exercise judgment. The test: *if the spec already authorized it, it's not journal-worthy; if you had to invent the authorization, it is.* The bar gates by **significance**, independent of who decided (agent or human) or the mode (build, interactive, grill).

Bar-crossing decisions fall into one of six **categories**:

| Category | What it is |
|---|---|
| `gate-resolution` | Answered a question the spec left open |
| `irreversible-action` | Did something hard to undo (data loss, migration, public-interface change, new dependency, spend) |
| `deviation` | Changed existing behavior / interface / the plan |
| `tradeoff` | Chose X over Y, accepting a cost |
| `FYI` | A notable fact worth surfacing — "something you should know" |
| `grill-self-resolution` | Answered your own interview question by exploration instead of asking |

Do **not** log routine, reversible, already-authorized choices (variable names, obvious refactors) — those are noise. **When in doubt, log:** an entry is cheap; a missing record is not.

### The citation rule

A `gate-resolution` or `irreversible-action` must **cite the repo artifact that justifies it** (PRD, acceptance criteria, an ADR, `CONTEXT.md`, existing code, a convention). **If nothing in the repo justifies the call, do not guess — escalate.** (`tradeoff` / `deviation` / `FYI` carry your *rationale* instead of a required citation.)

## Entry schema

One append-only `##` block per decision:

```
## <ISO-8601 timestamp> — <context> — <category>

**Question:** <the decision point, in your own words>
**Options considered:**
- <option 1>
- <option 2>
**Chosen:** <the full chosen answer; "—" if escalated>
**Decided-by:** agent | human
**Justification:** <the artifact that justifies it, cited by reference — or the rationale>
**Outcome:** applied | escalated
**Ref:** <commit / PR / issue, or "(pending)">
**Supersedes:** <prior entry's timestamp> — <why it changed>   (omit unless this entry revises an earlier decision)
```

- **timestamp** — ISO 8601 with offset, e.g. `2026-05-22T13:40:00-07:00`.
- **context** — where it happened: an issue ref (`auth/02`, `#57`), or `interactive/<topic>`, `grill-me/<topic>`.
- **category** — one of the six above. (`escalated` is an *outcome*, not a category.)
- **Options considered** — omit for open-ended notes (e.g. an `FYI`).
- **Decided-by** — `human` if a person made the call (even when you surfaced the options); `agent` if you decided on your own.
- **Justification** — see Content discipline below.
- **Outcome** — `applied` (made and acted on) or `escalated` (paused for a human — see below).
- **Supersedes** — include **only on a revision**: the timestamp of the entry this one replaces, and why it changed. The superseded entry is left untouched.

## Escalate when you can't decide

Some calls you must **not** make on your own. Escalate — don't guess — when any of these hold:

1. **No artifact justifies it** — a `gate-resolution` or `irreversible-action` with nothing in the repo to cite.
2. **Irreversible / high blast radius** — data loss, a migration, a public-interface change, a new dependency, spend — even when you feel confident.
3. **Conflicting evidence or low confidence** — the artifacts disagree, or you genuinely can't tell.

To escalate:

- Write an entry with `Outcome: escalated`, `Chosen: —`, and a `Justification` stating *why* you couldn't decide (e.g. "no artifact authorizes destroying this column").
- Then **pause for a human**: in a build, stop and surface it (the HITL pause); in an interactive or grill session, just ask the user.

(Resolving an escalation, and surfacing open ones, is handled by the broader workflow.)

## Content discipline

`DECISIONS.md` is committed and may be read back, so every entry must be safe:

- **Paraphrase** — write the `Question`, `Options`, and `Justification` in your own words.
- **Cite by reference, not payload** — point to the artifact (`PRD §Security`, `src/auth/tokens.ts`); never paste raw issue bodies, fetched web content, or untrusted text.
- **Never log secrets** — no tokens, credentials, keys, or PII. Reference sensitive material; don't reproduce it.

## Writing the entry

**First, dedup by `(context, question)`** — checked when you *act on* the decision, so retries never duplicate:

1. Look for an existing entry with the same `context` and `Question`:
   - **Same `Chosen`** → already logged. **Do nothing** — no duplicate. (This makes 3-strike / TDD-loop retries safe.)
   - **Different `Chosen`** → a **revision**: append a **new** entry with a `Supersedes:` line naming the prior entry's timestamp and why it changed. **Never edit the original.**
   - **No match** → a new decision; continue.
2. If `DECISIONS.md` does not exist at the repo root, **create it** with this header as the first line, followed by a blank line:
   ```
   <!-- Decision journal — consequential decisions made on the user's behalf. AI-maintained, append-only. -->
   ```
3. **Append** the entry block to the end of the file, separated from the previous content by one blank line.
4. **Append only.** Never edit, reorder, or delete existing entries — the journal is immutable history.

## Reusing past decisions (precedent)

Before deciding a question you suspect was settled before, you may look it up — to stay consistent and avoid re-deciding:

- Do a **targeted search**: `grep` `DECISIONS.md` for the specific question or its key terms. **Never load the whole journal into context** — pull only the relevant entry. (The journal is a write-mostly audit record, not a context input.)
- If a prior entry decided the same question and still applies, **follow it** and **cite it** as your `Justification` (e.g. "consistent with the 2026-05-22T13:40 decision"). A prior decision is itself a valid citation.
- If the prior decision no longer applies, treat your new call as a **revision** — supersede it (above).

## Examples

An `applied` decision:

```
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

A **revision** that supersedes it:

```
## 2026-05-22T16:05:00-07:00 — auth/02 — gate-resolution

**Question:** How long should password-reset tokens stay valid?
**Options considered:**
- 1 hour (prior choice)
- 30 minutes (tighter, after the security review)
**Chosen:** 30 minutes.
**Decided-by:** human
**Justification:** Security review asked to tighten the window; supersedes the earlier 1h call.
**Outcome:** applied
**Ref:** (pending)
**Supersedes:** 2026-05-22T13:40:00-07:00 — tightened after security review.
```

An `escalated` decision:

```
## 2026-05-22T14:10:00-07:00 — auth/03 — irreversible-action

**Question:** Drop the legacy `users.password_md5` column?
**Options considered:**
- Drop it now (irreversible, data loss)
- Keep it, ignore it in new code
- Migrate values, then drop in a later issue
**Chosen:** —
**Decided-by:** agent
**Justification:** No artifact authorizes destroying the column, and it's irreversible. Escalating rather than guessing.
**Outcome:** escalated
**Ref:** (pending)
```

See `scenarios.md` for the behavioral cases this skill must satisfy.
