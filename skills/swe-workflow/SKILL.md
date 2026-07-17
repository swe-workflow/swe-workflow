---
name: swe-workflow
description: The idiomatic idea → PRD → issues → shipped-PR workflow. Use to plan a feature end-to-end, turn a fuzzy idea into a PRD, or ship a ready-for-agent issue.
---

# SWE Workflow

The idiomatic software-engineer workflow — clarify the idea → spec it → slice it → ship it — as a chain of stages connected by durable markdown artifacts. **The files are the interface**: every stage hands the next an artifact (`CONTEXT.md`/ADRs → `FEATURES.md` → PRD → issues → `task_plan.md`/`progress.md`); nothing lives only in the agent's head.

One [Agent Skill](https://agentskills.io) — portable across Claude Code, Codex, Gemini CLI, Cursor, and other skills-compatible agents. Invoked with no specific stage, it **conducts** the whole chain ([Driving the chain](#driving-the-chain)); invoked at a point (*"ship issue 42"*), it routes to that stage's procedure. On Claude Code each procedure is also a `/swe-workflow:*` command (thin shims over the same files); on other agents, invoke the skill and say what you want.

## The chain

| # | Stage — the question it answers | Artifact out | Procedure (command) | Enter here when |
|---|---|---|---|---|
| 0 | Setup — how is this repo set up? | rules block in `AGENTS.md`/`CLAUDE.md`, `docs/agents/`, `.swe-workflow.conf` | [setup.md](references/setup.md) (`:setup`) | repo not bootstrapped yet |
| 1 | Grill the domain — what do I want? | `CONTEXT.md`, ADRs | [spec.md](references/spec.md) (`:spec` drives 1–4) | fuzzy terms, no glossary yet |
| 2 | Enumerate features — what does this break into? | `FEATURES.md` | [to-features.md](references/to-features.md) (`:to-features`) | domain clear, features not listed |
| 3 | Spec one feature — what does done look like? | one PRD per feature | [grill-feature.md](references/grill-feature.md) (`:grill-feature`) | feature picked, no PRD for it |
| 4 | Slice — what are the units of work? | `ready-for-agent` issues | [spec.md](references/spec.md), stage 4 | PRD exists but is one mega-issue |
| 5–7 | Ship — plan, build, close out | merged PR/branch + teardown | [ship.md](references/ship.md) (`:ship`); ×N [ship-all.md](references/ship-all.md) (`:ship-all`) | issue picked (5) · plan ready (6) · built (7) |
| ∥ | Triage — what's actionable? (parallel, not in the chain) | triage labels on external issues | the external `triage` skill | an outside-filed issue needs classification |
| — | Status — where is everything? | report only, read-only | [status.md](references/status.md) (`:status`) | returning to in-flight work |

Stages 1–4 are the **spec layer** (AFK-friendly interviews; leaves a `ready-for-agent` backlog); 5–7 the **execution layer** (worktree + planning files per issue; builds it). Chain-created issues are auto-labeled `ready-for-agent`, so `triage` exists only for issues filed *outside* the chain (bug reports, external contributions).

## Driving the chain

Invoked end-to-end (*"plan this feature end-to-end," "how do I start on this idea?"*), run three idempotent blocks in order — each skips whatever's already done:

1. **Setup** (stage 0) — only if the repo isn't bootstrapped; it no-ops otherwise.
2. **Spec** (stages 1–4) — grill → features → PRD → issues; resumes from whatever specs already exist.
3. **Ship-all** (stages 5–7) — build and ship the backlog.

Both layers record decisions via the `log-decisions` skill and proceed on recommended answers when you're away. The difference is what happens on an **unsure HITL call** (one only you can make): **spec pauses** to ask; **ship parks it** for batch review (via [status](references/status.md)) and keeps shipping independent issues, so the AFK batch never blocks (a pre-declared HITL issue is the exception — ship won't attempt it unattended). Re-invoking anything is safe: every command self-detects state and resumes where the chain left off.

How to map the chain onto CLI sessions — one long session, one per feature, one per issue — is the operator's call; see [Session topology](https://github.com/swe-workflow/swe-workflow/blob/main/docs/DESIGN.md#session-topology).

## Invariants

1. **Files are the interface** — every handoff is a markdown artifact, so any stage can re-read where things stand and resume. That's what makes every command idempotent: re-runs no-op finished steps, never duplicate or clobber.
2. **Issues are tracer bullets** — thin vertical slices through every layer (schema → API → UI → tests). "Backend issue" + "frontend issue" is a smell — re-slice.
3. **Only `ready-for-agent` issues enter execution** — the label comes from the chain (auto-applied) or from `triage` (external issues); stage 5 reads the label, not the source.
4. **One issue = one worktree = one `task_plan.md`** — filesystem isolation, no exceptions ([ship.md](references/ship.md)).
5. **Strike through, don't delete** — a shipped feature stays in `FEATURES.md`, struck through with shipped refs ([to-features.md](references/to-features.md)).
6. **Security boundary** — `task_plan.md` is re-injected into context on every tool call, so it gets only structured fields the executor wrote; raw external text (issue bodies, fetched docs) goes to `findings.md` ([ship.md](references/ship.md)).
7. **Don't double-track** — one home per fact: architecture decisions in the PRD, the raw brief in `findings.md`, execution-time decisions in `task_plan.md`, the session narrative in `progress.md` (which IS the PR body — don't rewrite it).
8. **Ephemeral by design** — worktrees and planning files die at merge; only the tracker, `CONTEXT.md`/ADRs, and `FEATURES.md` survive across features.
9. **PRD language = stage-1 glossary** — a PRD that conflicts with `CONTEXT.md` loops back to the domain grill.
10. **Done is layered** — phase = `task_plan.md` checkbox; issue = merged + terminal in the tracker; feature = every child terminal → strike; project = your judgment call ([status.md](references/status.md#levels-of-done)).

## When to skip

- Single-file edits — no spec, no plan needed.
- Bug fixes whose brief is one paragraph — just do it, skip the stage-5 bootstrap ([ship.md](references/ship.md)).
- Exploration / prototypes — use a prototyping skill instead.

---

A chain of small skills, not a framework: own the process, keep every artifact observable, keep per-issue state ephemeral, stay instructions-only (no scripts). The rationale, session topology, and spec-kit comparison live in [docs/DESIGN.md](https://github.com/swe-workflow/swe-workflow/blob/main/docs/DESIGN.md).
