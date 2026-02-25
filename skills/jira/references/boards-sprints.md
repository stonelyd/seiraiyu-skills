# Boards, Sprints, Filters & Dashboards

## Search boards

```bash
acli jira board search
acli jira board search --type scrum --project PROJ
acli jira board search --name "Sprint" --json
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--name` | Partial name match | — |
| `--project` | Filter by project | — |
| `--type` | Board type: `scrum`, `kanban`, `simple` | — |
| `--filter` | Filter by filter ID | — |
| `--orderBy` | Order: `name`, `-name`, `+name` | — |
| `--limit` | Max results | `50` |
| `--paginate` | Fetch all | `false` |
| `--private` | Include private boards | `false` |
| `--csv` | CSV output | `false` |
| `--json` | JSON output | `false` |

## List sprints on a board

```bash
acli jira board list-sprints --id 42
acli jira board list-sprints --id 42 --state active,future
acli jira board list-sprints --id 42 --json
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--id` | Board ID (required) | — |
| `--state` | Filter: `future`, `active`, `closed` (comma-separated) | — |
| `--limit` | Max sprints | `50` |
| `--paginate` | Fetch all | `false` |
| `--csv` | CSV output | `false` |
| `--json` | JSON output | `false` |

## List sprint work items

```bash
acli jira sprint list-workitems --sprint 456 --board 42
acli jira sprint list-workitems --sprint 456 --board 42 --jql "status = 'In Progress'" --json
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--board` | Board ID (required) | — |
| `--sprint` | Sprint ID (required) | — |
| `--fields` | Fields to display | `key,issuetype,summary,assignee,priority,status` |
| `--jql` | JQL filter within sprint | — |
| `--limit` | Max issues | `50` |
| `--paginate` | Fetch all | `false` |
| `--csv` | CSV output | `false` |
| `--json` | JSON output | `false` |

---

## Filters

### Search filters

```bash
acli jira filter search
acli jira filter search --owner user@co.com --name "report"
acli jira filter search --paginate --csv
```

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--name` | `-n` | Partial name match | — |
| `--owner` | `-e` | Owner email | — |
| `--limit` | `-l` | Max results | `30` |
| `--paginate` | | Fetch all | `false` |

### List personal/favorite filters

```bash
acli jira filter list --my
acli jira filter list --favourite
```

### Transfer filter ownership

```bash
acli jira filter change-owner --id 123,456,789 --owner new-owner@co.com
```

### Add favorite

```bash
acli jira filter add-favourite --filterId 10001
```

---

## Dashboards

```bash
acli jira dashboard search
acli jira dashboard search --owner user@co.com --name "report" --json
acli jira dashboard search --paginate --csv
```

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--name` | `-n` | Partial name match | — |
| `--owner` | `-e` | Owner email | — |
| `--limit` | `-l` | Max results | `30` |
| `--paginate` | | Fetch all | `false` |

---

## Common workflows

```bash
# Sprint planning
acli jira board search --type scrum --project PROJ
acli jira board list-sprints --id 42 --state active
acli jira sprint list-workitems --sprint 456 --board 42

# View in-progress sprint items
acli jira sprint list-workitems --sprint 456 --board 42 \
  --jql "status = 'In Progress'" --json

# Filter admin — transfer departing user's filters
acli jira filter search --owner departing@co.com --paginate
acli jira filter change-owner --id 123,456,789 --owner new-owner@co.com

# Export sprint data
acli jira sprint list-workitems --sprint 456 --board 42 --paginate --csv > sprint.csv
```
