# Managing Projects

## List projects

```bash
acli jira project list
acli jira project list --paginate --json
acli jira project list --recent
```

### Options

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--recent` | | Last 20 recently viewed | `false` |
| `--limit` | `-l` | Max projects | `30` |
| `--paginate` | | Fetch all | `false` |
| `--json` | | JSON output | `false` |

## View project details

```bash
acli jira project view --key PROJ
acli jira project view --key PROJ --json
```

## Create a project

```bash
# Clone from existing project
acli jira project create --from-project EXISTING --key NEWPROJ --name "New Project"

# From JSON template
acli jira project create --from-json project.json

# Generate template
acli jira project create --generate-json
```

### Options

| Flag | Short | Description |
|------|-------|-------------|
| `--key` | `-k` | Project key |
| `--name` | `-n` | Project name |
| `--description` | `-d` | Description |
| `--lead-email` | `-l` | Lead user email |
| `--url` | `-u` | Project URL |
| `--from-project` | `-f` | Clone from existing (company-managed only) |
| `--from-json` | `-j` | Create from JSON |
| `--generate-json` | `-g` | Print JSON template |

## Update a project

```bash
acli jira project update --project-key PROJ --name "New Name" --lead-email new-lead@co.com
acli jira project update --project-key PROJ --from-json project.json
```

| Flag | Short | Description |
|------|-------|-------------|
| `--project-key` | `-p` | Key of project to update |
| `--key` | `-k` | New project key |
| `--name` | `-n` | New name |
| `--description` | `-d` | New description |
| `--lead-email` | `-l` | New lead email |
| `--url` | `-u` | New URL |

## Archive / Restore / Delete

```bash
acli jira project archive --key PROJ
acli jira project restore --key PROJ
acli jira project delete --key PROJ
```

## Examples

```bash
# List all projects as JSON
acli jira project list --paginate --json

# Clone a project
acli jira project create --from-project TEMPLATE --key MYPROJ --name "My Project"

# Transfer project lead
acli jira project update --project-key PROJ --lead-email new-lead@co.com
```
