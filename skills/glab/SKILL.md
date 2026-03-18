---
name: glab
description: Interact with GitLab via the glab CLI. Use for merge requests (create, list, view, review, merge), issues (create, list, view, comment), CI/CD pipelines (status, view logs, retry), and repository management. Requires glab to be installed and authenticated.
---

# GitLab CLI (glab)

Interact with GitLab repositories, merge requests, issues, and CI/CD pipelines using the `glab` CLI.

## Prerequisites

- `glab` must be installed and authenticated (`glab auth status` to check)
- Must be in a git repository with a GitLab remote

## Merge Requests

```bash
# List open MRs
glab mr list

# List MRs with filters
glab mr list --assignee=@me
glab mr list --label=bug --state=merged
glab mr list --search="keyword"

# View MR details
glab mr view <id>

# View MR diff
glab mr diff <id>

# Create MR from current branch
glab mr create --fill                          # Auto-fill title/description from commits
glab mr create --title "Title" --description "Description"
glab mr create --target-branch main --label bug --assignee @me

# Create MR for an issue (creates branch + MR)
glab mr for <issue-id>

# Update MR
glab mr update <id> --title "New title"
glab mr update <id> --label add:bug --label remove:wip
glab mr update <id> --assignee @username

# Comment on MR (posted immediately, not as a review)
glab mr note <id> -m "Comment text"

# Approve / merge / rebase
glab mr approve <id>
glab mr merge <id>
glab mr merge <id> --squash
glab mr rebase <id>

# Checkout MR locally
glab mr checkout <id>
```

## Issues

```bash
# List issues
glab issue list
glab issue list --assignee=@me
glab issue list --label=bug --state=opened
glab issue list --search="keyword"

# View issue details
glab issue view <id>

# Create issue
glab issue create --title "Title" --description "Description"
glab issue create --title "Bug" --label bug --assignee @me

# Comment on issue
glab issue note <id> -m "Comment text"

# Update issue
glab issue update <id> --title "New title"
glab issue update <id> --label add:priority

# Close / reopen
glab issue close <id>
glab issue reopen <id>
```

## CI/CD Pipelines

```bash
# View current pipeline status
glab ci status

# List pipelines
glab ci list

# View pipeline interactively (shows jobs + status)
glab ci view

# Get pipeline details as JSON
glab ci get --branch main

# Trace job logs in real time
glab ci trace <job-id>

# Retry a failed job
glab ci retry <job-id>

# Trigger a manual job
glab ci trigger <job-id>

# Run a new pipeline
glab ci run --branch main
glab ci run --variables KEY1:value1

# Lint CI config
glab ci lint

# Download artifacts
glab ci artifact <ref> <job-name>
```

## Repository

```bash
# View repo info
glab repo view

# List repos
glab repo list

# Search repos
glab repo search --search "query"

# Clone
glab repo clone owner/repo

# View contributors
glab repo contributors
```

## Draft Review Notes (Review Comments)

`glab` has no built-in command for draft/review notes. Use `glab mr note` for immediate
comments, but for **review comments** (draft notes batched and submitted together, optionally
positioned on specific diff lines), you must use the GitLab API directly via `glab api`.

### Setup: Get project ID and diff SHAs

```bash
# Get project ID
PROJECT_ID=$(glab api projects/:id | jq '.id')

# Get diff SHAs for the MR (needed for positioning comments on diff lines)
glab api "projects/$PROJECT_ID/merge_requests/<mr_id>/versions" | jq '.[0] | {head_commit_sha, base_commit_sha, start_commit_sha}'
```

### Create a draft note on a specific diff line

Use `glab api` with `--input` and a JSON body. The `-f` flag does NOT work for nested
`position` fields — you must pass JSON via `--input`.

```bash
glab api "projects/$PROJECT_ID/merge_requests/<mr_id>/draft_notes" -X POST \
  -H "Content-Type: application/json" \
  --input <(echo '{
    "note": "Your review comment here.",
    "position": {
      "position_type": "text",
      "base_sha": "<base_sha>",
      "start_sha": "<start_sha>",
      "head_sha": "<head_sha>",
      "new_path": "path/to/file.ext",
      "old_path": "path/to/file.ext",
      "new_line": 42
    }
  }')
```

### Create a draft note without a diff position (general review comment)

```bash
glab api "projects/$PROJECT_ID/merge_requests/<mr_id>/draft_notes" -X POST \
  -H "Content-Type: application/json" \
  --input <(echo '{"note": "General review comment."}')
```

### List, delete, and submit draft notes

```bash
# List all draft notes
glab api "projects/$PROJECT_ID/merge_requests/<mr_id>/draft_notes" | jq '.[] | {id, note}'

# Delete a draft note
glab api "projects/$PROJECT_ID/merge_requests/<mr_id>/draft_notes/<note_id>" -X DELETE

# Submit/publish all draft notes as a review
glab api "projects/$PROJECT_ID/merge_requests/<mr_id>/draft_notes/bulk_publish" -X POST
```

## API (Advanced)

For anything not covered by specific commands, use the raw API:

```bash
# GET request
glab api projects/:id/merge_requests

# POST request
glab api projects/:id/issues -X POST -f title="Bug" -f description="Details"

# With pagination
glab api projects/:id/issues --paginate
```

## Tips

- Use `-R owner/repo` on any command to target a different repository
- Use `--output json` or pipe through `jq` for structured output where supported
- When creating MRs, `--fill` populates title and description from commit messages
- Use `glab mr for <issue-id>` to create a branch and MR linked to an issue in one step
