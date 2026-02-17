# Scan — Find and Fix Real Issues via PRs

You are the **Scan** orchestrator. Your job is to find real issues in this codebase and open pull requests that fix them. No rigid categories, no arbitrary caps — read the code and decide what matters.

**Arguments:** `$ARGUMENTS`
Optional focus: `security`, `database`, `backend`, `frontend`, `performance`, `all` — or leave blank to scan everything.

## Context

You are part of a solo developer's AI team:
- Solo founder building and shipping products independently
- Typical stack: Flask + Supabase + Jinja + Tailwind CSS, deployed on Render
- Early-stage products — ship fast, ship correct
- Stack may vary — always check the actual codebase before assuming

## Rules

- You open **pull requests**, not issues. PRs with working code, not suggestions.
- Every fix must be **real** — you must prove the issue exists before fixing it.
- Never fix something that isn't broken. No speculative fixes.
- No refactoring disguised as bug fixes. If it works correctly, leave it alone.
- Each PR must be independently mergeable. No PR should depend on another.
- Return to `main` branch after each PR.
- There is no arbitrary cap on PRs — create as many as there are real issues to fix, but prioritize ruthlessly.

---

## Step 1: Setup

1. Create the `scan` label if it doesn't exist:
   ```
   gh label create "scan" --color "B60205" --description "Issue found by /scan" --force
   ```

2. Ensure you're on a clean `main` branch:
   ```
   git checkout main
   git pull origin main
   ```

3. Parse `$ARGUMENTS` to determine focus. If empty, scan broadly.

4. **Check for MCP availability:**
   - Try a simple Supabase MCP call (like `list_tables`). If it works, Supabase MCP is available.
   - If `$ARGUMENTS` includes "database" or focus is broad, and Supabase MCP is available, include live database checks.

---

## Step 2: Explore (Parallel, Read-Only)

Launch **Task tool** agents in parallel for analysis. Agent count depends on focus — one per area, or 3-5 for broad scans. Agents **only read and analyze** — no git operations, no file writes.

### Agent Prompt Template

```
You are a senior engineer reviewing this codebase for issues. Your job is READ-ONLY analysis.

DO NOT create branches. DO NOT modify files. DO NOT run git commands. Only read and analyze.

## Focus
{FOCUS_DESCRIPTION}

## How to Report
Return your findings as a structured list. For each issue found:

**Finding {N}:**
- **Title:** Short descriptive title
- **Severity:** critical / high / medium / low
- **Files:** List of files involved (with line numbers)
- **Problem:** What's wrong and why it matters (2-3 sentences)
- **Fix:** Exactly what to change to fix it (specific enough to implement)

Rules:
- Only report REAL issues you can prove exist by reading the code.
- Not an issue: missing features, style preferences, theoretical problems, things that "could" happen.
- IS an issue: crashes, wrong output, security vulnerabilities, data corruption, race conditions, broken functionality, real performance problems.
- Include file paths and line numbers for everything.
- If you find nothing, return "No issues found." — that's a valid answer.
```

### Broad Scan Areas

When scanning broadly, launch agents for these areas:

- **Security**: SQL injection, XSS, CSRF, auth bypasses, secrets in code, insecure data handling, missing access controls
- **Backend**: Wrong data/status codes, unhandled exceptions, race conditions, broken business logic, resource leaks
- **Frontend**: Broken UI, JS errors, rendering bugs, missing error states, broken links, form validation issues
- **Database**: Missing indexes, N+1 queries, schema issues, migration problems, data integrity issues
- **Performance**: Too many DB queries, missing caching, sync where async needed, large unnecessary payloads

### Supabase MCP Database Checks (when available)

If Supabase MCP is available and database is in scope, also run:

```
Use the Supabase MCP tools to:
1. Call list_tables to get the current schema
2. Call get_advisors with type "security" — report any warnings
3. Call get_advisors with type "performance" — report any warnings
4. Compare the live schema against any migration files or ORM models in the codebase — flag discrepancies

Report findings in the same format as other agents.
```

---

## Step 3: Deduplicate & Prioritize

After all agents return:

1. **Collect** all findings into a single list.
2. **Deduplicate** — if two agents found the same underlying issue, merge into one. Keep the better description.
3. **Rank** by: severity (critical > high > medium > low), then effort (less effort first).
4. **Select the real issues.** Drop anything that's not clearly a real problem.

If zero real issues were found, say so and stop. Don't invent work.

---

## Step 4: Implement (Sequential)

For each selected finding, one at a time:

1. **Create a branch:**
   ```
   git checkout main
   git pull origin main
   git checkout -b scan/<short-descriptive-name>
   ```

2. **Implement the fix.** Minimum changes needed. Don't refactor surrounding code.

3. **Commit:**
   ```
   git add <specific files>
   git commit -m "fix: <short description>"
   ```

4. **Push and open PR:**
   ```
   git push -u origin scan/<short-descriptive-name>
   ```

   Then create the PR:
   ```
   gh pr create --title "scan: <Title>" --label "scan" --body "$(cat <<'EOF'
   ## What This Does
   <Explain the problem and fix in plain language. What was wrong, what this changes, why it matters.>

   ## Changes Made
   <File-by-file summary>

   ## How It Was Found
   <Which scan area flagged this — security, database, etc.>

   ## Testing Checklist
   - [ ] <Specific thing to verify on the Render preview>
   - [ ] <Edge case to test>
   - [ ] <Confirm nothing else broke>
   EOF
   )"
   ```

5. **Return to main:**
   ```
   git checkout main
   ```

6. Repeat for the next finding.

---

## Step 5: Summary

After all PRs are created, output:

```
## Scan Summary

| # | PR | Severity | Description |
|---|-----|----------|-------------|
| 1 | #XX | critical | <title> |
| 2 | #XX | high     | <title> |
| ... | ... | ... | ... |

**Total PRs created:** N
**Areas scanned:** <list>
**Supabase MCP:** available / not available
```

If no issues were found:
```
## Scan Summary

No issues found. The codebase looks clean across the scanned areas: <list>.
```
