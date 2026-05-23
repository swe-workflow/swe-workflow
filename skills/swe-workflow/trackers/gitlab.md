# Tracker: GitLab Issues

Satisfies the [tracker contract](README.md) — GitLab-specific bits only.

**CLI**: [`glab`](https://gitlab.com/gitlab-org/cli) — `glab auth login` once.
**ID format**: integer (e.g., `123`).

## Fetch one

```bash
glab issue view <id> --output json
```

Field deltas from the [normalized issue](README.md#normalized-issue): `body` = `.description` (**not** `.body`); `labels` = `.labels` (already a flat string array). AGENT-BRIEF lives in a **note** (GitLab's term for a comment) — fetch separately with `glab issue note list <id>` and filter for `## Agent Brief`.

## List ready

```bash
glab issue list --label <ready-for-agent> --output json
```

## Auto-detect signal

`glab` in `PATH` **AND** `git config --get remote.origin.url` contains `gitlab.com`.
