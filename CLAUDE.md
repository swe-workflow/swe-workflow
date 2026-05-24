# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`swe-workflow` is a **Claude Code plugin distributed as pure Markdown** — there is **no source code, build, test suite, lint, package manifest, or CI** (confirmed: zero `.sh/.js/.ts/.py`, no `.github/`). The deliverable *is* the instruction text in the command and skill files. "Developing" here means editing Markdown so a future Claude instance follows the workflow correctly; "testing" means reading for cross-file consistency.

It packages the idiomatic SWE workflow — **idea → PRD → issues → ship** — as a chain of small skills connected by Markdown artifacts, deliberately *not* a framework.

## Repository layout (each dir maps to an architectural role)

- `.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json` — plugin and marketplace manifests. Both carry a `version` that **must stay in sync** (see Releasing).
- `commands/*.md` — the 5 slash commands, the actionable entry points: `setup`, `spec`, `ship`, `ship-all`, `status`. Each has YAML frontmatter (`description`, `allowed-tools`, sometimes `argument-hint`) and references bundled skills via `${CLAUDE_PLUGIN_ROOT}`.
- `skills/swe-workflow/` — the flagship skill: `SKILL.md` (the map + end-to-end conductor), `REFERENCE.md` (per-stage detail, read when something feels off), `trackers/` (tracker-adapter docs).
- `skills/to-features/`, `skills/log-decisions/` — the two other bundled skills.
- `.scratch/` — untracked dogfooding workspace; the repo runs its own workflow on itself (e.g. the `log-decisions` feature's PRD + issues live here).

## The workflow's architecture (read SKILL.md + REFERENCE.md for the whole picture)

Stages 0→7, split into two layers plus a parallel concern. Each command automates a contiguous range:

- **Stage 0 — `/setup`**: one-time per-repo bootstrap (install prereq skills, inject the decision-logging rule, set the `DECISIONS.md` privacy policy). Idempotent.
- **Stages 1–4 — `/spec`** (*spec layer*, interactive): grill domain → enumerate features → PRD → tracer-bullet issues. Leaves a `ready-for-agent` backlog.
- **Stages 5–7 — `/ship` (one issue) and `/ship-all` (the backlog, AFK)** (*execution layer*): worktree + planning-with-files → test-first build → PR + teardown.
- **Parallel — `/triage`**: a state machine over *external* issues; **not** in the critical path (chain-created issues are auto-labeled `ready-for-agent`).
- The **`swe-workflow` skill itself** is the conductor for the whole 0→7 chain when invoked without a specific stage.

This plugin **orchestrates external skills it does not bundle** — `planning-with-files`, `karpathy-guidelines`, `tdd`, and mattpocock's `setup-matt-pocock-skills` / `grill-with-docs` / `to-prd` / `to-issues` / `triage`. `/setup` auto-installs them. When editing, assume these are separate skills, not files in this repo.

Five cross-cutting invariants hold the design together — internalize before editing:

1. **Files are the interface.** Every handoff is a Markdown artifact: `CONTEXT.md`/ADRs → `FEATURES.md` → PRD → issues → `task_plan.md`/`findings.md`/`progress.md`. Nothing lives only in the agent's head.
2. **Tracker contract.** `skills/swe-workflow/trackers/README.md` is the *interface*; each `trackers/<name>.md` (github, gitlab, linear, multica, local-markdown) is a thin *adapter*. Supporting a new tracker = adding one adapter doc, nothing else changes.
3. **Security boundary.** `planning-with-files` re-injects `task_plan.md` into context on every tool call, making it a prompt-injection amplifier, so it gets only **structured fields the executor wrote**. Raw external content (issue bodies, fetched docs) goes to `findings.md`.
4. **Decision journal** (`log-decisions` skill). Bar-crossing build decisions append to per-worktree `DECISIONS.staged.md` (gitignored), promoted to repo-root `DECISIONS.md` at close-out. The decide/assume/escalate call is a 2×2 over *determinable × reversible*, with a catastrophic-action floor that always escalates.
5. **Ephemeral state by design.** Per-issue worktrees + planning files die at PR merge. Only the tracker and `CONTEXT.md`/ADRs survive across features.

## Design philosophy is a hard constraint on changes

The whole point is being a **chain of small, observable skills — not a framework** (SKILL.md "Design philosophy"). Three principles gate every edit: own the process (you decide what enters each stage's context), every artifact is observable (`cat`-able Markdown), ephemeral state is intentional. Two load-bearing commitments follow:

- **Instructions-only, no scripts.** Deterministic steps are documented as instructions the agent runs, never wrapped in scripts. This is *why* the repo has zero code — keep it that way.
- **Transparent Markdown all the way down.** Reject any addition — even one borrowed from a useful-looking framework — that reduces user control or observability.

## Editing conventions / gotchas

- **Single source of truth for the planner prompt.** The exact Stage-5 `/planning-with-files:plan` prompt lives **only** in `commands/ship.md` (Stage 5, step 6). `SKILL.md` and `REFERENCE.md` deliberately point at it rather than restate it, so it can't drift. Don't copy it elsewhere.
- **No `": "` (colon-space) inside a `description:` frontmatter value.** A mid-sentence colon-space breaks the strict-YAML frontmatter parser — a recurring defect here (multiple commits fixing it). Use an em-dash (`—`) or "such as" instead. Applies to every `commands/*.md` and `skills/*/SKILL.md`.
- **Don't double-track.** Each fact has one home (see the "Don't double-track" table in SKILL.md). The same discipline applies to the docs themselves: cross-link with relative links rather than restating content.

## Releasing

No automated pipeline. To cut a release:

1. Bump `version` in **both** `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` (keep them identical).
2. Commit with the established convention: `Release X.Y.Z: <one-line summary>` (see `git log`). Releases are not git-tagged.
