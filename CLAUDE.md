# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`swe-workflow` is a **suite of [Agent Skills](https://agentskills.io) distributed as pure Markdown** — portable across Claude Code, Codex, Gemini CLI, Cursor, and other skills-compatible agents, and *also* packaged as a Claude Code plugin. There is **no source code, build, test suite, lint, or CI, and no scripts** (confirmed: zero `.sh/.js/.ts/.py`, no `.github/`) — staying scriptless is a hard constraint (see Design philosophy) and is *how* the suite stays cross-agent. The deliverable *is* the instruction text in the **skill** files; the `commands/` are thin Claude-only shims over them. "Developing" here means editing Markdown so a future agent follows the workflow correctly; "testing" means reading for cross-file consistency.

It packages the idiomatic SWE workflow — **idea → PRD → issues → ship** — as a chain of small skills connected by Markdown artifacts, deliberately *not* a framework.

## Repository layout (each dir maps to an architectural role)

- `.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json` — plugin and marketplace manifests. Both carry a `version` that **must stay in sync** (see Releasing).
- `commands/*.md` — **thin Claude-Code shims** (`setup`, `spec`, `to-features`, `grill-feature`, `ship`, `ship-all`, `status`) that keep the `/swe-workflow:*` slash-command UX; each just delegates to the `swe-workflow` skill's matching procedure (`Invoke the swe-workflow skill and run its ship procedure…`). Behavior lives in the skill, not here — so other agents get the same workflow without the commands.
- `skills/swe-workflow/` — the one **workflow skill** that conducts the whole 0→7 chain: `SKILL.md` (the map + conductor), `REFERENCE.md` (per-stage detail + the always-on rules registry, read when something feels off), `references/{setup,spec,to-features,grill-feature,ship,ship-all,status}.md` (the seven **stage procedures**; `ship.md` holds the single-source planner prompt, while `to-features.md` and `grill-feature.md` are the single-source stage-2 / stage-3 procedures that `/spec` runs inline and the standalone `/swe-workflow:to-features` / `/grill-feature` commands also run), `trackers/` (tracker-adapter docs).
- `skills/log-decisions/` — the one companion skill (the append-only decision journal). Stage-2 feature enumeration is **not** a separate skill — it's the `swe-workflow` skill's `references/to-features.md` procedure + the `/swe-workflow:to-features` command shim (a stage, per the no-split-stages rule below).
- `README.md` — install (universal `npx skills add` + Claude plugin) and per-agent invocation.
- `.scratch/` — untracked dogfooding workspace; the repo runs its own workflow on itself (e.g. the `log-decisions` feature's PRD + issues live here).

## The workflow's architecture (read SKILL.md + REFERENCE.md for the whole picture)

Stages 0→7, split into two layers plus a parallel concern. Each **stage procedure** automates a contiguous range (on Claude Code, run via its `/swe-workflow:*` command):

- **Stage 0 — `/setup`**: one-time per-repo bootstrap (install prereq skills, inject the always-on rules, set the `DECISIONS.md` privacy policy). Idempotent.
- **Stages 1–4 — `/spec`** (*spec layer*, AFK-friendly): grill domain → split into coarse features → grill + PRD each feature (`/grill-feature`) → tracer-bullet issues. Leaves a `ready-for-agent` backlog.
- **Stages 5–7 — `/ship` (one issue) and `/ship-all` (the backlog, AFK)** (*execution layer*): worktree + planning-with-files → test-first build → PR + teardown.
- **Parallel — `/triage`**: a state machine over *external* issues; **not** in the critical path (chain-created issues are auto-labeled `ready-for-agent`).
- The **`swe-workflow` skill itself** is the conductor for the whole 0→7 chain when invoked without a specific stage.

This suite **orchestrates external skills it does not bundle** — `planning-with-files`, `karpathy-guidelines`, `tdd`, and mattpocock's `setup-matt-pocock-skills` / `grill-with-docs` / `to-prd` / `to-issues` / `triage` — all themselves cross-agent Agent Skills. `swe-setup` auto-installs them. When editing, assume these are separate skills, not files in this repo.

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

- **One skill + many commands (the planning-with-files pattern).** Behavior lives in the single `swe-workflow` skill — a workflow stage is a `skills/swe-workflow/references/<stage>.md` procedure the conductor runs (keep `SKILL.md` ≤500 lines; push detail to `references/` and `REFERENCE.md`). A `commands/<name>.md` is at most a thin Claude-only shim that delegates (`Invoke the swe-workflow skill and run its <stage> procedure…`) — **never put behavior there**, or non-Claude agents miss it. **Don't split stages into separate skills**: Claude always scans `skills/` with no allowlist, so extra skills can't be hidden there — Claude shows the one `swe-workflow` skill + the `/swe-workflow:*` commands, and other agents invoke the skill (it routes to the stage). `references/*.md` link to the skill's own docs as `../REFERENCE.md`.
- **Single source of truth for the planner prompt.** The exact Stage-5 `/planning-with-files:plan` prompt lives **only** in the ship procedure (`skills/swe-workflow/references/ship.md`, Stage 5 step 6). `SKILL.md`, `REFERENCE.md`, and the `commands/ship.md` shim deliberately point at it rather than restate it, so it can't drift. Don't copy it elsewhere.
- **No `": "` (colon-space) inside a `description:` frontmatter value.** A mid-sentence colon-space breaks the strict-YAML frontmatter parser — a recurring defect here (multiple commits fixing it). Use an em-dash (`—`) or "such as" instead. Applies to every `commands/*.md` and `skills/*/SKILL.md`.
- **Don't double-track.** Each fact has one home (see the "Don't double-track" table in SKILL.md). The same discipline applies to the docs themselves: cross-link with relative links rather than restating content.
- **Stage vs. step.** A *stage* is a numbered workflow phase (0→7); a *step* is a numbered action within a stage's procedure (e.g. `ship.md` "Stage 5, step 6") or plain prose. **Never write "step N" for a phase** — the spec-layer phases are stages 2/3/4, not steps. Recurring drift; sibling command files have disagreed.

## Releasing

No automated pipeline. To cut a release:

1. Bump `version` in **both** `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` (keep them identical).
2. Commit with the established convention: `Release X.Y.Z: <one-line summary>` (see `git log`). Releases are not git-tagged.

There's no separate Agent-Skills publish step — `npx skills add swe-workflow/swe-workflow` reads `skills/` straight from the GitHub repo, so a merged change is live for every agent immediately (the `.claude-plugin/` version only governs the Claude plugin install).
