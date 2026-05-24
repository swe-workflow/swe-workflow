# Stages 5–7 ×N — Ship-all

A thin, unattended **loop over the ship stage** ([`ship.md`](ship.md)) — run it over every ready issue, one at a time. On Claude Code this runs as `/swe-workflow:ship-all`; on other agents, invoke the `swe-workflow` skill and ask to ship the backlog.

**Scope** comes from the user's request — optional. A feature directory (e.g. `.scratch/feature-x/` or `.scratch/feature-x/issues/`) limits the batch to that feature. If none is given, target **all `ready-for-agent` issues** for the active tracker.

The **ship** stage owns everything **per issue** — fetch, worktree, plan, test-first build, PR, teardown, its re-run/resume check, HITL pauses, and the prerequisites. **Don't restate any of that here.** Ship-all adds only the **batch wrapper**: which issues, in what order, one at a time, what to do when one parks or needs a human, and the closing summary.

## Procedure
1. **Enumerate** issues in scope via the active adapter's **list-ready** operation — the per-tracker query lives in [trackers/](../trackers/) (the [contract](../trackers/README.md#operations-every-adapter-implements)).
2. **Order by dependency**: ship prerequisites first; defer any issue whose `blocked-by` haven't merged yet, and note it.
3. **For each AFK issue, in order**: run the **ship** stage ([`ship.md`](ship.md)) on it to completion before starting the next — one worktree at a time. Ship's own re-run check resumes an interrupted issue and no-ops a finished one, so the loop is safe to restart. Two outcomes need batch handling:
   - **Escalation mid-build** (a `log-decisions` escalation — see [`ship.md`](ship.md), Stage 6; a reversible-but-unsettled call proceeds as a logged assumption and does *not* park): **park the issue** — leave its worktree + `DECISIONS.staged.md` intact, skip its dependents, and continue with the next independent issue. A park is not a halt.
   - **Build failure** (the 3-strike error protocol trips — something is broken): **halt the batch** and surface it.
4. **At a HITL issue** (`ready-for-human`, or an AGENT-BRIEF flagging HITL — see [`ship.md`](ship.md), HITL): **don't attempt it unattended** — handed to the ship stage it would only pause mid-build waiting for a human. Surface it (what shipped, what's blocked on a human, what remains) and resume on the user's go-ahead.
5. **Summarize**: issues shipped (PR refs), skipped/blocked (reasons), parked on an escalation (awaiting your decision), and any HITL issues awaiting the user.

**Isolation makes this safe** — one issue = one worktree = one set of planning files (a ship invariant), so parked and shipped issues never interfere.

**Resume** — re-running ship-all is idempotent: shipped issues close and drop out of step 1; parked or interrupted issues resume through ship's re-run check. To clear a park, resolve its escalation (appends a `Supersedes` entry per `log-decisions`), then re-run — ship-all picks it back up, or run **ship** on it directly. Review open escalations anytime with **status** ([`status.md`](status.md)) from the main checkout.

**Prerequisites**: same as the **ship** stage, which stops if any are missing.
