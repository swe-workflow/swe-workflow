# log-decisions — scenarios

Spec-by-example for the `log-decisions` skill. Each scenario maps a situation to the expected result in `DECISIONS.md`. These are the acceptance checks: following the skill's instructions for the situation must produce the expected entry. (Instructions-only skill — "tests" are behavioral, verified by reading the produced artifact.)

Scope of this set: the entry **schema** and the **direct-append** write path. The decision bar + escalation (issue 02), dedup/supersede/precedent (03), and worktree staging→promotion (04) carry their own scenarios in those slices.

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
