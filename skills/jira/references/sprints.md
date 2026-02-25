# Managing Sprints & Boards

## List boards

```bash
jira sprint boards
```

Shows: Name, Type (scrum/kanban), ID, associated project.

**Important:** Most sprint commands require a board ID. If only one board exists, it's auto-selected. Otherwise, use `--board <id>`.

## List sprints

```bash
jira sprint list --board 42
```

### Options

| Flag | Description |
|------|-------------|
| `-b, --board <id>` | Board ID (required when multiple boards exist) |
| `-a, --active` | Show only active sprints |
| `--state <state>` | Filter by state: `active`, `future`, `closed` |

Output: table with ID, Name, State (color-coded), Start Date, End Date. Shows sprint summary by state count when no filter applied.

## Active sprint

```bash
jira sprint active --board 42
```

Shortcut for `jira sprint list --active`. Shows active sprint(s) for the board.

## Common workflows

```bash
# Find your board ID
jira sprint boards

# Check active sprint
jira sprint active --board 42

# View all upcoming sprints
jira sprint list --board 42 --state future

# Sprint planning — list active sprint issues
jira sprint active --board 42
jira issue list --assignee currentUser --status "In Progress"

# Review closed sprints
jira sprint list --board 42 --state closed
```
