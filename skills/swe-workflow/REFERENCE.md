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

The product→engineering bridge — the **product-manager** stage. `/to-features` **invokes `/grill-with-docs` at a high (product) level** to split the project into **coarse-grained** user-facing features, then writes `FEATURES.md`. It grills rather than reading files because `CONTEXT.md` (a glossary) and `docs/adr/` (sparse architectural calls) don't enumerate features — the set must be elicited. The distinct **stage-1 domain grill runs first**; this builds on it but aims at *features*, not vocabulary. AFK-friendly and pausable — recommended answers via the `log-decisions` rules, an unsure HITL call pauses to escalate, feature-scope calls journaled.

The full process, the `FEATURES.md` file format (the per-feature detail block), the **strike-through-don't-delete** discipline on ship, the optional `.scratch/<feature>/` filesystem stub for local-markdown teams, and when to skip the stage all live in the **[`to-features.md` procedure](references/to-features.md)** — its home, now its own command (`/swe-workflow:to-features`). Not restated here.

## Stage 3: `/grill-feature` — What does done look like?

**Trigger**: a feature is picked from `FEATURES.md` and needs its spec.

The second **product-manager** stage — now its own command (`/grill-feature`) and procedure ([`references/grill-feature.md`](references/grill-feature.md)): **grill one feature, then synthesize its PRD.** `/spec` runs it inline for the feature it's targeting; `/grill-feature` runs it standalone, once per feature.

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
- **User-facing phases need more than unit-green.** A passing unit test proves a unit, not that the feature works for a user — so a user-visible phase isn't "done" until its behavior is exercised end-to-end, not just unit-tested.
- **A single phase can contain multiple `/tdd` cycles** if the AC bundles sub-behaviors. Often a sign the AC was too coarse — note it in `task_plan.md` so the next `/to-issues` run can slice finer.
- **Decisions discovered mid-`/tdd`** split by the bar: reversible, spec-authorized ones (e.g., extracting a private helper) land in `task_plan.md`'s Decisions table; **bar-crossing** ones — and a public-interface change is a *deviation* — go to the **decision journal** (`DECISIONS.staged.md` → `DECISIONS.md`) per the `log-decisions` skill (look first → decide / assume / escalate).
- **Errors during RED or GREEN** go in `task_plan.md`'s Errors table — same 3-strike protocol as any other error. Never silently retry a failed test in the same form.
- **`/tdd`'s own per-phase planning step is light** (interface confirmation, test list, prior-art lookup). It's not a duplicate of `/planning-with-files:plan`'s issue-level interview — just a quick check-in at the top of each phase.
- **Compile vs runtime.** Stage 5 *compiles* the plan (the interview bakes `/tdd` + `/karpathy-guidelines` into `task_plan.md`); Stage 6 *runs* it. In **AFK** builds `/tdd` does **not** interview the human — mid-build decisions are logged (`log-decisions`) — a grounded call or a flagged **assumption** — or escalated. **HITL** issues may pause at documented checkpoints.

**Mental shorthand**: `/planning-with-files` is the project manager keeping the calendar; `/tdd` is the engineer at the keyboard. One PM, many keyboard sessions per issue.

### Clean context per issue

The worktree isolates *files* per issue; the agent's *context* is the one piece that doesn't reset between issues. Because the planning files are the whole handoff — the security boundary and *files are the interface* make that literal — the per-issue work needs nothing carried over in-context, so each issue can, and ideally should, run in a **fresh context**.

**Recommended, host-neutral, not a mandated step.** Where the host supports it, run the per-issue work (seed → plan → build) in a fresh context rooted in the worktree — a sub-agent, or a new CLI session (`claude`, `codex`, `gemini`, …) launched once the worktree exists. Where it isn't, lean on **compaction**: modern compaction retains conventions across a long session, so one long-running session stays clean enough — a hard per-issue reset is an optimization, not a requirement. This applies the "initializer + coding agent / clean-state-per-session" lineage in [Further reading](#further-reading) at the granularity of one issue.

Two limits keep it from over-reaching:

- **It stays guidance, never a scripted spawn.** Mandating a launch command would break the cross-agent, scriptless contract — the command, headless/interactive mode, auth, and session lifecycle all differ per host. The procedure asks for *a clean context*; how you obtain one is the host's business.
- **AFK vs HITL.** A fresh *headless* context fits AFK issues; a **HITL** issue needs an *interactive* context so its documented checkpoints can still pause. Either way `/ship-all` stays [sequential](#parallel-execution) — clean-context-per-issue is context hygiene, not concurrency.

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

### Adversarial review (before close-out)

The procedure is a single step — [ship.md](references/ship.md), Stage 7 step 8. What that step leaves implicit — the *why* and the edge cases:

- **The failure it guards against.** The most-cited unattended-run failure is *trusting the agent's own summary of what it did* — `progress.md` is the implementer narrating its own success. Hence the step's two non-negotiables: a **separate** context (not the author) that is **read-only** (so it can't quietly fix-and-pass), there to read actual state rather than re-read the narrative. Separation of duties — a reviewer who can't merge their own PR.
- **It's the autonomy gate.** A clean pass is what lets an **AFK** issue complete *without a human merge* — the conductor merges, tears down, and marks it done on the green light, and the human spot-checks afterward via the PR + its Autonomy-decisions record. It's the only thing standing between a build and the default branch, so the bar is real — and for user-facing AC the bar is **observed behavior, not a plausible diff**: code review and unit tests miss bugs that only surface when you run it. (HITL waits for a human; a gap loops back to Stage 6; a parked escalation never reaches close-out.)
- **Complementary to `/tdd`, not a duplicate.** `/tdd` is the implementer's per-phase self-check (red → green as each phase is built); the review is a separate whole-diff cross-check at the end. One proves each phase in isolation; the other asks whether the finished change, in total, satisfies the issue.
- **Inside the security boundary.** If the reviewer needs the issue's raw acceptance text, it reads `findings.md` — never the re-injected `task_plan.md`.
- **Idempotent.** Read-only and side-effect-free, so an interrupted run re-runs it safely.
- **Fresh per diff.** The reviewer mirrors the build's rule one level down — [clean context per issue](#clean-context-per-issue) applied per *diff*: a fresh read-only context for each, never one long-running reviewer accumulating reviews (the reviewer's own ball of mud).
- **Host-neutral contract.** "A fresh read-only context" is all the procedure asks for; *how* you get one — sub-agent vs. new session — is the host's business.

### Landing the work

Close-out lands the **branch** — *the worktree is just a second checkout of it, irrelevant here* (it matters only at teardown). Two habits first: **rebase onto the target** (`git rebase <target>`) so conflicts resolve before review rather than during the merge, and match the repo's **merge convention** (merge / squash / rebase-and-merge). **How** to land is keyed on repo topology + branch protection — *not* the issue tracker:

| Topology | Land by |
|---|---|
| Fork, no upstream write | PR `fork:<branch> → upstream:<target>`; never route through the fork's `main` (keep it mirroring upstream) |
| Single repo, protected/shared `main` | PR `<branch> → main` |
| Single repo, solo / unprotected | rebase + `git merge --ff-only` onto `main`, then push |

Check the **target branch** first — not every project lands on `main` (some use `develop`/`next`/a release branch; read `CONTRIBUTING.md`). The PR, when one is used, carries the `progress.md` body + the Autonomy-decisions section and is the durable record. Whether the issue then reads *done* is the separate, tracker-keyed concern — [Lifecycle states](trackers/README.md#lifecycle-states).

### Teardown after landing

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

# 4. If the branch was pushed (PR landings), delete it on the remote too
git push origin --delete issue-<id>-<slug>
```

Don't run this from inside the worktree being removed — git rejects that.

If you want the planning files as a post-mortem record, commit them on the branch BEFORE the teardown — the landing carries them. Default is to discard.

### Decision journal (`DECISIONS.md`)

Bar-crossing decisions made during a build — gate-resolutions, deviations, tradeoffs, irreversible-action calls, flagged assumptions, escalations — are recorded by the `log-decisions` skill, **not** in the ephemeral `task_plan.md` Decisions table (which keeps only reversible execution decisions). The flow:

- **During the build (in the worktree):** append entries to `DECISIONS.staged.md` — plugin-owned, gitignored, ephemeral. Resolve each call per the **`log-decisions`** rules (look first → decide / assume / escalate, with the catastrophic floor that always escalates). In a build an escalation **parks for batch review — it doesn't pause** (the park model below).
- **At close-out (Stage 7, from the main checkout):** promote the staged entries into repo-root `DECISIONS.md` and commit them as their own `log:` commit. Serialized by teardown, so parallel worktrees never conflict. The PR body also carries an "Autonomy decisions" section for the human's post-merge spot-check.
- `DECISIONS.md` is **settled history only** (append-only) — including flagged **assumptions** (`assumed`), which promote and surface in the PR body for async review; open **escalations** stay in their worktrees, surfaced by `/status`.
- **Escalations park, they don't halt.** In `/ship-all`, a mid-build escalation parks that one issue (worktree intact, dependents skipped) and the batch continues with independents; a build *failure* (3-strike) halts. `/status` from the main checkout aggregates open escalations across worktrees, so you resolve them in a batch.

**Journal vs ADR.** The journal records *events* ("on this date, this was decided, by whom"); `docs/adr/` records *ratified architecture* ("this IS the decision now"). A journal entry that proves architecturally significant is **promoted to an ADR manually** — never automatically — and the entry notes the promotion. See the `log-decisions` skill for the entry schema and rules.

## Session topology

How to map the 0→7 chain onto agent CLI sessions — host-neutral operator guidance, not a mandated mechanism. **Files are the interface**, so a session can end wherever a durable artifact exists and the next one resumes from it (`CONTEXT.md` / `FEATURES.md` / PRD / issue + AGENT-BRIEF). But those artifacts are a *lossy* compression of the grilling conversation — rejected alternatives, the "why we landed here," the priority feel don't all survive. So every cut trades **context warmth** (fidelity to intent) against **isolation** (clean context, observability, parallelism, a fresh context for the [adversarial-review](#adversarial-review-before-close-out) gate). Three points on that spectrum:

**Option 1 — one long-running session for the whole project.** *Concurrency: none — every stage, feature, and issue runs sequentially* in a single session (execution here is `/ship-all` run inline).

- *Buys:* maximum warmth and zero setup — domain understanding stays hot; compaction holds conventions.
- *Costs:* the "ball of mud" the suite exists to avoid — by feature 3 the context is clogged with feature 1's grill and feature 2's PRD, and compaction across *unrelated* work drops the wrong things; the author also reviews its own work.

**Option 2 — one session per feature.** *Concurrency: feature-level — independent features run in parallel.* Spec the top of the chain once, then each feature owns stages 3→7.

1. **Stage 0** alone, then exit.
2. **Stage 1** (`/grill-with-docs`) — a fresh session or a continuation of stage 0, either works — then **Stage 2** (`/to-features`) in the *same* session, since `/to-features` builds on the stage-1 domain grill.
3. **A session per dependency-free feature, launched as blockers clear** — start a feature's session only when its `Depends on:` features (in `FEATURES.md`) are **done** (all their issues merged — see [Completion signals](#completion-signals)); a feature with unmet blockers waits and gets its session once they land. Each session runs `/grill-feature` (stage 3) → `/to-issues` (stage 4) → a **feature-scoped [`/ship-all`](references/ship-all.md)** (stages 5–7), shipping that feature's issues **one at a time inside the one session** — each `/ship` spins up its worktree, the session `cd`s in to build, then tears it down before the next. The session persists while the worktrees come and go.

- *Buys:* the **lowest information loss across the spec→execution seam** — the feature's grilling context stays warm while its issues are built.
- *Costs:* a feature's issues share one context (a smaller ball of mud — and by the last issue the grill context you wanted is buried under earlier issues' build noise); spec and build share one context and security posture; no per-issue parallelism.

**Option 3 — one session per issue.** *Concurrency: issue-level — independent issues run in parallel.* Option 2's spec handling, but the feature session ends at the backlog.

1. As Option 2 through stages 3–4, but **exit each feature session once `/to-issues` has run** — PRD + issues + AGENT-BRIEF are the handoff.
2. **A session per unblocked issue, run in parallel** — for each issue whose `blocked-by` have merged, spin up its own session and [`/ship`](references/ship.md) it; as dependencies land, newly-unblocked issues get their own. This is Option 2's `/ship-all` loop **unrolled and fanned out** — one `/ship` per session, [clean context per issue](#clean-context-per-issue).

- *Buys:* per-issue isolation — each issue builds only from its durable inputs, a fresh reviewer is cheap, and state stays durable+observable rather than fragile-and-warm.
- *Costs:* the handoff leans entirely on the artifact, so a thin AGENT-BRIEF surfaces as a build gap instead of being silently covered by warm context. The remedy is a **richer artifact, not a longer session**: push the tacit into the AGENT-BRIEF / seed `findings.md`, and mark genuinely context-hungry issues **HITL** so a human fills the gap at the checkpoint.

**Choosing.** Default to **Option 3** for AFK / observable / parallel-ready work: it honors every execution-layer invariant (worktree-per-issue, clean context per issue, the review gate) and keeps the *high-loss* boundary (grill → PRD → issues) warm in one spec session while cutting only at the *low-loss* one (issues → build). Choose **Option 2** when intent-fidelity outweighs isolation and the features are small, tightly-knit, and interactive. **Option 1** is for toy scope. Picking Option 2 or 3 only sets the *granularity* of concurrency; which of those concurrent runs is actually *safe* is the subject of [Parallel execution](#parallel-execution) (independent units only). The throughline: *the cure for a lossy artifact is a better artifact, not a longer session* — the long session's richness isn't durable, and the suite is built for work that outlives any one session.

### Parallel execution

The safety rulebook for running Option 2 or 3 sessions concurrently — which granularity is safe, and what breaks when you do. `/ship-all` itself runs the backlog **sequentially — one worktree at a time** ([`references/ship-all.md`](references/ship-all.md)) and never auto-fans-out; the concurrency is always *optional and human-driven*, and the worktree-per-issue isolation ([handoff rule #4](SKILL.md#critical-handoff-rules)) is what makes it **safe**.

**This doesn't contradict "one feature at a time"** (Anthropic, *Effective harnesses for long-running agents* — see [Further reading](#further-reading)). That rule bounds *what a single agent-iteration takes on* — don't one-shot a whole app and exhaust the context mid-feature — not *how many agents run at once*. Each worktree here is bound to one **issue** (a tracer-bullet slice, finer than the article's "feature"), so the bound holds whether one or several run. The violation to avoid is the inverse: one agent swallowing multiple issues to save worktrees, which rule #4 forbids.

**Parallelize across independent features, not slices of the same feature:**

| Across… | Verdict | Why |
|---|---|---|
| Independent features (different PRDs, no `Depends on:`, disjoint code) | Safe — the sweet spot | Small disjoint diffs, no dependency ordering, low merge-conflict surface. |
| Tracer-bullet slices of the *same* feature | Usually unsafe | Slices are deliberately *vertical* (schema → API → UI → tests, handoff rule #2), so two slices of one feature tend to touch the same schema/types/routing; a natural build order often `blocked-by`-serializes them anyway. |

So **Option 3's issue-level concurrency is safe *across* features but collapses to the unsafe row for two slices of the *same* feature** — Option 2's feature-level concurrency is the safe sweet spot by construction.

**The execution failure modes mutate under parallelism — they don't vanish:**

- *Leave a clean state* becomes *leave a clean state **and** a clean merge.* Each agent leaves its own worktree mergeable, but none sees the integration — concurrent agents both plan against `main`-as-it-is-now, not against each other's uncommitted work.
- *Verify the baseline first* assumes a **stable** `main`. Under parallelism the baseline moves as peers merge, so an agent can build against an already-stale snapshot and not know it.

**What the toolchain already makes parallel-safe** — leaving concurrent edits to shared *source* as the only real residual risk, which is git's to resolve, not the toolchain's:

- `DECISIONS.staged.md` is per-worktree and gitignored, promoted to `DECISIONS.md` serialized by teardown — conflict-free across worktrees (see [Decision journal](#decision-journal-decisionsmd)).
- [`/status`](references/status.md) aggregates open escalations across worktrees, so one human can supervise several concurrent AFK ships.
- Worktree + branch + planning-file isolation means no shared mutable state *during* a build.

**Guardrail:** parallelize only units with **no unresolved dependency and minimal file overlap**; keep same-feature vertical slices sequential.

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

All phases in `task_plan.md` ticked **and** the change merged into the default branch. For an **AFK** issue that merge is **autonomous** — the [adversarial review](#adversarial-review-before-close-out)'s clean pass triggers it, not a human at the merge button (the human spot-checks after); a **HITL** issue waits for a human. Either way the merge event is the durable signal — `task_plan.md` itself is ephemeral — and the tracker carries the persistent "closed/merged" status.

### Feature

Every issue produced by `/to-issues` on the feature's PRD has reached a **terminal** state — `shipped`, or closed-unshipped. A rejected or duplicate child is *resolved* and doesn't block the feature; a child still in-flight (`needs-info`, etc.) does. Workflow:

1. **Detect**: walk from the PRD to its child issues (via the parent reference each issue body carries) and confirm every child is [terminal](trackers/README.md#lifecycle-states) — none still in-flight.
   - **GitHub**: `gh issue list --search "parent:<PRD#> state:open"` returns empty.
   - **Local-markdown**: every file in `.scratch/<feature-slug>/issues/` has a terminal `Status:` ([lifecycle states](trackers/README.md#lifecycle-states)).
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

Long-running agent harnesses — the design lineage this suite operationalizes (incremental progress, files-as-handoff, clean-state-per-session, role separation):

- [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (Anthropic) — initializer + coding agent, a feature list, one-feature-at-a-time incremental progress, leave-it-clean-per-session — the source for the "one feature at a time" rule discussed in [Parallel execution](#parallel-execution).
- [Harness design for long-running application development](https://www.anthropic.com/engineering/harness-design-long-running-apps) (Anthropic) — a planner/generator/evaluator multi-agent architecture, context resets vs compaction with structured handoffs, and separating the builder from a skeptical evaluator — the lineage behind the [adversarial review](#adversarial-review-before-close-out) gate.
- [Simon Last — lessons on running coding agents at scale](https://x.com/simonlast/status/2057978156183957995) — field practice (fewer/longer/bigger-scoped sessions, role separation, self-contained plan docs over babysitting, adversarial review for unattended runs); the impetus for the [adversarial review](#adversarial-review-before-close-out) gate and [clean context per issue](#clean-context-per-issue).
