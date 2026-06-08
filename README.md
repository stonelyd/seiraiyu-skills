# Seiraiyu Skills

Productivity skills for Claude Code.

## Installation

Via the Seiraiyu Marketplace:

```bash
/plugin marketplace add stonelyd/seiraiyu-marketplace
/plugin install seiraiyu-skills@seiraiyu-marketplace
```

## Skills

### respond-to-coderabbitai

Automatically resolve PR review comments from CodeRabbit (or any reviewer) with atomic commits.

**Features:**
- Discovers all unresolved review threads via GraphQL
- Groups related comments by logical issue
- Creates atomic commits per issue
- Posts threaded replies with commit links
- Handles impasses with escalation to GitHub issues

**Usage:**
```
/seiraiyu-skills:respond-to-coderabbitai
```

### sop-creator

Create runbooks, playbooks, and technical documentation that people actually follow.

**Features:**
- Templates for runbooks, SOPs, how-to guides, checklists, decision trees, onboarding
- Philosophy: scannable, actionable, specific, testable
- Definition of Done as primary success criteria
- Anti-patterns and writing rules included

**Usage:** Trigger with requests like "create a runbook for...", "document this process", "write a playbook"

**Templates available:**
- `references/runbook.md` - Incidents, emergencies, on-call
- `references/standard-sop.md` - Any repeatable process
- `references/how-to-guide.md` - One-off tasks, setup
- `references/checklist.md` - QC, verification
- `references/decision-tree.md` - Complex if/then flows
- `references/onboarding-guide.md` - New person ramping up

### neon

Manage Neon serverless Postgres from the CLI. Projects, branches, databases, roles, connection strings.

**Features:**
- Full command reference for the Neon CLI (`neonctl`)
- Branch management including reset, restore, schema-diff
- Connection string generation with pooling, Prisma, and psql support
- Project lifecycle management
- IP allowlist and VPC configuration
- Agent-friendly `--output json` guidance

**Usage:**
```bash
# Set context so you don't repeat --project-id
neon set-context --project-id <id>

# Create a branch
neon branches create --name dev/feature

# Get connection string
neon cs dev/feature --pooled

# Schema diff
neon branches schema-diff main dev/feature
```

**Requires:** `neon` CLI installed (`npm i -g neonctl`)

### jira

Manage Jira projects, boards, sprints, and work items using the Atlassian CLI (acli).

**Features:**
- Project and board management
- Sprint lifecycle (create, start, complete)
- Work item operations (create, update, transition, assign)
- JQL search for issues
- Agent-friendly JSON output

**Usage:**
```bash
# List projects
acli jira project list --output json

# Create an issue
acli jira issue create --project KEY --type Story --summary "Title" --description "Details"

# Search issues
acli jira issue search --jql "project = KEY AND status = 'In Progress'"
```

**Requires:** `acli` CLI installed

### obsidian-cli

Control Obsidian vaults from the terminal. The CLI communicates with the running Obsidian desktop app via IPC for link-aware vault operations.

**Features:**
- Note CRUD with wikilink-style name resolution
- Property/frontmatter management (type-aware: text, list, number, checkbox, date, tags)
- Tags, links, backlinks, orphans, and unresolved link analysis
- Task management (list, toggle, filter by status)
- Template support (list, read, create notes from templates)
- Daily notes (create, read, append)
- Search with context and folder scoping
- Sync & local history (status, versions, restore)
- Plugin management (install, enable, disable, reload)
- Bases (databases), bookmarks, developer tools

**Usage:**
```bash
# Read a note
obsidian read file="Note Name"

# Create with template
obsidian create name="Title" template="Template Name"

# Search vault
obsidian search query="term" format=json

# Manage properties
obsidian property:set name=status value=active file="Note"
```

**Requires:** Obsidian desktop running. `obsidian` CLI installed.

### e2e-test-agent-browser

Comprehensive E2E browser testing using the Vercel Agent Browser CLI. Launches parallel sub-agents to research the codebase, then tests every user journey with screenshots, database validation, and bug detection.

**Features:**
- Parallel research phase (app structure, database schema, bug hunting)
- Full browser automation (click, fill, select, screenshot)
- Database record validation after each interaction
- Responsive testing across mobile, tablet, and desktop viewports
- Automatic issue detection and fixing

**Usage:**
```
/seiraiyu-skills:e2e-test-agent-browser
```

**Requires:** Linux, WSL, or macOS. `agent-browser` CLI installed (`npm i -g agent-browser`)

### e2e-test-playwright-cli

Comprehensive E2E browser testing using the Playwright CLI (@playwright/cli). Same 6-phase structure as agent-browser but with additional capabilities: named sessions, state persistence, tracing, multi-tab support, network monitoring, and device emulation.

**Features:**
- Parallel research phase (app structure, database schema, bug hunting)
- Full browser automation with organized command groups (core, navigation, diagnostics, tabs, tracing, state)
- Named sessions to prevent conflicts
- State persistence — save/load auth state across journeys
- Trace recording for debugging failures
- Network request monitoring for API error detection
- Device preset emulation for responsive testing
- Cross-platform support (Linux, macOS, WSL, Windows)

**Usage:**
```
/seiraiyu-skills:e2e-test-playwright-cli
```

**Requires:** `@playwright/cli` installed (`npm i -g @playwright/cli@latest`)

### handoff

Compact the current conversation into a self-contained handoff document so a fresh agent (or a future you) can resume the work without re-reading the whole session.

**Features:**
- Writes to the OS temp directory (never the workspace) with a timestamped, descriptive filename
- References durable artifacts (PRDs, plans, ADRs, issues, PRs, commits) by path/URL instead of duplicating them
- Captures decisions + rationale, blockers, dead ends, and the exact next action
- Records git state and the commands needed to resume
- Redacts secrets and PII
- Includes a "Suggested skills" section for the next agent
- Optional argument tailors the doc to what the next session will focus on

**Usage:**
```
/seiraiyu-skills:handoff
/seiraiyu-skills:handoff wire up the Redis store for the rate limiter
```

## Attribution

The `sop-creator` skill is based on [second-brain-skills](https://github.com/coleam00/second-brain-skills) by Cole Medin.

## License

MIT
