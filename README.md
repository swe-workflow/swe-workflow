# swe-workflow

The idiomatic software-engineering workflow: **idea → PRD → issues → ship.** A chain of small, observable steps connected by markdown files — not an opaque framework.

It ships as **[Agent Skills](https://agentskills.io)**, so it runs on **Claude Code, Codex, Gemini CLI, Cursor**, and any other skills-compatible agent. Pure markdown — no scripts, no build.

## The skill

`swe-workflow` is **one skill** that conducts the whole chain. Each stage is a procedure it runs on demand; on Claude Code each is also a slash command.

| Stage(s) | What it does | Claude command |
|---|---|---|
| 0 | **Run first.** Bootstrap a repo — auto-install missing prerequisite skills, wrap stage-0, inject the always-on rules (decision-logging + engineering discipline), set the `DECISIONS.md` privacy policy. Idempotent. | `/swe-workflow:setup` |
| 1–4 | Take ONE feature through **grill → features → PRD → issues** — leaves a `ready-for-agent` backlog. Interactive, idempotent. | `/swe-workflow:spec` |
| 5–7 | Take ONE issue through **plan → build → close-out** — worktree + planning files, test-first build, PR, teardown. | `/swe-workflow:ship` |
| 5–7 ×N | Run **ship** across a backlog in dependency order — each **AFK** issue, **pausing at HITL**. | `/swe-workflow:ship-all` |
| — | Show planning status for the current issue, or roll up all in-flight issues + open escalations. | `/swe-workflow:status` |
| 0–7 | Invoke the skill itself (*"plan this feature end-to-end"*) — it conducts the whole chain. | — |

Two companion skills ship alongside: **`to-features`** (stage 2 — enumerate features into `FEATURES.md`) and **`log-decisions`** (the append-only `DECISIONS.md` journal). A typical run is **setup → spec → ship-all**.

## Install

**Any Agent-Skills agent (universal):**

```text
npx skills add swe-workflow/swe-workflow
```

Installs the three skills — `swe-workflow`, `to-features`, `log-decisions`.

**Claude Code (plugin — also adds the `/swe-workflow:*` slash commands):**

```text
/plugin marketplace add swe-workflow/swe-workflow
/plugin install swe-workflow@swe-workflow
```

(`swe-workflow@swe-workflow` is `plugin@marketplace` — both happen to be named `swe-workflow`.)

## Invoking

- **Claude Code** — the slash commands `/swe-workflow:setup`, `:spec`, `:ship`, `:ship-all`, `:status` (thin shims over the skill), or invoke the `swe-workflow` skill to drive the whole chain.
- **Codex · Gemini CLI · Cursor · others** — invoke the `swe-workflow` skill and say what you want (*"ship issue #42"*); it routes to the right stage. Most hosts also surface it as a slash command or trigger it from its description.

### Prerequisites

The workflow orchestrates several skills this suite doesn't bundle. **The setup stage (`/swe-workflow:setup`) auto-installs the missing ones** (then restart to activate them); or install manually:

- [`planning-with-files`](https://github.com/OthmanAdi/planning-with-files) — the `plan` / `plan-goal` / `status` engine the ship stage runs for each issue. Cross-agent.
- [`andrej-karpathy-skills:karpathy-guidelines`](https://github.com/multica-ai/andrej-karpathy-skills) — code-quality guardrails (surgical, simple changes).
- [`mattpocock/skills`](https://github.com/mattpocock/skills) — `tdd` (the red → green → refactor inner loop) plus the upstream spec layer: `setup-matt-pocock-skills`, `grill-with-docs`, `to-prd`, `to-issues`, and `triage`. Cross-agent.

## How it fits together

```text
spec layer — stages 1–4                        execution layer — stages 5–7
grill → to-features → to-prd → to-issues  ──►  one issue: plan → build → close-out
(triage runs alongside)                        ship-all — the whole backlog, AFK
```

Every step hands the next a markdown artifact (`CONTEXT.md`, `FEATURES.md`, a PRD, issues, then `task_plan.md` and `progress.md`). The files are the interface; nothing lives only in the agent's head. See the **`swe-workflow`** skill (`skills/swe-workflow/SKILL.md` + `REFERENCE.md`, with each stage's procedure in `references/`) for the full picture.

## License

MIT
