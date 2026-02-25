# Managing Work Items & Comments

## Search work items

```bash
acli jira workitem search --jql "project = PROJ AND assignee = currentUser()" --limit 50
acli jira workitem search --jql "project = PROJ" --paginate --json
acli jira workitem search --jql "project = PROJ" --count
```

### Options

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| `--jql` | `-j` | JQL query | — |
| `--filter` | | Saved filter ID | — |
| `--fields` | `-f` | Fields to display | `issuetype,key,assignee,priority,status,summary` |
| `--limit` | `-l` | Max results | — |
| `--paginate` | | Fetch all pages | `false` |
| `--count` | | Return count only | `false` |
| `--csv` | | CSV output | `false` |
| `--json` | | JSON output | `false` |
| `--web` | `-w` | Open in browser | `false` |

### Field specifiers

- `*all` — all fields
- `*navigable` — all navigable fields
- `field1,field2` — specific fields
- `-description` — exclude a field
- `*navigable,-comment` — navigable minus comment

## View a work item

```bash
acli jira workitem view KEY-123
acli jira workitem view KEY-123 --json
acli jira workitem view KEY-123 --fields summary,status,comment
acli jira workitem view KEY-123 --web
```

| Flag | Short | Description |
|------|-------|-------------|
| `--fields` | `-f` | Fields to display (default: `key,issuetype,summary,status,assignee,description`) |
| `--json` | | JSON output |
| `--web` | `-w` | Open in browser |

## Create a work item

```bash
acli jira workitem create --project PROJ --type Bug --summary "Login broken" \
  --description "Steps to reproduce..." --assignee "@me" --label "urgent"
```

### Options

| Flag | Short | Description |
|------|-------|-------------|
| `--project` | `-p` | Project key (required) |
| `--type` | `-t` | Issue type: `Epic`, `Story`, `Task`, `Bug` (required) |
| `--summary` | `-s` | Summary (required) |
| `--description` | `-d` | Description (plain text or ADF) |
| `--description-file` | | Read description from file |
| `--assignee` | `-a` | Assignee email or `@me` |
| `--label` | `-l` | Labels (comma-separated) |
| `--parent` | | Parent issue key |
| `--editor` | `-e` | Open editor for summary/description |
| `--from-json` | | Create from JSON file |
| `--generate-json` | | Print JSON template |

### Bulk create

```bash
# From CSV (columns: summary, projectKey, issueType, description, label, parentIssueId, assignee)
acli jira workitem create-bulk --from-csv issues.csv --yes

# From JSON
acli jira workitem create-bulk --from-json issues.json --ignore-errors

# Generate template
acli jira workitem create-bulk --generate-json
```

## Edit work items

```bash
# Single issue
acli jira workitem edit --key KEY-123 --summary "New title" --assignee user@co.com

# Bulk via JQL
acli jira workitem edit --jql "project = PROJ AND status = Open" --assignee user@co.com --yes

# Bulk via filter
acli jira workitem edit --filter 10001 --description "Updated" --yes
```

| Flag | Short | Description |
|------|-------|-------------|
| `--key` | `-k` | Work item keys (comma-separated) |
| `--jql` | | JQL for bulk edit |
| `--filter` | | Filter ID for bulk edit |
| `--summary` | `-s` | New summary |
| `--description` | `-d` | New description |
| `--description-file` | | Description from file |
| `--assignee` | `-a` | New assignee (`@me`, email) |
| `--remove-assignee` | | Remove assignee |
| `--labels` | `-l` | Set labels |
| `--remove-labels` | | Remove labels |
| `--type` | `-t` | Change issue type |
| `--from-json` | | Edit from JSON |
| `--ignore-errors` | | Continue on errors |
| `--yes` | `-y` | Skip confirmation |

## Assign work items

```bash
acli jira workitem assign --key KEY-123 --assignee "@me"
acli jira workitem assign --jql "project = PROJ" --assignee user@co.com --yes
acli jira workitem assign --key KEY-123 --remove-assignee
```

## Transition (change status)

```bash
acli jira workitem transition --key KEY-123 --status "Done"
acli jira workitem transition --jql "project = PROJ AND status = 'To Do'" --status "In Progress" --yes
```

## Delete work items

```bash
acli jira workitem delete --key KEY-123 --yes
acli jira workitem delete --jql "project = TEST AND type = Task" --yes
```

## Clone work items

```bash
acli jira workitem clone --key KEY-123 --to-project NEWPROJ
acli jira workitem clone --key KEY-123 --to-site other.atlassian.net
```

## Archive / Unarchive

```bash
acli jira workitem archive --jql "project = PROJ AND status = Done" --yes
acli jira workitem unarchive --key KEY-123
```

---

## Comments

### Add a comment

```bash
acli jira workitem comment create --key KEY-123 --body "This is fixed in v2.1"
acli jira workitem comment create --key KEY-123 --body-file review.txt
acli jira workitem comment create --key KEY-123 --editor
```

### List comments

```bash
acli jira workitem comment list --key KEY-123
acli jira workitem comment list --key KEY-123 --json --paginate
```

| Flag | Description | Default |
|------|-------------|---------|
| `--limit` | Max comments per page | `50` |
| `--order` | Sort: `+created`, `-created`, `+updated`, `-updated` | `+created` |
| `--paginate` | Fetch all | `false` |

### Update a comment

```bash
acli jira workitem comment update --key KEY-123 --id 10001 --body "Updated text"
acli jira workitem comment update --key KEY-123 --id 10001 --visibility-role "Administrators"
```

### Delete a comment

```bash
acli jira workitem comment delete --key KEY-123 --id 10001
```

---

## Links

### Create a link

```bash
acli jira workitem link create --out KEY-123 --in KEY-456 --type Blocks
acli jira workitem link create --from-json links.json
```

### List links

```bash
acli jira workitem link list --key KEY-123
```

### Delete a link

```bash
acli jira workitem link delete --id 10001
```

---

## Attachments

```bash
acli jira workitem attachment list --key KEY-123
acli jira workitem attachment delete --id 12345
```

---

## Common workflows

```bash
# Daily standup — my in-progress work
acli jira workitem search --jql "assignee = currentUser() AND status = 'In Progress'"

# Bug triage
acli jira workitem search --jql "project = PROJ AND type = Bug AND status = Open" --limit 50

# CI/CD — transition on deploy
acli jira workitem transition --key "$ISSUE_KEY" --status "Done" --yes

# Export issues to CSV
acli jira workitem search --jql "project = PROJ" --paginate --csv > issues.csv

# Bulk reassign
acli jira workitem assign --jql "assignee = departing@co.com" --assignee new@co.com --yes
```
