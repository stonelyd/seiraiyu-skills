---
name: jira
description: >
  Manage Jira issues, projects, and sprints from the CLI.
  Use when the user needs to create, search, update, or triage Jira issues,
  view sprints, manage projects, or automate Jira workflows from the terminal.
allowed-tools: Bash(jira:*)
---

# Jira CLI

Manage Jira issues, projects, and sprints from the terminal. Non-interactive — designed for automation and CI/CD pipelines.

## Quick start

```bash
# Configure (Bearer auth — token only)
jira config --server https://your-jira.atlassian.net --token YOUR_TOKEN

# Or with Basic auth (email + token)
jira config --server https://your-jira.atlassian.net \
  --username your-email@company.com --token YOUR_TOKEN

# List your in-progress issues
jira issue list --assignee currentUser --status "In Progress"

# View an issue
jira issue view PROJ-123

# Create an issue
jira issue create --project PROJ --type Bug \
  --summary "Login fails on mobile" --priority High

# Add a comment
jira issue comment add PROJ-123 "Fixed in commit abc123"

# Active sprint
jira sprint active --board 42
```

## Commands

| Command | Purpose |
|---------|---------|
| `jira config` | Configure server, credentials. `--show`, `--server`, `--username`, `--token`. |
| `jira config get [key]` | Get config value. |
| `jira config set <key> <value>` | Set config value. |
| `jira config unset <key>` | Remove config value. |
| `jira issue list` | List issues. `--project`, `--assignee`, `--status`, `--type`, `--reporter`, `--priority`, `--created`, `--updated`, `--limit`, `--jql`. |
| `jira issue view <KEY>` | View issue details. `--format` (terminal/markdown), `--output <file>`. |
| `jira issue create` | Create issue. `--project`, `--type`, `--summary` (required). `--description`, `--description-file`, `--assignee`, `--priority`. |
| `jira issue edit <KEY>` | Edit issue. `--summary`, `--description`, `--description-file`, `--assignee`, `--priority`. |
| `jira issue delete <KEY>` | Delete issue. `-f/--force` required. |
| `jira issue comment add <KEY> [text]` | Add comment. `--file`, `--internal`. |
| `jira issue comment list <KEY>` | List comments. `--format` (table/json). |
| `jira issue comment edit <ID> [text]` | Edit comment. `--file`. |
| `jira issue comment delete <ID>` | Delete comment. `-f/--force` required. |
| `jira project list` | List projects. `--type`, `--category`. |
| `jira project view <KEY>` | Project details with components and versions. |
| `jira project components <KEY>` | List project components. |
| `jira project versions <KEY>` | List project versions. |
| `jira sprint list` | List sprints. `-b/--board`, `-a/--active`, `--state`. |
| `jira sprint active` | Active sprint shortcut. `-b/--board`. |
| `jira sprint boards` | List all boards. |

## Aliases

| Full | Alias |
|------|-------|
| `jira config` | `jira c` |
| `jira issue` | `jira i` |
| `jira issue list` | `jira issue ls` |
| `jira issue view` | `jira issue show` |
| `jira issue create` | `jira issue new` |
| `jira issue edit` | `jira issue update` |
| `jira issue delete` | `jira issue rm` |
| `jira project` | `jira p` |
| `jira project list` | `jira project ls` |
| `jira project view` | `jira project show` |
| `jira sprint` | `jira s` |
| `jira sprint list` | `jira sprint ls` |

## Authentication

**Bearer auth** (recommended): `jira config --server <url> --token <token>`

**Basic auth**: `jira config --server <url> --username <email> --token <token>`

**Environment variables** (override stored config):

| Variable | Description |
|----------|-------------|
| `JIRA_HOST` | Jira domain (auto-prefixed with `https://`) |
| `JIRA_API_TOKEN` | API token |
| `JIRA_USERNAME` | Email (optional, for Basic auth) |
| `JIRA_API_VERSION` | `auto` (default), `2`, or `3` |

Legacy env vars: `JIRA_DOMAIN`, `JIRA_USERNAME`, `JIRA_API_TOKEN`.

**Get an API token**: https://id.atlassian.com/manage-profile/security/api-tokens

## Global options

All commands accept: `--config <path>`, `--verbose`, `--no-color`.

## API version

Default `auto`: tries v3 first, falls back to v2 on 404/410. Override with `jira config set apiVersion 2`.

## Detailed guides

- **Issues & Comments** — [references/issues.md](references/issues.md)
- **Projects** — [references/projects.md](references/projects.md)
- **Sprints & Boards** — [references/sprints.md](references/sprints.md)
