# claude-ops

AI-powered development operations toolkit. Slash commands for Claude Code that scan for bugs, redesign UIs, run visual QA, check project health, and more.

## Commands

| Command | What it does | MCPs used |
|---------|-------------|-----------|
| `/scan` | Find and fix real issues, open PRs | Supabase (optional, for DB checks) |
| `/design` | Visual design changes with before/after screenshots | Playwright (optional, for screenshots) |
| `/plan` | Create detailed implementation plan as GitHub issue | Supabase (optional, for schema) |
| `/qa` | Visual testing — navigate site, test flows, report bugs | Playwright (required) |
| `/report` | Development activity summary | GitHub (built-in) |
| `/health` | Project health check + CLAUDE.md audit | Supabase (optional, for advisors) |

## Workflows

| Workflow | Trigger | What it does |
|----------|---------|-------------|
| `claude.yml` | `@claude` mentions in issues/PRs | Responds to mentions with full MCP access |
| `claude-code-review.yml` | PR opened/updated | Automated code review |
| `claude-dispatch.yml` | Manual (Actions UI) | Run any command from the GitHub Actions UI |

## Installation

Copy the `.claude/` and `.github/` directories into your target repo:

```bash
# From your target repo root
cp -r /path/to/claude-ops/.claude/ .claude/
cp -r /path/to/claude-ops/.github/ .github/
```

Or cherry-pick what you need — each file is self-contained.

## GitHub Secrets

Set these in your target repo's Settings > Secrets and variables > Actions:

| Secret | Required | Purpose |
|--------|----------|---------|
| `CLAUDE_CODE_OAUTH_TOKEN` | Yes | Powers all Claude workflows |
| `SUPABASE_ACCESS_TOKEN` | Optional | Supabase MCP in CI (database checks, health advisors) |
| `SUPABASE_PROJECT_REF` | Optional | Scopes Supabase MCP to your project |

## MCP Configuration

### Local
MCPs configured in your local Claude Code settings work automatically. Commands detect MCP availability at runtime and degrade gracefully.

### CI
- **Supabase**: Configured automatically by workflows when secrets are present (remote HTTP MCP)
- **Playwright**: Added automatically for `/design` and `/qa` commands only
- **GitHub**: Built into `claude-code-action` already

## CLAUDE.md Template

The `CLAUDE.md.template` file is a reference for creating project-specific `CLAUDE.md` files. A good CLAUDE.md directly improves code review quality — the `/health` command audits yours against this template.

## Usage

### Local (Claude Code CLI)
```
/scan                     # Scan everything
/scan security            # Focus on security issues
/design dark modern       # Redesign with dark modern aesthetic
/plan add stripe payments # Create implementation plan
/qa http://localhost:5000 # Run visual QA on local dev
/report last week         # Activity summary for last week
/health                   # Full project health check
/health claude-md         # Just audit the CLAUDE.md
```

### CI (GitHub Actions UI)
1. Go to Actions > Claude Ops Dispatch
2. Select command from dropdown
3. Enter arguments
4. Click "Run workflow"

### @claude mentions
Comment `@claude <task>` on any issue or PR to trigger Claude with full MCP access.