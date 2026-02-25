# Managing Issues & Comments

## List issues

```bash
jira issue list --project PROJ --status "In Progress" --limit 50
```

### Filter options

| Flag | Description |
|------|-------------|
| `--project <key>` | Project key |
| `--assignee <user>` | Assignee. Use `currentUser` for yourself. |
| `--status <status>` | Status (e.g., `Open`, `In Progress`, `Done`) |
| `--type <type>` | Issue type (e.g., `Bug`, `Story`, `Task`) |
| `--reporter <user>` | Reporter |
| `--priority <level>` | Priority (e.g., `High`, `Medium`, `Low`) |
| `--created <date>` | Created filter (e.g., `-7d`, `2024-01-01`) |
| `--updated <date>` | Updated filter |
| `--limit <n>` | Max results (default: 20) |
| `--jql <query>` | Raw JQL (overrides other filters) |

Filters are combined with AND. `--assignee currentUser` maps to JQL `currentUser()`.

### Examples

```bash
# My open bugs
jira issue list --assignee currentUser --type Bug --status Open

# Recent issues in a project
jira issue list --project PROJ --created -7d --limit 50

# Custom JQL
jira issue list --jql "project = PROJ AND priority = High ORDER BY created DESC"
```

## View an issue

```bash
jira issue view PROJ-123
jira issue view PROJ-123 --format markdown
jira issue view PROJ-123 --output ./issue.md
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--format <fmt>` | `terminal` or `markdown` | `terminal` |
| `--output <path>` | Save to file (always markdown) | — |

Terminal output shows: Key, Summary, Status, Type, Priority, Assignee, Reporter, Created, Updated, Description, Labels, URL.

## Create an issue

```bash
jira issue create --project PROJ --type Story \
  --summary "Add user dashboard" \
  --description "Dashboard showing recent activity" \
  --assignee john.doe --priority Medium
```

### Options

| Flag | Required | Description |
|------|----------|-------------|
| `--project <key>` | Yes | Project key |
| `--type <type>` | Yes | Issue type (`Bug`, `Story`, `Task`, etc.) |
| `--summary <text>` | Yes | Issue summary |
| `--description <text>` | No | Description (mutually exclusive with `--description-file`) |
| `--description-file <path>` | No | Read description from file |
| `--assignee <user>` | No | Assignee username |
| `--priority <level>` | No | Priority level |

On success: prints the created issue key and browse URL.

### Multi-line descriptions

```bash
jira issue create --project PROJ --type Bug \
  --summary "Login broken" \
  --description-file ./bug-report.md
```

## Edit an issue

```bash
jira issue edit PROJ-123 --summary "Updated title" --priority High
```

### Options

| Flag | Description |
|------|-------------|
| `--summary <text>` | New summary |
| `--description <text>` | New description |
| `--description-file <path>` | Read description from file |
| `--assignee <user>` | New assignee |
| `--priority <level>` | New priority |

At least one flag required. Only sends changed fields. Reports "No changes made" if values match.

## Delete an issue

```bash
jira issue delete PROJ-123 --force
```

`--force` / `-f` is required. Displays warning with issue key, summary, and type before deleting.

---

## Comments

### Add a comment

```bash
# Inline text
jira issue comment add PROJ-123 "This is fixed in v2.1"

# From file
jira issue comment add PROJ-123 --file ./review-notes.md

# Internal/private comment
jira issue comment add PROJ-123 "Internal note" --internal
```

| Flag | Description |
|------|-------------|
| `--file <path>` | Read comment from file (mutually exclusive with text arg) |
| `--internal` | Mark as internal (visible to Administrators only) |

### List comments

```bash
jira issue comment list PROJ-123
jira issue comment list PROJ-123 --format json
```

| Flag | Description | Default |
|------|-------------|---------|
| `--format <fmt>` | `table` or `json` | `table` |

Table output: ID, Author, Body (truncated 150 chars), Created, Updated.

### Edit a comment

```bash
jira issue comment edit 12345 "Updated comment text"
jira issue comment edit 12345 --file ./updated.md
```

### Delete a comment

```bash
jira issue comment delete 12345 --force
```

`--force` / `-f` is required.

## Common workflows

```bash
# Daily standup — my in-progress work
jira issue list --assignee currentUser --status "In Progress"

# Bug triage
jira issue list --project PROJ --type Bug --status Open --limit 50

# CI/CD — create issue on build failure
jira issue create --project INFRA --type Bug \
  --summary "Build failed: $CI_JOB_NAME" \
  --description "Build log: $CI_JOB_URL"

# Export issue to markdown for documentation
jira issue view PROJ-123 --output ./docs/issue-PROJ-123.md
```
