# Reference: swe-workflow

Detailed notes on each stage. Read this when something feels off, not on every invocation.

## Stage 0: `/setup-matt-pocock-skills` — How is this repo set up?

**Trigger**: a fresh repo (or one that hasn't adopted swe-workflow), with no `## Agent skills` block in `AGENTS.md`/`CLAUDE.md` and no `docs/agents/` directory.

One-time per-repo bootstrap. It wires the toolchain into a specific repo by writing:

- `## Agent skills` block in `AGENTS.md` (or `CLAUDE.md` if that's where the team keeps agent instructions) — declares which skills this repo uses and points at their per-repo docs.
- `docs/agents/` directory — repo-local agent documentation: which issue tracker is in use (GitHub or local markdown), the triage label vocabulary, and the layout for domain docs (`CONTEXT.md`, ADRs, `FEATURES.md`).

Downstream skills read this to know the repo's conventions: `/triage` learns which labels are valid here, stage 5 bootstrap learns where to fetch issues from (the canonical repo record at rung 3 of [tracker selection](trackers/README.md#selection--which-adapter)), `/grill-with-docs` learns whether `CONTEXT.md` should live at the root or under `src/<bounded-context>/`.

**Skip this stage** when the repo already has the `## Agent skills` block and `docs/agents/` populated — running it again would clobber existing config. This is a *per-repo* bootstrap, not *per-feature*; once it's run, you don't run it again.

**Not the same as spec-kit's "constitution" stage.** This is *tooling configuration* (where to fetch, what labels exist, where domain docs go), not *architectural invariants* (which patterns to follow, what the system must do). Those live in `CONTEXT.md` and ADRs and are produced by stage 1 — see the [framework comparison](#how-this-differs-from-spec-kit-class-frameworks) for why the distinction matters.

## Stage 1: `/grill-with-docs` — What do I want?

**Trigger**: vague language, fuzzy boundaries, no shared vocabulary yet.

The session interviews you about every branch of the design tree, one question at a time. Outputs:

- `CONTEXT.md` (project root or `src/<bounded-context>/`) — pure glossary, no implementation details
- `docs/adr/NNNN-*.md` — only for substantive architectural decisions

If a `CONTEXT-MAP.md` exists at repo root, the project uses multiple bounded contexts; each has its own `CONTEXT.md`.

**Re-runnable until quiet — or until you call it.** `/grill-with-docs` isn't one-shot: run it repeatedly, each pass sharpening `CONTEXT.md` (and possibly adding ADRs). Two things end the loop — a run surfaces **no more interview questions** (terminology has stabilized), or **you abort the interview** (you've heard enough). Either way, what's already in `CONTEXT.md`/ADRs stands; aborting forfeits only the unasked questions, not captured decisions.

**Skip this stage** when the terminology is settled and no new concepts are being introduced.

## Stage 2: `/to-features` — What features does this break into?

The product→engineering bridge — the **product-manager** step. `/to-features` **invokes `/grill-with-docs` at a high (product) level** to split the project into **coarse-grained** user-facing features, then writes `FEATURES.md`. It grills rather than reading files because `CONTEXT.md` (a glossary) and `docs/adr/` (sparse architectural calls) don't enumerate features — the set must be elicited. The distinct **stage-1 domain grill runs first**; this builds on it but aims at *features*, not vocabulary. AFK-friendly and pausable — recommended answers via the `log-decisions` rules, an unsure HITL call pauses to escalate, feature-scope calls journaled.

The full process, the `FEATURES.md` file format (the per-feature detail block), the **strike-through-don't-delete** discipline on ship, the optional `.scratch/<feature>/` filesystem stub for local-markdown teams, and when to skip the stage all live in the **[`to-features.md` procedure](references/to-features.md)** — its home, now its own command (`/swe-workflow:to-features`). Not restated here.

## Stage 3: `/grill-feature` — What does done look like?

**Trigger**: a feature is picked from `FEATURES.md` and needs its spec.

The second **product-manager** step — now its own command (`/grill-feature`) and procedure ([`references/grill-feature.md`](references/grill-feature.md)): **grill one feature, then synthesize its PRD.** `/spec` runs it inline for the feature it's targeting; `/grill-feature` runs it standalone, once per feature.

1. **Grill the feature** — `grill-with-docs` scoped to the one feature (an **intensive, feature-level** interview — deeper than `to-features`' high-level project grill), grounded on `CONTEXT.md`/ADRs.
2. **Synthesize** — `to-prd` turns that conversation into **one PRD** without re-interviewing (the grill is what gave it the material). The PRD contains:
   - Problem Statement / Solution (user perspective)
   - User Stories (extensive, numbered)
   - Implementation Decisions (modules, interfaces, schemas — NO file paths)
   - Testing Decisions (which modules get tested, prior art)
   - Out of Scope

   Published as a parent issue with the `ready-for-agent` triage label. One feature → one PRD; **skip** if one already exists.

**Common mistake**: putting file paths or line numbers in the PRD. Those rot. Describe interfaces and behavior instead. Full procedure: [`references/grill-feature.md`](references/grill-feature.md).

## Stage 4: `/to-issues` — What are the units of work?

**Trigger**: PRD exists but is too big for any single agent to grab.

Breaks the PRD into **tracer-bullet** issues — thin vertical slices that cut through every layer (schema → API → UI → tests). For each slice:

- Title, type (HITL/AFK), blocked-by chain, user stories covered
- Issue body: parent reference, what-to-build (behavioral), acceptance criteria, blocked-by

You'll be quizzed before publishing: is the granularity right? Are dependencies correct? Are the right slices marked HITL vs AFK?

- **HITL** (human-in-the-loop) — needs architectural decision or design review during build.
- **AFK** (away-from-keyboard) — fully specified, agent can complete unattended.

Prefer AFK when possible. HITL is for genuine judgment calls, not for "I want to review every PR."

## Parallel concern: `/triage` — What's actionable for external issues?

**`/triage` is NOT a sequential stage in the chain.** `/to-prd` and `/to-issues` auto-apply the `ready-for-agent` label on the issues they create — those issues skip triage entirely and go straight to stage 5. `/triage` exists for issues filed *outside* the chain: user bug reports, external contributions, ad-hoc feature requests.

**Trigger**: an external issue arrives in the tracker without a triage label, OR an existing labeled issue needs re-evaluation after new info.

A small state machine: every issue carries one **category** (bug / enhancement) and one **state** (needs-triage, needs-info, ready-for-agent, ready-for-human, wontfix).

Outputs to issues:
- `ready-for-agent` → AGENT-BRIEF comment (the durable execution contract); from here the issue enters stage 5 like any chain-created issue
- `ready-for-human` → same structure, with rationale for why agents can't handle it
- `needs-info` → triage notes with specific outstanding questions
- `wontfix` (enhancement) → entry in `.out-of-scope/<concept>.md` before closing

Every triage comment starts with `> *This was generated by AI during triage.*`

`/triage` and the chain share vocabulary (`ready-for-agent` etc.) but not invocation — they're independent entry points into the same issue tracker.

## Stage 5: worktree + `/planning-with-files` — Build it

The skill is **instructions-only** — there are no scripts. The agent performs the bootstrap manually, adapting to the team's issue tracker (see [trackers/](trackers/)).

### Bootstrap

The numbered bootstrap — fetch → derive paths → create the worktree (with the **idempotent re-run check**) → seed `task_plan.md` / `findings.md` / `progress.md` → invoke `planning-with-files`' `plan` then `plan-goal` — is single-sourced in the [ship procedure](references/ship.md) (Stages 5–6). Don't restate the steps here; this file carries only the *why* and the edge cases below.

The split enforces the **security boundary**: `task_plan.md` is re-injected by hooks every tool call, so it gets only structured fields; raw external content (issue body, fetched docs) goes to `findings.md` instead. `/karpathy-guidelines` also carries a repo-wide standing form in the agent-instructions file; `/tdd` stays scoped to `/ship`+`/ship-all` builds — see [Always-on engineering rules](#always-on-engineering-rules) for the split.

### Tracker selection

The selection algorithm (priority order), the normalized-issue schema, AGENT-BRIEF resolution, branch/worktree naming, and the per-adapter fetch/list operations all live in the **[tracker contract](trackers/README.md)** — the single interface every `trackers/<name>.md` adapter satisfies. Adding a tracker = one new adapter doc; nothing else changes.

### Inner loop: `/tdd` within each code-producing phase

`/planning-with-files:plan-goal` is the outer loop; `/tdd` is the inner loop. For each phase in `task_plan.md` that produces testable code:

1. Mark the phase `in_progress` in `task_plan.md`.
2. Invoke `/tdd`:
   - **RED** — write the failing test for this phase's behavior
   - **GREEN** — minimal code to pass
   - **REFACTOR** (optional)
3. Log the cycle outcome in `progress.md`.
4. Mark the phase `complete` in `task_plan.md`.
5. Move to the next phase.

**Nuances:**

- **Not every phase needs `/tdd`.** Exploration, `.gitignore` updates, config tweaks, infra changes have no behavior to test — just do them and log.
- **A single phase can contain multiple `/tdd` cycles** if the AC bundles sub-behaviors. Often a sign the AC was too coarse — note it in `task_plan.md` so the next `/to-issues` run can slice finer.
- **Decisions discovered mid-`/tdd`** split by the bar: reversible, spec-authorized ones (e.g., extracting a private helper) land in `task_plan.md`'s Decisions table; **bar-crossing** ones — and a public-interface change is a *deviation* — go to the **decision journal** (`DECISIONS.staged.md` → `DECISIONS.md`) per the `log-decisions` skill (look first → decide / assume / escalate).
- **Errors during RED or GREEN** go in `task_plan.md`'s Errors table — same 3-strike protocol as any other error. Never silently retry a failed test in the same form.
- **`/tdd`'s own per-phase planning step is light** (interface confirmation, test list, prior-art lookup). It's not a duplicate of `/planning-with-files:plan`'s issue-level interview — just a quick check-in at the top of each phase.
- **Compile vs runtime.** Stage 5 *compiles* the plan (the interview bakes `/tdd` + `/karpathy-guidelines` into `task_plan.md`); Stage 6 *runs* it. In **AFK** builds `/tdd` does **not** interview the human — mid-build decisions are logged (`log-decisions`) — a grounded call or a flagged **assumption** — or escalated. **HITL** issues may pause at documented checkpoints.

**Mental shorthand**: `/planning-with-files` is the project manager keeping the calendar; `/tdd` is the engineer at the keyboard. One PM, many keyboard sessions per issue.

### Discovered scope

Work you hit mid-build that this issue's slice didn't plan for. **Don't grow the worktree** (it breaks tracer-bullet slicing and bloats the PR), and don't just jot it in `task_plan.md` / `progress.md` — those die at teardown, so the note is lost. Route it to a durable home, then keep building:

- **Belongs to this AC** (you under-estimated the slice) → do it now; if the AC was bundling behaviors, note that in `task_plan.md` so the next `/to-issues` slices finer.
- **A new issue** (bug, follow-up, separate slice) → file it to the tracker (a `.scratch/<feature>/issues/` stub, or `gh issue create` / Linear); `/triage` classifies it later. Set `blocked-by` if it depends on this work.
- **A new feature** → add a line to `FEATURES.md`, specced later via `/to-prd`.
- **Rejecting it** → `.out-of-scope/<concept>.md` with the reason.

Filed work won't disrupt an AFK batch — only `ready-for-agent` issues enter execution. Expanding the current issue to absorb a discovery is itself a *deviation* (log it per `log-decisions`); **escalate** only if the discovery blocks this issue and needs a human.

### When to skip stage 5 bootstrapping

For trivial AFK issues (rename a flag, fix a typo, bump a version), just do it. The hook overhead of `/planning-with-files` is unjustified for <5 tool calls.

### HITL execution

If the issue is `ready-for-human` or the AGENT-BRIEF flags HITL checkpoints:
- Don't let the planning-with-files Stop hook auto-finish you.
- Pause at the documented checkpoint, surface the decision, wait for direction.
- Resume by re-invoking `/planning-with-files` after the human responds — it re-reads the updated `task_plan.md`.

### Teardown after PR merge

From the **main checkout** (NOT inside the worktree):

```bash
# 1. Verify no uncommitted changes
git -C ../<repo>-issue-<id> status --porcelain    # output must be empty

# 2. Remove the worktree
git worktree remove ../<repo>-issue-<id>

# 3. Delete the branch only if merged into the default branch
default_branch=$(git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@')
if git branch --merged "$default_branch" | grep -qE "^[[:space:]]*\*?[[:space:]]*issue-<id>-<slug>$"; then
  git branch -d "issue-<id>-<slug>"
fi
```

Don't run this from inside the worktree being removed — git rejects that.

If you want the planning files as a post-mortem record, commit them on the branch BEFORE the teardown — the PR will carry them. Default is to discard.

### Decision journal (`DECISIONS.md`)

Bar-crossing decisions made during a build — gate-resolutions, deviations, tradeoffs, irreversible-action calls, flagged assumptions, escalations — are recorded by the `log-decisions` skill, **not** in the ephemeral `task_plan.md` Decisions table (which keeps only reversible execution decisions). The flow:

- **During the build (in the worktree):** append entries to `DECISIONS.staged.md` — plugin-owned, gitignored, ephemeral. Resolve each call per the **`log-decisions`** rules (look first → decide / assume / escalate, with the catastrophic floor that always escalates). In a build an escalation **parks for batch review — it doesn't pause** (the park model below).
- **At close-out (Stage 7, from the main checkout):** promote the staged entries into repo-root `DECISIONS.md` and commit them as their own `log:` commit. Serialized by teardown, so parallel worktrees never conflict. The PR body also carries an "Autonomy decisions" section for pre-merge visibility.
- `DECISIONS.md` is **settled history only** (append-only) — including flagged **assumptions** (`assumed`), which promote and surface in the PR body for async review; open **escalations** stay in their worktrees, surfaced by `/status`.
- **Escalations park, they don't halt.** In `/ship-all`, a mid-build escalation parks that one issue (worktree intact, dependents skipped) and the batch continues with independents; a build *failure* (3-strike) halts. `/status` from the main checkout aggregates open escalations across worktrees, so you resolve them in a batch.

**Journal vs ADR.** The journal records *events* ("on this date, this was decided, by whom"); `docs/adr/` records *ratified architecture* ("this IS the decision now"). A journal entry that proves architecturally significant is **promoted to an ADR manually** — never automatically — and the entry notes the promotion. See the `log-decisions` skill for the entry schema and rules.

## Parallel execution

`/ship-all` runs the backlog **sequentially — one worktree at a time** ([`references/ship-all.md`](references/ship-all.md)); it never auto-fans-out. The worktree-per-issue isolation ([handoff rule #4](SKILL.md#critical-handoff-rules)) exists to make *optional, human-driven* parallelism **safe**, not to have the loop run issues concurrently. To parallelize, a human starts a separate `/ship` (or a feature-scoped `/ship-all`) per independent unit, each in its own session.

**This doesn't contradict "one feature at a time"** (Anthropic, *Effective harnesses for long-running agents* — see [Further reading](#further-reading)). That rule bounds *what a single agent-iteration takes on* — don't one-shot a whole app and exhaust the context mid-feature — not *how many agents run at once*. Each worktree here is bound to one **issue** (a tracer-bullet slice, finer than the article's "feature"), so the bound holds whether one or several run. The violation to avoid is the inverse: one agent swallowing multiple issues to save worktrees, which rule #4 forbids.

**Parallelize across independent features, not slices of the same feature:**

| Across… | Verdict | Why |
|---|---|---|
| Independent features (different PRDs, no `blocked-by`, disjoint code) | Safe — the sweet spot | Small disjoint diffs, no dependency ordering, low merge-conflict surface. |
| Tracer-bullet slices of the *same* feature | Usually unsafe | Slices are deliberately *vertical* (schema → API → UI → tests, handoff rule #2), so two slices of one feature tend to touch the same schema/types/routing; a natural build order often `blocked-by`-serializes them anyway. |

**The execution failure modes mutate under parallelism — they don't vanish:**

- *Leave a clean state* becomes *leave a clean state **and** a clean merge.* Each agent leaves its own worktree mergeable, but none sees the integration — concurrent agents both plan against `main`-as-it-is-now, not against each other's uncommitted work.
- *Verify the baseline first* assumes a **stable** `main`. Under parallelism the baseline moves as peers merge, so an agent can build against an already-stale snapshot and not know it.

**What the toolchain already makes parallel-safe** — leaving concurrent edits to shared *source* as the only real residual risk, which is git's to resolve, not the toolchain's:

- `DECISIONS.staged.md` is per-worktree and gitignored, promoted to `DECISIONS.md` serialized by teardown — conflict-free across worktrees (see [Decision journal](#decision-journal-decisionsmd)).
- [`/status`](references/status.md) aggregates open escalations across worktrees, so one human can supervise several concurrent AFK ships.
- Worktree + branch + planning-file isolation means no shared mutable state *during* a build.

**Guardrail:** parallelize only units with **no `blocked-by` relationship and minimal file overlap**; keep same-feature vertical slices sequential.

## Always-on engineering rules

A few skills apply to *every* run rather than to one stage. There's no framework toggle — each is a **named-skill-activation rule** written into a file that's already in context where it bites, so the agent reads the rule and invokes the skill. Two things decide *which* file: the rule's **scope** and the matching **injection site**.

| Skill | Scope | Injection site(s) |
|---|---|---|
| `log-decisions` | All stages (spec *and* build) | setup procedure §3 → `AGENTS.md`/`CLAUDE.md` sentinel block |
| `andrej-karpathy-skills:karpathy-guidelines` | Any code-writing (ad-hoc *and* build) | setup §3 standing rule **+** ship Stage 5 planner → `task_plan.md` |
| `/tdd` | ship + ship-all builds only | ship Stage 5 planner → `task_plan.md` (per-issue, so ship-all inherits it) |

**The site follows the scope** — each is the file already loaded at the moment the rule applies:

- **All-stages → the agent-instructions file** (`AGENTS.md`/`CLAUDE.md`). Agent-agnostic — Claude and Codex both read it through the `## Agent skills` block [stage 0](#stage-0-setup-matt-pocock-skills--how-is-this-repo-set-up) writes — persistent across features, and reloaded into every fresh context window. So `log-decisions` (fires during spec as readily as a build) and `karpathy-guidelines` (every code edit, ship or not) stay present throughout. Written once by the setup procedure, sentinel-wrapped so a re-run is a no-op.
- **Execution-only → `task_plan.md`.** `planning-with-files` re-injects it on every tool call, keeping the rule maximally salient through the build, and it dies with the worktree — the right lifetime for a rule that means nothing until code is being written. Only structured rules the executor wrote belong here (the **security boundary**); a named-skill activation qualifies, raw external text never does.

**Karpathy sits in both sites; `/tdd` in one — by deliberate scope.** `karpathy-guidelines` is repo-wide: the setup standing rule states the policy (*keep every change surgical — ad-hoc edits included*), and the ship planner prompt re-names it in `task_plan.md` so it stays salient through a build (complementary, not double-tracked — one declares, the other re-fires at the keyboard). `/tdd` is scoped to ship and ship-all builds only — it lives **solely** in the planner prompt → `task_plan.md`, never in the standing block, so opening the repo for a quick edit doesn't force red → green → refactor. Flip either knob in one place: widen `/tdd` by adding it to the setup engineering-discipline block; narrow `karpathy` by dropping its standing rule.

**Adding a rule later** — add a row, then route it to its site: all-stages → extend the [setup procedure](references/setup.md)'s §3 sentinel block; execution-only → extend the [ship procedure](references/ship.md)'s Stage 5 planner prompt (its single source, kept there so it can't drift). This table is the registry; those two sites are the only writers. The same property that lets a new *tracker* be one adapter doc lets a new *rule* be one row plus one injection site.

## Completion signals

Four levels of "done"; the toolchain provides explicit semantics for three of them and deliberately stays out of the fourth.

### Phase

A phase in `task_plan.md` is done when its `/tdd` cycle lands green (or, for non-code phases, when the work is logged in `progress.md`). Recorded as a ticked checkbox in `task_plan.md`. Local to the worktree — dies at teardown.

### Issue

All phases in `task_plan.md` ticked **and** PR merged into the default branch. The merge event is the durable signal — `task_plan.md` itself is ephemeral. The tracker carries the persistent "closed/merged" status.

### Feature

Every issue produced by `/to-issues` on the feature's PRD has merged. Workflow:

1. **Detect**: walk from the PRD to its child issues (via the parent reference each issue body carries) and confirm all closed.
   - **GitHub**: `gh issue list --search "parent:<PRD#> state:open"` returns empty.
   - **Local-markdown**: every file in `.scratch/<feature-slug>/issues/` has a terminal `Status:` (e.g., `closed`).
   - **Linear / GitLab / Multica**: tracker-specific child-issue query.
2. **Mark**: strike through the `FEATURES.md` line and append shipped refs:
   ```
   - [x] ~~user-can-reset-password~~ — ~~A user can reset a forgotten password~~ (shipped: #42, #43, #44)
   ```
3. **Never delete the line** (see the [`to-features.md` procedure](references/to-features.md)) — strike-through preserves institutional memory and prevents quiet scope drift.

### Project

No native concept. The toolchain has `FEATURES.md` (per-repo backlog) but deliberately no `PROJECT.md`. Reasons:

- **Software projects rarely "complete".** Features keep getting added; the planned scope is always shifting.
- **Project completion is policy, not a fact.** "Are we done?" depends on release cadence, business commitments, and milestone definitions — none of which the toolchain has visibility into.

If you need a hard project-level signal, layer it on top of your tracker (GitHub milestones, Linear cycles, release tags) and define "project complete" as that milestone closing. The closest signal the toolchain itself provides is the state of `FEATURES.md` at a moment in time — zero `- [ ]` lines = every currently-enumerated feature has shipped — but this is a snapshot, not a guarantee. Adding a line resets it.

The asymmetry is intentional: phase/issue/feature completion is a **fact** the toolchain can verify; project completion is a **judgment call** that lives outside.

## Gotchas

- **Don't re-litigate PRD decisions in `task_plan.md`.** Architecture choices ("Postgres not Redis") are upstream. The `task_plan.md` Decisions table is only for *new* decisions made during execution (e.g., "extracted a CronExpression validator").
- **Don't `/to-issues` an issue you've already started executing.** That forks state. If a `ready-for-agent` issue turns out to need decomposition, send it back to triage as `needs-info` or split it via a new `/to-issues` run on the PRD parent.
- **Worktree path collisions.** If `../<repo>-issue-<id>/` already exists, the bootstrap aborts. Either you didn't tear down a previous attempt, or someone else is on the same issue. Resolve before retrying.
- **Tracker auth.** Each tracker has its own auth (`gh auth status`, `glab auth status`, `linear auth`, `multica` config, etc.). Verify before bootstrap — the fetch step fails fast if auth is missing.
- **`tp` skill conflict.** If you have the `tp` skill loaded, it overlaps with `/planning-with-files` at the execution layer. Pick one. This workflow assumes `/planning-with-files`.
- **Teardown location.** Run the teardown commands from the main checkout, NOT from inside the worktree being removed — git rejects removing the worktree you're currently in.

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

A [community survey (~2000 AI coding course participants, April 2026, by Matt Pocock)](https://x.com/mattpocockuk/status/2044029094942159126) flagged **framework opacity** as the dominant failure mode for spec-kit-class tools: debugging is hard when you can't see what went into context. Specific failure reports from the same thread:

- *"Spec Kit outright ruined the process at the last place at the point we needed to ship the most"*
- *"Most of these frameworks optimize for demos, not debugging. The moment context goes wrong, everything falls apart"*
- *"The moment you hand context over to a 'workflow' the output drifts within days"*

This rules out some patterns from spec-kit-class even when they look useful:

- **A spec-kit-style "constitution" stage** (a dedicated step for project-level *architectural* invariants — patterns to follow, rules the system must obey) was considered and rejected. Architectural invariants already live in `CONTEXT.md` and `docs/adr/` (produced by `/grill-with-docs`); a separate constitution stage would import the framework opacity this chain claims to fight. swe-workflow's own [Stage 0](#stage-0-setup-matt-pocock-skills--how-is-this-repo-set-up) is a different beast — pure *tooling configuration* (tracker name, label vocabulary, doc layout), orthogonal to architectural rules.
- **Framework-driven implementation** (like `/speckit.implement`) is replaced by the layered worktree + `/planning-with-files` + `/tdd` execution stage. More moving pieces, but each is observable.
- **Long-lived spec artifacts in the repo** are avoided. Planning files (`task_plan.md`, `findings.md`, `progress.md`) live in worktrees and die at PR merge. Only the issue tracker and `CONTEXT.md`/ADRs survive across features.

## Source skills

- `grill-with-docs`, `to-prd`, `to-issues`, `triage` — https://github.com/mattpocock/skills
- `planning-with-files` — https://github.com/OthmanAdi/planning-with-files

## Further reading

Anthropic Engineering on long-running agent harnesses — the design lineage this suite operationalizes (incremental progress, files-as-handoff, clean-state-per-session):

- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — initializer + coding agent, a feature list, one-feature-at-a-time incremental progress, leave-it-clean-per-session — the source for the "one feature at a time" rule discussed in [Parallel execution](#parallel-execution).
- [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) — a planner/generator/evaluator multi-agent architecture, context resets vs compaction with structured handoffs, and separating the builder from a skeptical evaluator.
