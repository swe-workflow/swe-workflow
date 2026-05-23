# Tracker: Linear

Satisfies the [tracker contract](README.md) — Linear-specific bits only.

**CLI**: [`linear`](https://linear.app/changelog) (official) or community alternatives.
**Auth**: API key via `linear auth login` (or env var per the CLI version).
**ID format**: `TEAM-123` (e.g., `ENG-42`) — **string, not integer**. Passed through literally for branch/worktree names; do not strip the team prefix or convert to a number.

## Fetch one

```bash
linear issue view TEAM-123 --json
```

Field deltas from the [normalized issue](README.md#normalized-issue): `body` = `.description`; `labels` = `[.labels[].name]`. AGENT-BRIEF: standard (`## Agent Brief` comment in Linear's comment thread).

## List ready

```bash
linear issue list --label <ready-for-agent> --json
```

## Auto-detect signal

A `.linear/` directory at the repo root, OR `~/.config/linear/` exists and the project has a Linear team configured.
