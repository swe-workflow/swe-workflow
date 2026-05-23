# Tracker: Local Markdown Files

Satisfies the [tracker contract](README.md). File-based, so its deltas are larger than the CLI trackers.

**CLI**: none — issues are markdown files on disk.
**Convention**: mattpocock's `.scratch/<feature-slug>/` layout. mattpocock's [`issue-tracker-local.md`](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/issue-tracker-local.md) is **authoritative** for the directory layout, issue file format, and triage `Status:` vocabulary.
**ID format**: full path to the issue file, OR `<feature-slug>/<NN>` when unambiguous (e.g. `auth/01`). Accept both; resolve to a real path before reading.

## Layout (in brief)

`.scratch/<feature-slug>/PRD.md` + `.scratch/<feature-slug>/issues/<NN>-<slug>.md` — one feature directory per PRD, issues numbered from `01`. Full tree and file format live in the authoritative doc above.

## Fetch one

Read the file and map to the [normalized issue](README.md#normalized-issue):

| Field | From the file |
|---|---|
| title | first H1 (`# …`) |
| body | between the H1 and `## Comments` (exclusive), minus the `Status:` line |
| labels | the `Status:` line value (the triage role string) |
| agent-brief | the body IS the brief (the whole file is one structured issue) |

`Status:` values use mattpocock's canonical triage vocabulary (see [`triage-labels.md`](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/triage-labels.md)). Extra label-like lines (`Type:`, `Priority:`) may be extracted too, but `Status:` is the only one the chain reads.

## List ready

Glob the issue files and filter on the `Status:` line:

```bash
grep -lE '^Status:[[:space:]]*ready-for-agent' .scratch/<feature-slug>/issues/*.md
```

(Omit the feature path to scan all features.)

## Branch & worktree naming

The `<id>` is the compound `<feature-slug>-<NN>`, so per the contract: branch `issue-<feature-slug>-<NN>-<slug>` (e.g. `issue-auth-01-add-login`); worktree `../<repo>-issue-<feature-slug>-<NN>/`.

## Auto-detect signal

A `.scratch/` directory exists at the repo root.
