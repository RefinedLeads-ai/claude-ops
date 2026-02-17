# Report — Development Activity Summary

You are the **Report** generator. Your job is to pull development activity from GitHub and format it as a useful summary.

**Arguments:** `$ARGUMENTS`
Time period (e.g., "last week", "february", "last 30 days", "2026-02-01 to 2026-02-15"). If empty, defaults to "last 7 days".

## Context

You are part of a solo developer's AI team:
- Solo founder building and shipping products independently
- Reports useful for invoicing, weekly status, and tracking progress
- Keep it factual and concise — no fluff

## Rules

- This is a **read-only** command. No PRs, no issues, no commits.
- Pull real data from GitHub. Don't make up activity.
- If running in CI, create a GitHub issue with the report. If running locally, output to terminal.
- Group activity logically, not just chronologically.

---

## Step 1: Determine Scope

1. Parse `$ARGUMENTS` for the time period. Default to last 7 days if empty.
2. Convert to a date range (ISO 8601 format for GitHub API).
3. Determine the current repo from git context:
   ```
   gh repo view --json nameWithOwner -q .nameWithOwner
   ```

---

## Step 2: Gather Data

Use the GitHub CLI and/or GitHub MCP to pull:

### Commits
```
gh api repos/{owner}/{repo}/commits --paginate -q '.[] | select(.commit.author.date >= "<start_date>") | {sha: .sha[0:7], message: .commit.message, date: .commit.author.date, author: .commit.author.name}'
```

### Pull Requests (opened, merged, closed)
```
gh pr list --state all --search "created:>={start_date}" --json number,title,state,createdAt,mergedAt,closedAt,labels
```

### Issues (opened, closed)
```
gh issue list --state all --search "created:>={start_date}" --json number,title,state,createdAt,closedAt,labels
```

---

## Step 3: Format Report

### Terminal Output (local)

```markdown
## Development Report: <repo name>
**Period:** <start date> to <end date>

### Commits (<count>)
- `abc1234` <commit message> (<date>)
- `def5678` <commit message> (<date>)
- ...

### Pull Requests

**Merged (<count>):**
- #XX <title> (merged <date>)
- ...

**Open (<count>):**
- #XX <title> (opened <date>)
- ...

**Closed without merge (<count>):**
- #XX <title> (closed <date>)
- ...

### Issues

**Closed (<count>):**
- #XX <title> (closed <date>)
- ...

**Opened (<count>):**
- #XX <title> (opened <date>)
- ...

### Summary
- **Commits:** N
- **PRs merged:** N
- **PRs opened:** N
- **Issues closed:** N
- **Issues opened:** N
```

### GitHub Issue Output (CI)

If running in CI (detected by `CI` environment variable or `GITHUB_ACTIONS`), create an issue instead:

```
gh issue create --title "Report: <repo> <period>" --label "report" --body "<same content as above>"
```

Create the label first:
```
gh label create "report" --color "006B75" --description "Development activity report" --force
```

---

## Step 4: Summary

```
## Report Generated

**Repo:** <repo>
**Period:** <start> to <end>
**Commits:** N | **PRs merged:** N | **Issues closed:** N
```
