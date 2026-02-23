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

### mcp-client

Universal MCP client for connecting to any MCP server with progressive disclosure. Load tool schemas on-demand instead of bloating your context window.

**Features:**
- Connects to Zapier, GitHub, Sequential Thinking, and any MCP server
- Supports stdio, SSE, HTTP, and FastMCP transports
- Auto-detects transport type from config
- JSON output for easy integration

**Usage:**
```bash
# List configured servers
python .claude/skills/mcp-client/scripts/mcp_client.py servers

# List tools from a server
python .claude/skills/mcp-client/scripts/mcp_client.py tools zapier

# Call a tool
python .claude/skills/mcp-client/scripts/mcp_client.py call zapier <tool> '{"arg": "value"}'
```

**Setup:** Copy `references/example-mcp-config.json` to `references/mcp-config.json` and add your API keys.

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

## Attribution

The `mcp-client` and `sop-creator` skills are based on [second-brain-skills](https://github.com/coleam00/second-brain-skills) by Cole Medin.

## License

MIT
