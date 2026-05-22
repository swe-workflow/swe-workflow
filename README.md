# swe-workflow

The idiomatic software-engineering workflow: **idea → PRD → issues → ship.** A chain of small, observable steps connected by markdown files — not an opaque framework.

## Commands

| Command | What it does |
|---|---|
| `/swe-workflow:ship <issue-slug>` | Take ONE issue through **plan → build → close-out** (stages 5–7): worktree + planning files, a test-first build, PR, and teardown. The arg is an issue slug — a local-markdown path like `.scratch/feature-x/issues/02-foo.md`, or a tracker id (GitHub `#`, Linear key). |
| `/swe-workflow:ship-all [scope]` | Run `/ship` across a backlog: walk issues in dependency order, ship each **AFK** issue, and **pause at HITL** issues. Optional scope = a feature dir; default = all `ready-for-agent` issues. |
| `/swe-workflow:to-features` | Read `CONTEXT.md` + ADRs and enumerate user-facing features into `FEATURES.md`. |
| `/swe-workflow:status` | Show planning status for the current issue (phase, progress, decisions, errors). |

The full workflow — stages 0–7, the parallel `/triage` concern, and the design philosophy — lives in the bundled **swe-workflow** skill: `skills/swe-workflow/SKILL.md` and `skills/swe-workflow/REFERENCE.md`.

## Install

```text
/plugin marketplace add swe-workflow/swe-workflow
/plugin install swe-workflow@swe-workflow
```

(`swe-workflow@swe-workflow` is `plugin@marketplace` — both happen to be named `swe-workflow`.)

### Prerequisites

The workflow relies on several skills this plugin doesn't bundle — install the ones you need:

- [`planning-with-files`](https://github.com/OthmanAdi/planning-with-files) — the `plan` / `plan-goal` / `status` engine `/ship` runs for each issue.
- [`andrej-karpathy-skills:karpathy-guidelines`](https://github.com/multica-ai/andrej-karpathy-skills) — code-quality guardrails (surgical, simple changes).
- [`mattpocock/skills`](https://github.com/mattpocock/skills) — `tdd` (the red → green → refactor inner loop) plus the upstream spec layer: `setup-matt-pocock-skills`, `grill-with-docs`, `to-prd`, `to-issues`, and `triage`.

## How it fits together

```text
spec layer (external skills)         execution layer (this plugin)
grill → to-features → to-prd  ──►   /ship  one issue: plan → build → close-out
       → to-issues → triage          /ship-all  the whole backlog, AFK
```

Every step hands the next a markdown artifact (`CONTEXT.md`, `FEATURES.md`, a PRD, issues, then `task_plan.md` and `progress.md`). The files are the interface; nothing lives only in the agent's head. See the bundled skill for the full picture.

## License

MIT
