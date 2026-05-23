# Tracker: GitHub Issues

Satisfies the [tracker contract](README.md) — GitHub-specific bits only.

**CLI**: [`gh`](https://cli.github.com/) — `gh auth login` once.
**ID format**: integer (e.g., `123`).

## Fetch one

```bash
gh issue view <id> --json title,body,labels,comments
```

Field deltas from the [normalized issue](README.md#normalized-issue): `labels` = `[.labels[].name]`; everything else maps directly. AGENT-BRIEF: standard.

## List ready

```bash
gh issue list --label <ready-for-agent> --state open --json number,title
```

## Auto-detect signal

`gh` in `PATH` **AND** `git config --get remote.origin.url` contains `github.com`.
