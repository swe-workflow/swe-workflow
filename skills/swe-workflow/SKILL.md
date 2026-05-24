---
name: swe-workflow
description: Orchestrates the full five-stage flow from raw idea to shipped PR — grill-with-docs → to-prd → to-issues → triage → worktree+planning-with-files. Each stage answers one question (What do I want? / What does done look like? / What are the units of work? / What's actionable? / Build it). Use when the user has an idea but no spec yet, wants to plan a feature end-to-end, says "let's PRD this," asks "how do I start on this idea?", or grabs a ready-for-agent issue to implement.
---

# SWE Workflow

The idiomatic software-engineer workflow: clarify the idea → spec it → slice it → triage it → ship it. Five stages, each with a dedicated skill and a durable artifact that feeds the next. Invoked end-to-end with no specific stage in mind, this skill is the **conductor** that drives the chain 0→7 — see [Driving the chain](#driving-the-chain-stages-07); invoked at a point, it's the map for that stage.

## How it's exposed

`swe-workflow` is **one [Agent Skill](https://agentskills.io)** — portable across Claude Code, Codex, Gemini CLI, Cursor, and other skills-compatible agents — that conducts the whole 0→7 chain. Each stage's procedure lives in a reference file it loads on demand:

| Stage(s) | Procedure | Claude command |
|---|---|---|
| 0 | [`references/setup.md`](references/setup.md) | `/swe-workflow:setup` |
| 1–4 | [`references/spec.md`](references/spec.md) | `/swe-workflow:spec` |
| 2 | [`references/to-features.md`](references/to-features.md) | `/swe-workflow:to-features` |
| 3 | [`references/grill-feature.md`](references/grill-feature.md) | `/swe-workflow:grill-feature` |
| 5–7 | [`references/ship.md`](references/ship.md) | `/swe-workflow:ship` |
| 5–7 ×N | [`references/ship-all.md`](references/ship-all.md) | `/swe-workflow:ship-all` |
| — | [`references/status.md`](references/status.md) | `/swe-workflow:status` |

- **Claude Code** — invoke the seven `/swe-workflow:*` commands (thin `commands/` shims that run the matching procedure), or invoke this `swe-workflow` skill to drive the whole chain.
- **Other agents** — invoke the `swe-workflow` skill and say what you want (*"ship issue 42"*); it routes to the right stage's reference file.

**`log-decisions`** ships in the suite as a companion skill (the decision journal). Stage 2 — **`to-features`**, which splits the project into coarse features in `FEATURES.md` — is a stage of *this* skill (its [`references/to-features.md`](references/to-features.md) procedure + the `/swe-workflow:to-features` command), not a separate skill. The spec-layer skills (`grill-with-docs`, `to-prd`, `to-issues`, and the parallel `triage`) and the execution engine (`planning-with-files`, `tdd`, `karpathy-guidelines`) are **external skills this suite orchestrates** — the [setup procedure](references/setup.md) auto-installs them (see the [README](../../README.md)). The diagram below is the conceptual chain; **spec** automates stages 1–4 of it and **ship** stages 5–7.

## The workflow

```
┌──────────────── BOOTSTRAP — /swe-workflow:setup (0) ─────────────────┐
│                                                                      │
│  0. How is this repo set up?                                         │
│     /setup-matt-pocock-skills ──► AGENTS.md, docs/agents/            │
│              (one-time: tracker, triage labels, doc layout —         │
│               wires this repo's conventions into the chain)          │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────── SPEC LAYER — /swe-workflow:spec (1-4) ───────────────┐
│                                                                      │
│  1. What do I want?                                                  │
│     /grill-with-docs ──► CONTEXT.md, ADRs                            │
│              (resolve domain language; capture decisions —           │
│               re-run until no questions remain or you abort)         │
│                                                                      │
│  2. What features does this break into?                              │
│     /to-features ──► FEATURES.md                                     │
│              (PM — high-level grill → coarse user-facing features;   │
│               strike through, don't delete, on ship)                 │
│                                                                      │
│  3. What does done look like?  (per feature)                         │
│     /grill-feature = grill-with-docs <feature> + /to-prd ──► one PRD │
│              (intensive feature grill, then synthesize —             │
│               Problem / Solution / Stories / Decisions / Scope)      │
│                                                                      │
│  4. What are the units of work?                                      │
│     /to-issues ──► N tracer-bullet issues                            │
│              (vertical slices, all auto-labeled `ready-for-agent`    │
│               — /triage NOT in the critical path)                    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘

                                  │
                  (Agent grabs ONE `ready-for-agent` issue)
                                  │
                                  ▼
┌───────────── EXECUTION LAYER — /swe-workflow:ship (5-7) ─────────────┐
│                                                                      │
│  5. How do I plan each issue?                                        │
│     Fetch issue (per tracker) ──► worktree + branch + seed files     │
│              (task_plan.md, findings.md, progress.md from AC)        │
│                                                                      │
│     /planning-with-files:plan ──► interview → make the plan          │
│              (prompt bakes in /karpathy-guidelines + /tdd —          │
│               shapes phases, key questions, decisions to make)       │
│                                                                      │
│                    step 5 writes ▼                                   │
│                        ┌────────────────────┐                        │
│                        │    task_plan.md    │                        │
│                        └────────────────────┘                        │
│                     step 6 reads ▼                                   │
│                                                                      │
│  6. How do I build each issue?                                       │
│     /planning-with-files:plan-goal ──► read task_plan.md,            │
│              work each sub-task in order → commit                    │
│              (sub-tasks already name /tdd + /karpathy-guidelines)    │
│                                                                      │
│  7. How do I close out each issue?                                   │
│     progress.md highlights ──► PR body / closing comment             │
│              (the session log IS the PR narrative — don't rewrite)   │
│                                                                      │
│     Teardown ──► git worktree remove + branch -d if merged           │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

*File-based end to end — each step hands the next a markdown artifact: `CONTEXT.md`/ADRs → `FEATURES.md` → PRD → issues → `task_plan.md` → `progress.md`. The files are the interface between steps; nothing lives only in the agent's head.*

## Parallel concern: `/triage`

`/triage` sits beside the chain, not inside it — a small state machine over the issue tracker (`needs-info` / `ready-for-agent` / `ready-for-human` / `wontfix`). Required for issues filed *outside* the chain (user bug reports, external contributions, ad-hoc feature requests); redundant for chain-created issues, since `/to-prd` and `/to-issues` auto-label `ready-for-agent` at creation.

See [REFERENCE.md](REFERENCE.md#parallel-concern-triage--whats-actionable-for-external-issues) for the full state machine and per-state outputs.

## Design philosophy

This is a **chain of small skills, not a framework.** Three principles guard against drifting into framework opacity:

1. **Own the process.** "Process" here means *deciding what goes into context at each stage*. Every skill in the chain is a markdown file you can read, edit, swap, or skip — there is no opaque orchestrator.
2. **Every artifact is observable.** PRDs, issues, AGENT-BRIEFs, `task_plan.md`, `findings.md`, `progress.md` — all human-readable markdown, all `cat`-able at any point.
3. **Ephemeral state is intentional.** Per-issue worktrees and planning files die when the PR ships. Deliberate defense against spec/plan drift accumulating into a "ball of mud" over time.

Operating maxim (Matt Pocock, after [surveying ~2000 AI coding course participants on framework dissatisfaction](https://x.com/mattpocockuk/status/2044029094942159126)): *"a good framework hands a lot of control over to the user and is easy to observe."* If a proposed addition reduces either, reject it — even if it's borrowed from a framework that looks useful.

**Concrete commitments** derived from these principles:

- **Instructions-only, no scripts.** Deterministic operations are documented as instructions the agent runs, not wrapped in scripts. Every script reintroduced would move the chain toward the opacity Matt's surveyed users rejected.
- **Transparent markdown all the way down.** Seven chain stages plus `/triage` as a parallel concern — every link is a markdown skill or documented procedure you can read, edit, or replace without touching code. None of them opaque. The direct test of the operating maxim above.

**Engineering-side, by design.** The mattpocock toolchain assumes features come from product thinking (user needs, business goals) that lives outside this skill ecosystem. Stage 2 (`/to-features`) is the deliberate seam: features get *enumerated* here — an interview run in the grill's conversation that turns domain understanding into a backlog — but *discovered* elsewhere, in user research, product strategy, sales conversations, whatever your team uses. This toolchain has no opinion on that.

See [REFERENCE.md](REFERENCE.md#how-this-differs-from-spec-kit-class-frameworks) for the comparison with spec-kit / BMAD / GSD.

## Where to enter the chain

Don't always start at stage 1 — jump to where the chain actually breaks.

| Entry signal | Start at |
|--------------|----------|
| Fresh repo, no `## Agent skills` block or `docs/agents/` yet | 0 |
| Vocabulary fights, fuzzy terms, no glossary yet | 1 |
| Domain understood, features not yet enumerated | 2 |
| Feature picked, no PRD yet for this one | 3 |
| PRD exists but is one mega-issue | 4 |
| Picked a `ready-for-agent` issue, ready to plan | 5 |
| `task_plan.md` refined, ready to implement | 6 |
| Implementation committed, ready to open the PR + tear down | 7 |
| External issue filed by a user, needs classification | (parallel: `/triage`) |

## Driving the chain (stages 0→7)

Invoked without a specific stage — *"plan this feature end-to-end," "how do I start on this idea?"* — act as the **conductor**. The chain packs into three **idempotent** building blocks; run them in order and let each skip whatever's already done:

1. **Setup** (stage 0) — only if the repo isn't bootstrapped; it skips itself otherwise.
2. **Spec** (stages 1–4, *AFK-friendly*) — grill → features → PRD → issues, leaving a `ready-for-agent` backlog. Resumes from whatever specs already exist.
3. **Ship-all** (stages 5–7, *AFK*) — build and ship the backlog.

**Spec is pausable; execution stays automatic via post-batch review.** Both blocks are **AFK-friendly** and record decisions via `log-decisions`; the **key difference** is what happens on an **unsure HITL call** (one only you can make). **Spec (1–4) pauses** to ask — its interviews proceed on recommended answers when you're away, stopping only to escalate a call that genuinely needs you. **Ship-all persists instead** — it journals and **parks** the escalation for your **batch review** (via [`/status`](references/status.md)) and keeps shipping independent issues, so the AFK batch never blocks (a pre-declared HITL issue is the exception — `ship-all` won't attempt it unattended and waits). **Re-invoking is safe** — each command self-detects state, so a re-run continues where the chain left off. To jump straight to a single stage instead of driving the whole thing, use [Where to enter the chain](#where-to-enter-the-chain). `/triage` stays a parallel concern — pull external issues into the backlog as needed.

## When is it done?

The mirror image of "Where to enter the chain" — four levels of "done", four signals:

| Level | Done when | Recorded in |
|-------|-----------|-------------|
| Phase | TDD cycle green + logged | `task_plan.md` checkbox ticked |
| Issue | All phases ticked, PR merged | tracker status (closed/merged) |
| Feature | All issues from its PRD merged | `FEATURES.md` strike-through w/ shipped refs |
| Project | (no native concept — judgment call) | — |

Phase/issue/feature completion is a **fact** the toolchain verifies; project completion is a **judgment call** it stays out of — layer on `gh milestone` / Linear cycles / release tags if you need a hard milestone. The mechanical walk (PRD → child issues → confirm closed → strike the `FEATURES.md` line) and per-tracker completion queries are in [REFERENCE.md](REFERENCE.md#completion-signals).

## Stages 5-7: worktree + planning-with-files

The diagram above is the map. The procedure is single-sourced in [`references/ship.md`](references/ship.md) (one issue) and [`references/ship-all.md`](references/ship-all.md) (the backlog) — **instructions-only, no scripts**, **idempotent** (a re-run resumes or no-ops via ship.md's re-run check), carrying the single-source planner prompt (ship.md Stage 5, step 6) that bakes `/tdd` + `karpathy-guidelines` into `task_plan.md`. Per-stage detail — the `/tdd` inner loop, discovered scope, HITL, teardown commands, the decision journal — is in [REFERENCE.md](REFERENCE.md#stage-5-worktree--planning-with-files--build-it). The bootstrap enforces the [security boundary](#security-boundary) below: structured fields only in `task_plan.md`, raw external text in `findings.md`.

## Critical handoff rules

1. **PRD uses the glossary from stage 1.** If `to-prd` introduces terms that conflict with `CONTEXT.md`, loop back to `/grill-with-docs`.
2. **Issues are tracer bullets, not horizontal layers.** Each is a thin vertical slice (schema → API → UI → tests). "Backend issue" + "frontend issue" is a smell — re-slice.
3. **Only `ready-for-agent` issues enter execution.** `/to-issues` auto-applies the label on chain-created issues; `/triage` applies it to external issues (user reports, etc.). Either way, stage 5 reads from the label, not the source.
4. **One issue = one worktree = one `task_plan.md`.** Filesystem isolation for parallel AFK agents — no exceptions. (`/ship-all` itself is sequential; the isolation enables *optional, human-driven* parallelism — see [Parallel execution](REFERENCE.md#parallel-execution).)
5. **Strike through, don't delete.** When a feature ships, strike it through in `FEATURES.md` with a shipped reference — never delete. Preserves institutional memory; prevents quiet scope drift.

## Don't double-track

| Lives in… | Don't also put in… |
|-----------|--------------------|
| PRD (immutable arch decisions) | `task_plan.md` (would rot; the spec is authoritative) |
| AGENT-BRIEF (durable contract) | `task_plan.md` (copy only AC + key interfaces; raw brief goes in `findings.md`) |
| `task_plan.md` (execution-time decisions, errors hit) | The issue (don't litter the spec with build noise) |
| `progress.md` (session log) | A hand-written PR summary (the log IS the summary) |

## Security boundary

`planning-with-files` re-injects `task_plan.md` into context on every tool call. Any text in `task_plan.md` is an amplified prompt-injection target.

- Raw issue bodies, fetched docs, web content → `findings.md` only.
- `task_plan.md` gets only **structured fields** the executor wrote (Goal, Phases from AC, Decisions, Errors).

The bootstrap procedure ([Stages 5-7](#stages-5-7-worktree--planning-with-files)) enforces this split.

## When to skip this skill

- Single-file edits (no spec, no plan needed)
- Bug fixes where the AGENT-BRIEF is one paragraph — just do it, skip stage 5 bootstrap
- Exploration / prototypes — use the `prototype` skill instead

## Further reading

- [REFERENCE.md](REFERENCE.md) — per-stage detail, HITL vs AFK execution, gotchas
- Source skills: `grill-with-docs`, `to-prd`, `to-issues`, `triage` (mattpocock/skills), `planning-with-files` (OthmanAdi/planning-with-files)
