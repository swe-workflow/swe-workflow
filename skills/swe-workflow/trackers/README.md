# Tracker contract

A **tracker adapter** is one `trackers/<name>.md` doc that teaches the swe-workflow execution layer (`/ship`, `/ship-all`) how to read one issue tracker. The contract below is the **interface**; each adapter is a thin **implementation** that documents only what's specific to its tracker. To support a new tracker, add one doc that satisfies this contract — nothing else changes.

## Normalized issue

Every adapter's **fetch** returns an issue normalized to these fields, so the bootstrap stays tracker-agnostic:

| Field | Meaning |
|---|---|
| `title` | one-line issue title |
| `body` | the issue description / what-to-build |
| `labels` | flat list of label strings (the triage role lives here) |
| `agent-brief` | the durable execution contract — see resolution below |

Read off the `body`: acceptance criteria (`- [ ]` checkbox lines), the `blocked-by` chain, and whether the issue is HITL or AFK.

An adapter documents only its **deltas** from this default mapping (e.g. a field that's named differently in the tracker's JSON).

## AGENT-BRIEF resolution

The AGENT-BRIEF is the durable contract `/triage` writes when an issue becomes `ready-for-agent`:

1. **A `## Agent Brief` comment** (case-insensitive heading). If several exist, **last match wins** (`/triage` may re-post on long-running issues).
2. **Fallback — the body IS the brief**: when there's no such comment, treat the whole `body` as the brief. (Always so for local-markdown; common for Multica's structured bodies.)

An adapter only restates this if its location deviates (e.g. GitLab's "notes").

## Branch & worktree naming

Derived once, the same way for every tracker:

- `slug` = title → lowercase → non-alphanumerics to `-` → truncate to 40 chars
- `branch` = `issue-<id>-<slug>`
- `worktree` = `../<repo>-issue-<id>/`

`<id>` is the adapter's **native ID form** (GitHub integer, Linear `ENG-42`, Multica `ARC-47`, local `<feature>/<NN>`). Each adapter states its `<id>`; nothing else about naming is tracker-specific.

## Operations every adapter implements

- **Fetch one** — given an issue ID → a normalized issue. Used by `/ship`.
- **List ready** — enumerate the tracker's `ready-for-agent` issues (scoped to a feature if one is given). Used by `/ship-all`. `ready-for-agent` is the canonical *role*; the **actual label string** for this repo comes from mattpocock's `docs/agents/triage-labels.md` (written here as `<ready-for-agent>`).

## Selection — which adapter

First match wins:

1. **Explicit arg form** — from `/ship`'s `$ARGUMENTS`: a `.md` path → `local-markdown`; a `TEAM-NN` key → `linear`; a bare integer → `github`/`gitlab` (per remote host).
2. **`$SWE_WORKFLOW_TRACKER`** env var — per-invocation override.
3. **Repo record** — mattpocock's `docs/agents/issue-tracker.md` (written by stage-0) is the canonical per-repo tracker. If stage-0 was skipped, fall back to a `tracker=<name>` line in `.swe-workflow.conf`.
4. **Auto-detect** — each adapter's signal (below).
5. **Ask** the user.

The tracker name is *not* double-recorded: `.swe-workflow.conf` carries only swe-workflow-specific config (e.g. `decisions=`).

## Adding a tracker

Write `trackers/<name>.md` satisfying this contract — that's the whole job:

- **CLI / auth** and **ID format**
- **Fetch one** command + any field-mapping *deltas* from the normalized issue
- **List ready** command
- **AGENT-BRIEF location** — only if it deviates from the standard resolution
- **Auto-detect signal**

No other file changes. The five existing adapters are worked examples.
