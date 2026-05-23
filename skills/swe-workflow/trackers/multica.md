# Tracker: Multica

Satisfies the [tracker contract](README.md) — Multica-specific bits only.

**CLI**: `multica` — see the `multica-cli` skill; auth via `multica login`.
**Config**: user-level (`~/.multica/config.json`), not project-level.
**ID format**: dual — UUID (`bb3166b2-…`) and human `identifier` (`ARC-47`). Use `.identifier` for branch/worktree names, read off the fetch response regardless of which form the user typed.

## Fetch one

Details and comments are **separate calls**:

```bash
multica issue get <id-or-uuid> --output json           # defaults to JSON
multica issue comment list <id-or-uuid> --output json  # table by default — force JSON
```

Accepts UUID, hex prefix (≥4 chars), or `ARC-47` identifier (best-effort; falls back to UUID if rejected).

Field deltas from the [normalized issue](README.md#normalized-issue): `body` = `.description`; `labels` = `[.labels[].name]` (label objects → `.name`). AGENT-BRIEF: try the comment list for `## Agent Brief`; if none, the structured `.description` IS the brief (the contract's fallback).

**Triage state**: Multica carries `.status`/`.priority` as first-class fields, *not* labels. mattpocock's triage roles appear in `.labels[].name` only if the team adopted that convention on top — trust `.labels` for triage state, `.status` for tracker-native state.

## List ready

```bash
multica issue list --label <ready-for-agent> --output json
```

## Auto-detect signal

`$MULTICA_WORKSPACE_ID` is set. (No project-level signal — config is user-level; set `MULTICA_WORKSPACE_ID` in shell init for Multica-heavy workspaces.) Workspace is selected via `$MULTICA_WORKSPACE_ID` / `--workspace-id` / `multica config set` — the bootstrap inherits whatever is active.
