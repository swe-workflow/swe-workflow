# Tracker: Local Markdown Files

**CLI**: none — issues are markdown files on disk.
**Convention**: mattpocock's `.scratch/<feature-slug>/` layout. mattpocock's [`issue-tracker-local.md`](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/issue-tracker-local.md) is **authoritative** for the directory layout, issue file format, and triage `Status:` vocabulary; this doc covers only what swe-workflow's bootstrap adds on top — field extraction, ID handling, and branch/worktree naming.
**ID format**: full path to the issue file, OR `<feature-slug>/<NN>` when context is unambiguous.

## Layout (in brief)

`.scratch/<feature-slug>/PRD.md` + `.scratch/<feature-slug>/issues/<NN>-<slug>.md` — one feature directory per PRD, issues numbered from `01`. Grouping by feature preserves the `/to-prd` → `/to-issues` parent/child link in the filesystem (`ls .scratch/` is the backlog view). Full tree and issue file format live in the [authoritative doc](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/issue-tracker-local.md).

## Extracting fields

| Normalized field | How to extract |
|---|---|
| title | First H1 (`# …`) in the file |
| body | Everything between the H1 and `## Comments` (exclusive), minus the `Status:` line |
| labels | The value on the `Status:` line (single label by convention; the role string) |
| agent brief | The body itself IS the brief (the whole file is one well-structured issue). Return as `[{"body": "<entire body>"}]`. |

If a file carries additional label-like lines (e.g. `Type:`, `Priority:`), extract them too — but `Status:` is the only one mattpocock's chain reads. Its values use mattpocock's canonical triage vocabulary (see [`triage-labels.md`](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/triage-labels.md)).

## ID handling

The user typically passes either:

- A **full path** to the issue file: `.scratch/auth/issues/01-add-login.md`
- The **`<feature>/<NN>`** form: `auth/01` (when unambiguous)

The bootstrap should accept both forms; resolve to a real path before reading.

Branch and worktree naming:

- Branch: `issue-<feature-slug>-<NN>-<slug>` (e.g., `issue-auth-01-add-login`)
- Worktree: `../<repo>-issue-<feature-slug>-<NN>/`

## Auto-detect signal

A `.scratch/` directory exists at the repo root.

## See also

- [setup-matt-pocock-skills/issue-tracker-local.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/issue-tracker-local.md) — authoritative layout, file format, and conventions
- [setup-matt-pocock-skills/triage-labels.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/triage-labels.md) — canonical triage role vocabulary
