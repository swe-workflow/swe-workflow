---
name: swe-workflow
description: Orchestrates the full five-stage flow from raw idea to shipped PR — grill-with-docs → to-prd → to-issues → triage → worktree+planning-with-files. Each stage answers one question (What do I want? / What does done look like? / What are the units of work? / What's actionable? / Build it). Use when the user has an idea but no spec yet, wants to plan a feature end-to-end, says "let's PRD this," asks "how do I start on this idea?", or grabs a ready-for-agent issue to implement.
---

# SWE Workflow

The idiomatic software-engineer workflow: clarify the idea → spec it → slice it → triage it → ship it. Five stages, each with a dedicated skill and a durable artifact that feeds the next.

## Plugin commands

This skill ships as the **`swe-workflow` plugin**. Its actionable entry points are slash commands:

| Command | Stage(s) | What it does |
|---|---|---|
| `/swe-workflow:ship <issue-slug>` | 5–7 | Plan → build → close out ONE issue (worktree, test-first build, PR, teardown). |
| `/swe-workflow:ship-all [scope]` | 5–7 ×N | Run `/ship` over a backlog in dependency order — AFK issues only, pausing at HITL. |
| `/swe-workflow:to-features` | 2 | Enumerate user-facing features into `FEATURES.md`. |
| `/swe-workflow:status` | — | Show planning status for the current issue. |

The spec-layer stages (1 `grill-with-docs`, 3 `to-prd`, 4 `to-issues`, plus the parallel `/triage`) and the execution engine (`planning-with-files`, `tdd`, `karpathy-guidelines`) are **external skills this plugin orchestrates** — install them separately (see the plugin README). The diagram below is the conceptual chain; `/swe-workflow:ship` automates stages 5–7 of it.

## The workflow

```
┌────────────────────── SPEC LAYER (mattpocock) ──────────────────────┐
│                                                                      │
│  0. How is this repo set up?                                         │
│     /setup-matt-pocock-skills ──► AGENTS.md, docs/agents/            │
│              (one-time: tracker, triage labels, doc layout —         │
│               wires this repo's conventions into the chain)          │
│                                                                      │
│  1. What do I want?                                                  │
│     /grill-with-docs ──► CONTEXT.md, ADRs                            │
│              (resolve domain language; capture decisions —           │
│               re-run until no questions remain or you abort)         │
│                                                                      │
│  2. What features does this break into?                              │
│     /to-features ──► FEATURES.md                                     │
│              (read CONTEXT.md + ADRs → user-facing features;         │
│               strike through, don't delete, on ship)                 │
│                                                                      │
│  3. What does done look like?                                        │
│     /to-prd ──► PRD (auto-labeled `ready-for-agent`)                 │
│              (Problem / Solution / User Stories /                    │
│               Implementation Decisions / Testing Decisions / Scope)  │
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
┌────────── EXECUTION LAYER (worktree + planning-with-files) ──────────┐
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

**Engineering-side, by design.** The mattpocock toolchain assumes features come from product thinking (user needs, business goals) that lives outside this skill ecosystem. Stage 2 (`/to-features`) is the deliberate seam: features get *enumerated* here (read from `CONTEXT.md` + ADRs), but *discovered* elsewhere — in user interviews, product strategy, sales conversations, whatever your team uses. This toolchain has no opinion on that.

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

## When is it done?

The mirror image of "Where to enter the chain" — four levels of "done", four signals:

| Level | Done when | Recorded in |
|-------|-----------|-------------|
| Phase | TDD cycle green + logged | `task_plan.md` checkbox ticked |
| Issue | All phases ticked, PR merged | tracker status (closed/merged) |
| Feature | All issues from its PRD merged | `FEATURES.md` strike-through w/ shipped refs |
| Project | (no native concept — judgment call) | — |

A feature's completion is mechanical: walk from the PRD to its child issues (via the parent reference `/to-issues` writes), confirm all closed, then strike through the `FEATURES.md` line:

```
- [x] ~~user-can-reset-password~~ — ~~A user can reset...~~ (shipped: #42, #43, #44)
```

Software projects rarely "complete" — features keep getting added. If you need a hard milestone, layer on your tracker's mechanism (`gh milestone`, Linear cycles, release tags) and define "project complete" as that milestone closing. See [REFERENCE.md](REFERENCE.md#completion-signals) for per-tracker completion queries.

## Stages 5-7: worktree + planning-with-files

The skill is **instructions-only** — there are no scripts. The agent performs each step manually, adapting to the team's issue tracker.

### Bootstrap

1. **Pick the tracker.** See [Tracker selection](REFERENCE.md#tracker-selection).
2. **Fetch the issue** per [`trackers/<name>.md`](trackers/) — extract title, body, labels, AGENT-BRIEF.
3. **Derive paths**:
   - slug = title → lowercase → non-alphanumerics replaced with `-` → truncate to 40 chars
   - branch = `issue-<id>-<slug>` (Linear's `TEAM-123` passes through literally)
   - worktree = `../<repo>-issue-<id>/`
4. **Create the worktree**: `git worktree add ../<repo>-issue-<id> -b issue-<id>-<slug>`
5. **`cd` in and seed** three planning files:

   | File | Contents |
   |---|---|
   | `task_plan.md` | Goal = title; Phases = AC checkboxes. **Structured fields only** (hook re-injection risk). |
   | `findings.md` | Raw issue body + AGENT-BRIEF pasted verbatim. Safe sink for external content. |
   | `progress.md` | Initial session log entry with bootstrap timestamp. |

6. **Invoke `/planning-with-files:plan`** (Stage 5) with the exact prompt the [`/swe-workflow:ship`](../../commands/ship.md) command uses (Stage 5, step 6) — kept there as the single source so it can't drift. That prompt has the planner **write `/tdd` and `andrej-karpathy-skills:karpathy-guidelines` as named rules into `task_plan.md`**, which carries the methodology into Stage 6: `plan-goal` re-reads the plan and applies the named skills instead of being re-told. The interview refines the AC seeds into real phases; `task_plan.md` is the **core artifact** Stage 6 reads, `findings.md` holds the raw issue body.
7. **Invoke `/planning-with-files:plan-goal`** to execute (Stage 6) — reads `task_plan.md`, drives each phase as a goal via Claude Code's goal command; outer loop runs phases; `/tdd` is the inner loop for code-producing phases. Since the Stage 5 prompt already named `/tdd` and `/andrej-karpathy-skills:karpathy-guidelines`, the plan calls for them — `plan-goal` carries them out: test-first, surgical changes, simplicity first, no speculative abstractions, surfaced assumptions.
8. **Close out** (Stage 7) — open the PR with the body drawn from `progress.md` highlights (the session log *is* the narrative; don't rewrite it). After it merges, [tear down](#teardown-after-pr-merges) the worktree and branch.

### Inner loop: `/tdd` for code-producing phases

`/planning-with-files:plan-goal` is the **outer loop** (phases, state, errors); `/tdd` is the **inner loop** (one failing test → one minimal fix). For each phase in `task_plan.md` that produces testable code:

```
Mark phase in_progress  →  /tdd (red → green → refactor)  →  log to progress.md  →  Mark phase complete
```

Not every phase needs `/tdd` — exploration, config tweaks, and infra changes skip it. See [REFERENCE.md](REFERENCE.md#inner-loop-tdd-within-each-code-producing-phase) for the full nuances (multiple cycles per phase, decision/error capture, when `/tdd`'s own planning step duplicates vs. complements the issue-level plan).

### Teardown (after PR merges)

From the **main checkout** (never inside the worktree): verify the worktree is clean, `git worktree remove`, then `git branch -d` only if it merged into the default branch. Exact commands: [REFERENCE.md](REFERENCE.md#teardown-after-pr-merge).

## Critical handoff rules

1. **PRD uses the glossary from stage 1.** If `to-prd` introduces terms that conflict with `CONTEXT.md`, loop back to `/grill-with-docs`.
2. **Issues are tracer bullets, not horizontal layers.** Each is a thin vertical slice (schema → API → UI → tests). "Backend issue" + "frontend issue" is a smell — re-slice.
3. **Only `ready-for-agent` issues enter execution.** `/to-issues` auto-applies the label on chain-created issues; `/triage` applies it to external issues (user reports, etc.). Either way, stage 5 reads from the label, not the source.
4. **One issue = one worktree = one `task_plan.md`.** Filesystem isolation for parallel AFK agents. No exceptions.
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
