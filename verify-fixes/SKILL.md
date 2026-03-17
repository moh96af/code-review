---
name: code-review:verify-fixes
description: Use when checking if a PR author resolved your review comments, following up on a previously reviewed PR, or verifying comment resolution on Bitbucket pull requests. Triggers on "did he fix my comments", "check if resolved", "follow up on PR".
---

# Verify PR Review Fixes

Fetches your review comments from a Bitbucket PR, checks the latest code locally, and reports which comments were resolved.

## Prerequisites

Requires the [Bitbucket MCP server](#bitbucket-mcp-setup) to be configured.

## Trigger

`/code-review:verify-fixes <bitbucket-pr-url>`

---

## Workflow

### Step 1: Fetch Your Comments Only

Use Bitbucket MCP to get PR comments. Filter to only the user's comments (skip bots like CodeRabbit).

```
# Get PR metadata
bb_get /repositories/{workspace}/{repo}/pullrequests/{id}
  jq: {title, author: author.display_name, state, source_branch: source.branch.name}

# Get comment index (skip content to save tokens)
bb_get /repositories/{workspace}/{repo}/pullrequests/{id}/comments
  pagelen: 50
  jq: values[*].{id, author: user.display_name, file: inline.path, line: inline.to, resolved, parent_id: parent.id}

# Fetch full content of each user comment individually
bb_get /repositories/{workspace}/{repo}/pullrequests/{id}/comments/{comment_id}
  jq: {id, content: content.raw, file: inline.path, line: inline.to}
```

**Token optimization:** Page 1 often contains huge bot comments. Fetch the index first (no content), identify human comment IDs, then fetch each individually.

### Step 2: Check Code Locally

```bash
git fetch origin <source-branch>
git show origin/<source-branch>:<file-path>
```

For each comment, read the relevant file from the branch and check if the issue was addressed.

### Step 3: Report Resolution Table

Present a summary table:

| # | Comment | File | Status |
|---|---------|------|--------|
| 1 | Brief description | `file:line` | Fixed / Not fixed / Partial / Discussion |

- **Fixed** — Code change directly addresses the comment
- **Not fixed** — No change or issue remains
- **Partial** — Some aspects addressed, others remain
- **Discussion** — Author replied with reasoning, judgment call for reviewer

### Step 4: Handle Threaded Replies

When fetching comments, also check for author replies:

```
# Get replies to a comment (child comments)
bb_get /repositories/{workspace}/{repo}/pullrequests/{id}/comments
  jq: values[?parent.id == '{comment_id}'].{author: user.display_name, content: content.raw}
```

If the author replied with reasoning (not just "fixed"), include it in the report:

| # | Comment | File | Status | Author Reply |
|---|---------|------|--------|--------------|
| 3 | Use constants | `config.php:12` | Discussion | "We prefer inline values here because..." |

### Step 5: Resolve Comments (if supported)

```
# Resolve via API
bb_put /repositories/{workspace}/{repo}/pullrequests/{id}/comments/{comment_id}/resolve
```

**Known limitation:** Bitbucket's `/resolve` endpoint does not support token-based auth (returns 403). If this fails, inform the user they need to resolve manually in the browser.

---

## Key Behaviors

- **Only fetch user's comments** — skip CodeRabbit, bots, and PR description comments
- **Check code locally** — use `git show origin/<branch>:<file>` instead of fetching diffs via API (saves tokens)
- **Include author replies** — note when the author pushed back with reasoning
- **Report partial fixes** — distinguish between fully fixed, partially addressed, and untouched
- **Paginate carefully** — Bitbucket comments API paginates at 50; bot comments can be massive
