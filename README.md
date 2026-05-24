# swe-workflow

The idiomatic software-engineering workflow: **Idea → PRDs → Issues → ship.** A chain of small, observable steps connected by markdown files — not an opaque framework.

It ships as **[Agent Skills](https://agentskills.io)**, so it runs on **Claude Code, Codex, Gemini CLI, Cursor**, and any other skills-compatible agent. Pure markdown — no scripts, no build.

## The skill

`swe-workflow` is **one skill** that conducts the whole chain. Each stage is a procedure it runs on demand; on Claude Code each is also a slash command.

One companion skill ships alongside: **`log-decisions`** (the append-only `DECISIONS.md` journal). A typical run is **setup → spec → ship-all**.

## Install

### Claude Code

Installs the plugin — both skills (`swe-workflow`, `log-decisions`) plus the `/swe-workflow:*` slash commands.

```text
/plugin marketplace add swe-workflow/swe-workflow
/plugin install swe-workflow@swe-workflow
```

(`swe-workflow@swe-workflow` is `plugin@marketplace` — both happen to be named `swe-workflow`.)

### Other agents (universal)

Installs both skills — `swe-workflow` and `log-decisions`.

```text
npx skills add swe-workflow/swe-workflow
```

## Invoking

- **Claude Code** — the slash commands `/swe-workflow:setup` (0), `:spec` (1–4), `:to-features` (2), `:grill-feature` (3), `:ship` (5–7), `:ship-all` (5–7 ×N), `:status` (thin shims over the skill), or invoke the `swe-workflow` skill to drive the whole chain (0–7).
- **Codex · Gemini CLI · Cursor · others** — invoke the `swe-workflow` skill and say what you want (*"ship issue #42"*); it routes to the right stage. Most hosts also surface it as a slash command or trigger it from its description.

### Prerequisites

The workflow orchestrates several skills this suite doesn't bundle. **The setup stage (`/swe-workflow:setup`) auto-installs the missing ones** (then restart to activate them); or install manually:

- [`planning-with-files`](https://github.com/OthmanAdi/planning-with-files) — the `plan` / `plan-goal` / `status` engine the ship stage runs for each issue. Cross-agent.
- [`andrej-karpathy-skills:karpathy-guidelines`](https://github.com/multica-ai/andrej-karpathy-skills) — code-quality guardrails (surgical, simple changes).
- [`mattpocock/skills`](https://github.com/mattpocock/skills) — `tdd` (the red → green → refactor inner loop) plus the upstream spec layer: `setup-matt-pocock-skills`, `grill-with-docs`, `to-prd`, `to-issues`, and `triage`. Cross-agent.

## How it fits together

```text
   IDEA
   │
   ▼   /setup (0)  ──►  bootstrap repo — prereq skills · always-on rules
   │
   ▼   SPEC LAYER  ·  /spec  (stages 1–4, AFK-friendly)
   │      grill (1)          ──►  CONTEXT.md + ADRs
   │      to-features (2)    ──►  FEATURES.md
   │      grill-feature (3)  ──►  a PRD  (one per feature)
   │      to-issues (4)      ──►  ready-for-agent issues
   │
   ▼   EXECUTION LAYER  ·  /ship (one issue) · /ship-all (backlog, AFK)
   │      plan (5)           ──►  task_plan.md  (+ findings.md)
   │      build (6)          ──►  progress.md  (test-first, via /tdd)
   │      close-out (7)      ──►  PR  +  worktree teardown
   │
   ▼
   SHIPPED

   /triage runs alongside — sorts externally-filed issues into the ready-for-agent backlog
```

Those right-hand artifacts are the interface between steps — the files, not the agent's memory, carry state from stage to stage. See the **`swe-workflow`** skill (`skills/swe-workflow/SKILL.md` + `REFERENCE.md`, each stage's procedure in `references/`) for the full picture, including the detailed operational diagram.

## A typical run

Spelled out for a fresh project — what to run at each stage, and the artifact it leaves behind:

1. `/swe-workflow:setup` *(stage 0)* — run once at the repo root; bootstraps the repo and auto-installs the external skills the later steps need.
2. `/grill-with-docs` *(stage 1)* — grill the domain → `CONTEXT.md` + `docs/adr/`.
3. `/swe-workflow:to-features` *(stage 2)* — split the project into coarse features → `FEATURES.md`.
4. `/swe-workflow:grill-feature` *(stage 3)* — once **per feature** → one PRD each.
5. `/to-issues` *(stage 4)* — once **per PRD** → tracer-bullet issues, auto-labeled `ready-for-agent`.
6. `/swe-workflow:ship-all` *(stages 5–7)* — build and ship the whole backlog, AFK.

`/grill-with-docs` and `/to-issues` are external (mattpocock) skills, while `/swe-workflow:to-features` and `:grill-feature` are commands in this suite — and steps 2–5 are exactly what `/swe-workflow:spec` runs for you. Drive the stages by hand, or run `/swe-workflow:spec` then `/swe-workflow:ship-all` for the hands-off path.

## License

MIT
