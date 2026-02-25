---
name: jira
description: >
  Manage Jira issues, projects, sprints, and boards using the Atlassian CLI (acli).
  Use when the user needs to create, search, update, transition, or triage Jira work items,
  manage projects, view sprints, bulk operations, or automate Jira workflows from the terminal.
allowed-tools: Bash(acli:*)
---

# Jira CLI (acli)

Manage Jira Cloud using the official Atlassian CLI. Supports work items, projects, boards, sprints, filters, dashboards, and bulk operations via CSV/JSON.

## Quick start

```bash
# Authenticate (API token via stdin)
echo YOUR_TOKEN | acli jira auth login --site mysite.atlassian.net --email user@company.com --token

# Or authenticate via browser (OAuth)
acli jira auth login --web

# Search work items
acli jira workitem search --jql "project = PROJ AND assignee = currentUser()" --limit 20

# View an issue
acli jira workitem view KEY-123

# Create a work item
acli jira workitem create --project PROJ --type Bug --summary "Login fails on mobile"

# Transition status
acli jira workitem transition --key KEY-123 --status "In Progress"

# Add a comment
acli jira workitem comment create --key KEY-123 --body "Fixed in commit abc123"

# Find boards and sprints
acli jira board search --type scrum
acli jira board list-sprints --id 42 --state active
```

## Commands

| Command | Purpose |
|---------|---------|
| **Auth** | |
| `acli jira auth login` | Authenticate. `--site`, `--email`, `--token` (stdin) or `--web` (OAuth). |
| `acli jira auth logout` | Remove authentication. |
| `acli jira auth status` | Check auth state. |
| `acli jira auth switch` | Switch authenticated account. |
| **Work Items** | |
| `acli jira workitem create` | Create issue. `--project`, `--type`, `--summary`, `--description`, `--assignee`, `--label`, `--parent`. |
| `acli jira workitem create-bulk` | Bulk create. `--from-csv`, `--from-json`. |
| `acli jira workitem view [key]` | View issue. `--fields`, `--json`, `--web`. |
| `acli jira workitem search` | Search issues. `--jql`, `--filter`, `--fields`, `--limit`, `--paginate`, `--count`. |
| `acli jira workitem edit` | Edit issues. `--key`, `--jql`, `--summary`, `--description`, `--assignee`, `--labels`, `--type`. |
| `acli jira workitem assign` | Assign issues. `--key`, `--jql`, `--assignee` (`@me`, email), `--remove-assignee`. |
| `acli jira workitem transition` | Change status. `--key`, `--jql`, `--status`. |
| `acli jira workitem delete` | Delete issues. `--key`, `--jql`, `--yes`. |
| `acli jira workitem clone` | Duplicate issues. `--key`, `--to-project`, `--to-site`. |
| `acli jira workitem archive` | Archive issues. `--key`, `--jql`. |
| `acli jira workitem unarchive` | Restore archived. `--key`. |
| **Comments** | |
| `acli jira workitem comment create` | Add comment. `--key`, `--body`, `--body-file`, `--editor`. |
| `acli jira workitem comment list` | List comments. `--key`, `--limit`, `--order`, `--paginate`. |
| `acli jira workitem comment update` | Edit comment. `--key`, `--id`, `--body`. |
| `acli jira workitem comment delete` | Delete comment. `--key`, `--id`. |
| **Links & Attachments** | |
| `acli jira workitem link create` | Link issues. `--out`, `--in`, `--type`. |
| `acli jira workitem link list` | List links. `--key`. |
| `acli jira workitem link delete` | Delete link. `--id`. |
| `acli jira workitem attachment list` | List attachments. `--key`. |
| `acli jira workitem attachment delete` | Delete attachment. `--id`. |
| **Projects** | |
| `acli jira project list` | List projects. `--limit`, `--paginate`, `--recent`. |
| `acli jira project view` | View project. `--key`, `--json`. |
| `acli jira project create` | Create project. `--key`, `--name`, `--from-project`. |
| `acli jira project update` | Update project. `--project-key`, `--name`, `--description`, `--lead-email`. |
| `acli jira project archive` | Archive project. `--key`. |
| `acli jira project restore` | Restore project. `--key`. |
| `acli jira project delete` | Delete project. `--key`. |
| **Boards & Sprints** | |
| `acli jira board search` | Search boards. `--name`, `--project`, `--type` (scrum/kanban/simple). |
| `acli jira board list-sprints` | List sprints. `--id` (board), `--state` (active/future/closed). |
| `acli jira sprint list-workitems` | Sprint issues. `--board`, `--sprint`, `--fields`, `--jql`. |
| **Filters & Dashboards** | |
| `acli jira filter search` | Search filters. `--name`, `--owner`. |
| `acli jira filter list` | List filters. `--my`, `--favourite`. |
| `acli jira filter change-owner` | Transfer ownership. `--id`, `--owner`. |
| `acli jira dashboard search` | Search dashboards. `--name`, `--owner`. |
| **Fields** | |
| `acli jira field create` | Create custom field. `--name`, `--type`. |
| `acli jira field delete` | Trash custom field. `--id`. |

## Output formats

Default is table. Add `--json` or `--csv` to any command for machine-readable output.

```bash
acli jira workitem search --jql "project = PROJ" --json | jq '.[].key'
acli jira workitem search --jql "project = PROJ" --csv > issues.csv
```

## Bulk operations

All bulk commands accept `--key` (comma-separated), `--jql`, `--filter`, `--from-file`, `--from-csv`, `--from-json`. Use `--ignore-errors` to continue on failures, `--yes` to skip confirmation.

```bash
# Bulk edit via JQL
acli jira workitem edit --jql "project = PROJ AND status = Open" --assignee user@co.com --yes

# Bulk create from CSV
acli jira workitem create-bulk --from-csv issues.csv --yes

# Generate JSON template
acli jira workitem create --generate-json
```

## Authentication

**API token**: `echo TOKEN | acli jira auth login --site site.atlassian.net --email user@co.com --token`

**OAuth (browser)**: `acli jira auth login --web`

**CI/CD**: Store token as secret, pipe to `--token` flag.

**Get an API token**: https://id.atlassian.com/manage-profile/security/api-tokens

## Detailed guides

- **Work Items & Comments** — [references/workitems.md](references/workitems.md)
- **Projects** — [references/projects.md](references/projects.md)
- **Boards & Sprints** — [references/boards-sprints.md](references/boards-sprints.md)
