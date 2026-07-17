# Design

Why swe-workflow is shaped the way it is. **Human-facing** — nothing here is loaded by an agent at runtime; the skill files under [`skills/swe-workflow/`](../skills/swe-workflow/) are the product, this is the rationale behind them.

## Philosophy

This is a **chain of small skills, not a framework.** Four principles:

1. **Own the process.** "Process" means *deciding what goes into context at each stage*. Every link in the chain is a markdown file you can read, edit, swap, or skip — no opaque orchestrator.
2. **Every artifact is observable.** PRDs, issues, AGENT-BRIEFs, `task_plan.md`, `findings.md`, `progress.md` — all human-readable markdown, all `cat`-able at any point.
3. **Ephemeral state is intentional — files *and* context.** Per-issue worktrees and planning files die when the PR ships, and each issue ideally builds in a fresh context. Two kinds of state, one defense: neither stale artifacts nor stale context can pile into a "ball of mud."
4. **Idempotent by design.** State lives in observable artifacts, not the agent's head, so any stage can re-read where things stand and pick up. Every command is safe to re-run: it no-ops finished steps, resumes interrupted ones, never duplicates or clobbers.

Operating maxim (Matt Pocock, after [surveying ~2000 AI coding course participants on framework dissatisfaction](https://x.com/mattpocockuk/status/2044029094942159126)): *"a good framework hands a lot of control over to the user and is easy to observe."* If a proposed addition reduces either, reject it — even if it's borrowed from a framework that looks useful.

Concrete commitments that follow:

- **Instructions-only, no scripts.** Deterministic operations are documented as instructions the agent runs, never wrapped in scripts. This is *how* the suite stays cross-agent.
- **Transparent markdown all the way down.** Every link is a markdown skill or documented procedure you can read, edit, or replace without touching code.
- **Engineering-side, by design.** Features get *enumerated* here (stage 2's product-level grill) but *discovered* elsewhere — user research, product strategy, sales. The toolchain has no opinion on that.

## Documentation architecture (the 2.0 restructure)

Two layers, one home per fact:

- **`SKILL.md`** is a pure router — the stage table, the conductor procedure, the cross-stage invariants as one-liners. Under 100 lines, since it loads on every invocation.
- **`references/<stage>.md`** is self-contained — everything an agent needs while running that stage, loaded only then. No third "detail" layer; a fact lives in the stage file where it bites.

**The seam rule.** The suite orchestrates external skills it doesn't bundle (`grill-with-docs`, `to-spec`, `to-tickets`, `triage`, `planning-with-files`, `tdd`). Each stage file documents **only the seam**: what the suite feeds the external skill, the invocation, the artifact that must exist afterward, and any constraint the suite *adds* (e.g. "no file paths in the PRD"). Never restate the external's own interview flow, states, or output format — it documents itself, and the agent reads it at invocation time. An external skill's name appears **only in the stage file that invokes it** (plus the README's prerequisites list), so an upstream rename is a one-file edit. (Upstream renaming `/to-prd` → `/to-spec` once rippled through eight docs; that's the disease this rule cures.)

## How this differs from spec-kit-class frameworks

swe-workflow shares the "idea → PRD → issues → implement" arc with github/spec-kit, BMAD-METHOD, and GSD. The difference is structural:

| | swe-workflow | spec-kit-class frameworks |
|---|---|---|
| Composition | Chain of small skills (each = one markdown file) | Monolithic framework |
| Context control | You decide what enters each stage's context | Framework manages context flow |
| Modifiability | Edit any skill's markdown to change behavior | Configure within the framework's surface |
| Debuggability | Every input/output is a readable artifact | Internal state often opaque |
| Time decay | Per-issue planning files die at PR merge | Spec/plan files accumulate over time |
| LLM stance | Probabilistic reasoning partner | Tries to coerce determinism via guardrails |

The [Pocock survey](https://x.com/mattpocockuk/status/2044029094942159126) flagged **framework opacity** as the dominant failure mode: *"Most of these frameworks optimize for demos, not debugging. The moment context goes wrong, everything falls apart."* This rules out patterns even when they look useful:

- **A "constitution" stage** — rejected. Architectural invariants already live in `CONTEXT.md` and `docs/adr/` (stage 1's outputs); a separate constitution stage imports the opacity this chain fights. Stage 0 is different — pure *tooling* configuration (tracker, labels, doc layout), not architectural rules.
- **Framework-driven implementation** (`/speckit.implement`-style) — replaced by the layered worktree + `planning-with-files` + `tdd` execution stage. More moving pieces, each observable.
- **Long-lived spec artifacts in the repo** — avoided. Planning files live in worktrees and die at PR merge; only the tracker and `CONTEXT.md`/ADRs/`FEATURES.md` survive across features.

## Session topology

How to map the 0→7 chain onto agent CLI sessions — operator guidance, not mechanism. Files are the interface, so a session can end wherever a durable artifact exists and the next resumes from it. But artifacts are a *lossy* compression of the grilling conversation, so every cut trades **context warmth** (fidelity to intent) against **isolation** (clean context, observability, parallelism, a fresh reviewer). Three points on the spectrum:

1. **One long session for the whole project.** Maximum warmth, zero setup — and the "ball of mud" the suite exists to avoid: by feature 3 the context is clogged with feature 1's grill, and the author reviews its own work. Toy scope only.
2. **One session per feature.** Stages 0–2 once, then a session per dependency-free feature (launched as its `Depends on:` features finish), running grill-feature → to-tickets → a feature-scoped ship-all inline. Lowest information loss across the spec→execution seam; costs shared context across the feature's issues and no per-issue parallelism.
3. **One session per issue.** As option 2 through stage 4, but the feature session ends at the backlog; each unblocked issue gets its own session and `/ship`. Per-issue isolation, cheap fresh reviewers, durable-and-observable state; the handoff leans entirely on the artifact, so a thin AGENT-BRIEF surfaces as a build gap. The remedy is a **richer artifact, not a longer session** — push the tacit into the brief, and mark genuinely context-hungry issues HITL.

**Default to option 3** for AFK / parallel-ready work: it honors every execution-layer invariant and cuts only at the *low-loss* boundary (issues → build), keeping the *high-loss* one (grill → PRD → issues) warm in one spec session. Option 2 when intent-fidelity outweighs isolation and features are small and interactive. The throughline: *the cure for a lossy artifact is a better artifact, not a longer session.*

**Effort per session type (Claude Code).** Spec is the highest-leverage reasoning in the chain — run spec sessions at max effort (`claude --effort max`, `/effort max`, or `CLAUDE_CODE_EFFORT_LEVEL=max`). The *session* level is the lever that reaches the external stages (they inherit it; the suite can't set their frontmatter). Keep execution sessions at the default — `max` there mostly overthinks a plan that already exists.

**Model tier per session type.** Same shape, one level up: run **spec sessions on a frontier-tier model** (the strongest the host supports); **execution sessions can drop to a workhorse model** — the plan already exists, so the run wants competent code-following, not novel reasoning. Invest where the reasoning is irreversible; economize where you're executing.

## Parallel execution

`/ship-all` runs the backlog **sequentially** and never auto-fans-out; concurrency is always optional and human-driven, made safe by worktree-per-issue isolation. The safety rule:

| Across… | Verdict | Why |
|---|---|---|
| Independent features (different PRDs, no `Depends on:`, disjoint code) | Safe — the sweet spot | Small disjoint diffs, no ordering, low conflict surface |
| Tracer-bullet slices of the *same* feature | Usually unsafe | Slices are deliberately vertical, so they touch the same schema/types/routing; `blocked-by` often serializes them anyway |

This doesn't contradict Anthropic's "one feature at a time" rule for long-running agents — that bounds *what one agent-iteration takes on*, not *how many agents run*. Each worktree here is bound to one issue, so the bound holds. Under parallelism the failure modes mutate, not vanish: *leave a clean state* becomes *leave a clean merge* (concurrent agents plan against `main`-as-it-was, not each other's work), and the baseline moves as peers merge. The toolchain keeps its own state parallel-safe — `DECISIONS.staged.md` is per-worktree, promotion is serialized by teardown, `/status` aggregates escalations across worktrees — leaving concurrent edits to shared *source* as the only residual risk, which is git's to resolve. **Guardrail:** parallelize only units with no unresolved dependency and minimal file overlap.

## Always-on rules registry

A few skills apply to *every* run rather than one stage. There's no framework toggle — each is a **named-skill-activation rule** written into a file already in context where it bites:

| Skill | Scope | Injection site |
|---|---|---|
| `log-decisions` | All stages (spec *and* build) | [setup procedure](../skills/swe-workflow/references/setup.md) §3 → `AGENTS.md`/`CLAUDE.md` sentinel block |
| `tdd` | ship + ship-all builds only | [ship procedure](../skills/swe-workflow/references/ship.md) Stage 5 planner prompt → `task_plan.md` |

The site follows the scope: an all-stages rule goes in the agent-instructions file (agent-agnostic, persistent, reloaded into every fresh context; sentinel-wrapped so re-runs no-op); an execution-only rule goes in `task_plan.md` (re-injected every tool call, dies with the worktree — the right lifetime for a rule that means nothing until code is written). `tdd` deliberately sits only in the planner prompt so opening the repo for a quick edit doesn't force red → green → refactor; widen it by adding it to setup's engineering-discipline block. **Adding a rule** = one row here + one injection site there. Those two sites are the only writers.

## Source skills & further reading

- `grill-with-docs`, `to-spec`, `to-tickets`, `triage`, `tdd`, `setup-matt-pocock-skills` — https://github.com/mattpocock/skills
- `planning-with-files` — https://github.com/OthmanAdi/planning-with-files

Design lineage (incremental progress, files-as-handoff, clean-state-per-session, role separation):

- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (Anthropic) — the "one feature at a time" rule discussed under Parallel execution.
- [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Anthropic) — context resets vs compaction, separating the builder from a skeptical evaluator; the lineage behind the adversarial-review gate.
- [Simon Last — lessons on running coding agents at scale](https://x.com/simonlast/status/2057978156183957995) — self-contained plan docs over babysitting, adversarial review for unattended runs; the impetus for the review gate and clean-context-per-issue.
