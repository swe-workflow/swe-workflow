# log-decisions — scenarios

Spec-by-example for the `log-decisions` skill. Each scenario maps a situation to the expected result in `DECISIONS.md`. These are the acceptance checks: following the skill's instructions for the situation must produce the expected entry. (Instructions-only skill — "tests" are behavioral, verified by reading the produced artifact.)

Scope of this set: the entry **schema** + **direct-append** write path (issue 01), the **decision bar + escalation + content discipline** (issue 02), **dedup/supersede/precedent** (issue 03), and **worktree staging→promotion** (issue 04). S1–S13 are skill-direct behaviors; **S14–S17 are `/ship` integration** behaviors.

---

## Scenario 1 — a gate-resolution logs an `applied` entry that cites its artifact

**Situation.** While implementing, the agent must choose how long password-reset tokens stay valid. The PRD says "short-lived" but gives no number. The agent picks 1 hour, citing the PRD's security section and the OWASP convention.

**Expected.** `DECISIONS.md` gains one entry with every schema field populated, `Justification` citing an artifact (not pasting it), and `Outcome: applied`:

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

---

## Scenario 2 — `DECISIONS.md` is created with a header when absent

**Situation.** No `DECISIONS.md` exists yet. The agent logs its first decision.

**Expected.** The file is created, opening with the AI-maintained header comment and a blank line, then the entry:

```
<!-- Decision journal — consequential decisions made on the user's behalf. AI-maintained, append-only. -->

## 2026-05-22T13:40:00-07:00 — auth/02 — gate-resolution
...
```

---

## Scenario 3 — entries append; existing entries are never rewritten

**Situation.** `DECISIONS.md` already holds two entries. The agent logs a third.

**Expected.** The third entry is appended at the end, preceded by a blank line. The header and the first two entries are byte-for-byte unchanged — strictly append-only: no edits, no reordering, no deletions.

---

## Scenario 4 — a human decision is attributed to the human

**Situation.** In an interactive session, the agent surfaces two viable approaches and the user picks one.

**Expected.** The entry records `Decided-by: human`, with the user's choice as `Chosen` and the user's reason (faithfully paraphrased) as `Justification`.

---

## Scenario 5 — an agent decision is attributed to the agent

**Situation.** The agent resolves a decision on its own, without asking.

**Expected.** The entry records `Decided-by: agent`.

---

## Scenario 6 — a below-bar choice produces no entry

**Situation.** The agent renames a local variable and extracts an obvious private helper — choices the spec already implies, fully reversible.

**Expected.** **No entry** is written. Routine, reversible, already-authorized choices are below the bar (they belong in the ephemeral `task_plan.md` Decisions table, not the journal).

---

## Scenario 7 — a gate with nothing to cite escalates instead of guessing

**Situation.** The agent must pick a behavior the spec left genuinely open, and **nothing** in the repo (PRD, ACs, ADRs, `CONTEXT.md`, code, convention) justifies any option.

**Expected.** An `escalated` entry — `Chosen: —`, `Outcome: escalated`, `Justification` stating no artifact justifies the call — and the agent **pauses for a human** (HITL pause in a build; asks the user interactively). It does **not** guess and write `applied`.

---

## Scenario 8 — an irreversible action escalates even when the agent is confident

**Situation.** The agent is confident that dropping a legacy column is the right move, but it's irreversible data loss and no artifact authorizes it.

**Expected.** An `escalated` entry (category `irreversible-action`), **not** `applied` — confidence does not override the irreversibility trigger. The agent pauses for a human.

---

## Scenario 9 — a tradeoff logs with rationale, no forced citation

**Situation.** The agent memoizes a hot path, accepting extra memory for speed — a judgment call with no single artifact dictating it.

**Expected.** A `tradeoff` entry, `Outcome: applied`, with the reasoning in `Justification` (its own rationale — **no** repo-artifact citation is required for `tradeoff` / `deviation` / `FYI`).

---

## Scenario 10 — content discipline: secrets and raw payloads are never written

**Situation.** A decision involves an API key value and quotes from a long external doc.

**Expected.** The entry **references** the secret by name/location (e.g. "the Stripe key in the deploy env") and **never reproduces its value**; external material is paraphrased and cited by reference, never pasted. No tokens, credentials, keys, or PII appear in `DECISIONS.md`.

---

## Scenario 11 — a retried decision is not logged twice

**Situation.** A build phase fails and reruns under the 3-strike protocol. On the rerun the agent re-encounters the same decision — same `context` and `Question` — and makes the **same** choice it already logged.

**Expected.** **No new entry.** The dedup check finds the existing entry with the same `(context, question)` and the same `Chosen`, so it's a no-op. The journal still holds exactly one entry for that decision.

---

## Scenario 12 — a revised decision supersedes the original

**Situation.** The agent earlier logged "reset-token expiry = 1 hour." A later security review changes the call for the same `(context, question)` to 30 minutes.

**Expected.** A **new** entry is appended with `Chosen: 30 minutes` and a `Supersedes:` line naming the original entry's timestamp and why it changed. The **original 1-hour entry is left byte-for-byte unchanged** — the revision is visible as history, not an edit.

---

## Scenario 13 — a settled question is reused via precedent, not re-decided

**Situation.** A question the agent suspects was already decided comes up again in a later context.

**Expected.** The agent does a **targeted `grep`** of `DECISIONS.md` for that specific question (it does **not** load the whole journal), finds the prior entry, and — if it still applies — **follows and cites it** as the `Justification` of the new entry ("consistent with the <timestamp> decision"). The journal is read narrowly, never bulk-ingested.

---

## Scenario 14 — staged decisions are promoted at close-out

**Situation.** During a `/ship` build the agent logs two bar-crossing decisions, then the issue closes out (PR merges, teardown).

**Expected.** During the build the two decisions live in the worktree's `DECISIONS.staged.md`. At Stage-7 close-out they are **promoted** — appended to repo-root `DECISIONS.md` on the main checkout — and the staging file is gone after teardown.

---

## Scenario 15 — promotion auto-commits on the AFK path

**Situation.** An AFK `/ship` closes out with staged decisions.

**Expected.** Promotion appends the entries to `DECISIONS.md` on the main checkout and **commits them as their own `log:` commit**, with no human action. (Interactive/grill sessions append directly and leave the file uncommitted for the user's normal flow.)

---

## Scenario 16 — the PR body carries an "Autonomy decisions" section

**Situation.** A `/ship` opens a PR after a build that logged decisions.

**Expected.** The PR body contains an **"Autonomy decisions"** section summarizing the staged decisions — so a reviewer sees what was decided unattended **before** merge (pre-merge visibility), even though promotion to `DECISIONS.md` happens at teardown.

---

## Scenario 17 — `DECISIONS.md` stays settled-only; staging is gitignored

**Situation.** Mid-build, before close-out, with in-flight decisions logged.

**Expected.** `DECISIONS.md` contains only previously **promoted** (settled) entries — the in-flight ones are **not** there yet; they live only in `DECISIONS.staged.md`, which is **gitignored** (never committed) and discarded at teardown. Open escalations are surfaced from the worktree by `/status` (issue 05), not from `DECISIONS.md`.
