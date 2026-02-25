# Managing Projects

## List projects

```bash
jira project list
```

### Options

| Flag | Description |
|------|-------------|
| `--type <type>` | Filter by project type (substring match on `projectTypeKey`) |
| `--category <cat>` | Filter by category name (substring match) |

Output: table with Key, Name, Type, Lead columns.

## View project details

```bash
jira project view PROJ
```

Displays: Key, Name, Type, Lead, Description, Category, Components (with descriptions and leads), Versions (with release status and dates), browse URL.

## List components

```bash
jira project components PROJ
```

Shows name, description, and component lead for each component.

## List versions

```bash
jira project versions PROJ
```

Shows name, status (Released/Archived/Unreleased), description, and release date for each version.

## Examples

```bash
# Find all software projects
jira project list --type software

# View project details before creating issues
jira project view PROJ

# Check available components for a project
jira project components PROJ

# Check release versions
jira project versions PROJ
```
