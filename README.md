# swe-workflow

The idiomatic software-engineering workflow: **Idea → PRDs → Issues → ship.** A chain of small, observable steps connected by markdown files — not an opaque framework.

It ships as **[Agent Skills](https://agentskills.io)**, so it runs on **Claude Code, Codex, Gemini CLI, Cursor**, and any other skills-compatible agent. Pure markdown — no scripts, no build.

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
- [`mattpocock/skills`](https://github.com/mattpocock/skills) — `tdd` (the red → green → refactor inner loop) plus the upstream spec layer: `setup-matt-pocock-skills`, `grill-with-docs`, `to-spec`, `to-tickets`, and `triage`. Cross-agent.

## How it fits together

```text
   IDEA
   │
   ▼   /setup (0)  ──►  bootstrap repo — prereq skills · always-on rules
   │
   ▼   SPEC LAYER  ·  /spec  (stages 1–4, AFK-friendly)
   │      grill the domain (1)    ──►  CONTEXT.md + ADRs
   │      enumerate features (2)  ──►  FEATURES.md
   │      spec one feature (3)    ──►  a PRD  (one per feature)
   │      slice into issues (4)   ──►  ready-for-agent issues
   │
   ▼   EXECUTION LAYER  ·  /ship (one issue) · /ship-all (backlog, AFK)
   │      plan (5)       ──►  task_plan.md  (+ findings.md)
   │      build (6)      ──►  progress.md  (test-first)
   │      close-out (7)  ──►  PR  +  worktree teardown
   │
   ▼
   SHIPPED

   triage runs alongside — sorts externally-filed issues into the ready-for-agent backlog
```

Those right-hand artifacts are the interface between stages — the files, not the agent's memory, carry state from stage to stage. The **`swe-workflow`** skill's stage table (`skills/swe-workflow/SKILL.md`) is the canonical map, with each stage's full procedure in `references/`; the design rationale lives in [docs/DESIGN.md](docs/DESIGN.md).

## A typical run

Spelled out for a fresh project — three commands, and what each leaves behind:

1. `/swe-workflow:setup` *(stage 0)* — run once at the repo root; bootstraps the repo and auto-installs the external skills the later stages need.
2. `/swe-workflow:spec` *(stages 1–4)* — the spec layer, AFK-friendly: grill the domain → coarse features → a PRD per feature → tracer-bullet issues. Leaves `CONTEXT.md` + ADRs, `FEATURES.md`, PRDs, and a backlog of `ready-for-agent` issues.
3. `/swe-workflow:ship-all` *(stages 5–7)* — build and ship the whole backlog, AFK.

Prefer to drive the spec layer stage by stage? Every stage's procedure and command is listed in the skill's stage table (`skills/swe-workflow/SKILL.md`).

## License

MIT
