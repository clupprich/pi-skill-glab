# pi-skill-glab

A [pi](https://github.com/badlogic/pi) skill for interacting with GitLab via the [glab](https://gitlab.com/gitlab-org/cli) CLI.

Covers merge requests, issues, CI/CD pipelines, repository management, and draft review notes via the GitLab API.

## Prerequisites

- `glab` must be installed and authenticated (`glab auth status` to check)
- Must be in a git repository with a GitLab remote

## Install

```bash
pi install npm:pi-skill-glab
```

## What's Included

The skill teaches the agent how to:

- **Merge Requests** — list, view, create, update, approve, merge, rebase, comment
- **Issues** — list, view, create, update, close, reopen, comment
- **CI/CD Pipelines** — status, list, trace logs, retry jobs, trigger, lint config, download artifacts
- **Repository** — view info, list, search, clone, contributors
- **Draft Review Notes** — create positioned review comments via `glab api`, list, delete, and bulk publish
- **Raw API** — fallback for anything not covered by specific commands

## License

MIT
