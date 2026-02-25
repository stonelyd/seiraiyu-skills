# Comments & Content Properties

## List comments

```bash
confluence comments 123456789
confluence comments 123456789 --format json --all
confluence comments 123456789 --location footer,resolved --depth all
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--format <fmt>` / `-f` | `text`, `markdown`, `json` | `text` |
| `--limit <n>` / `-l` | Max comments | `25` |
| `--start <n>` | Pagination start index | `0` |
| `--location <loc>` | Filter: `inline`, `footer`, `resolved` (comma-separated) | — |
| `--depth <depth>` | `""` root only, `"all"` for all | — |
| `--all` | Fetch all comments (ignores pagination) | `false` |

## Add a comment

```bash
# Footer comment (default)
confluence comment 123456789 --content "Looks good!" --format markdown

# Reply to another comment
confluence comment 123456789 --parent 998877 --content "Agreed"

# From file
confluence comment 123456789 --file ./review.md --format markdown
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--content <text>` / `-c` | Comment text | — |
| `--file <path>` / `-f` | Read content from file | — |
| `--format <fmt>` | `storage`, `html`, `markdown` | `storage` |
| `--parent <commentId>` | Reply to specific comment | — |
| `--location <loc>` | `inline` or `footer` | `footer` |

**Note**: Inline comments require editor-generated metadata not available via public REST API. Footer comments are fully supported.

## Delete a comment

```bash
confluence comment-delete 998877 --yes
```

---

## Content Properties

Key-value metadata attached to pages. Values are JSON.

### List properties

```bash
confluence property-list 123456789
confluence property-list 123456789 --format json --all
```

| Flag | Description | Default |
|------|-------------|---------|
| `--format <fmt>` / `-f` | `text`, `json` | `text` |
| `--limit <n>` / `-l` | Max properties | `25` |
| `--start <n>` | Pagination start | `0` |
| `--all` | Fetch all properties | `false` |

### Get a property

```bash
confluence property-get 123456789 my-key --format json
```

### Set a property

```bash
# Inline JSON value
confluence property-set 123456789 my-key --value '{"color":"#ff0000","enabled":true}'

# From file
confluence property-set 123456789 my-key --file ./config.json
```

| Flag | Description |
|------|-------------|
| `--value <json>` / `-v` | Property value as JSON |
| `--file <path>` | Read value from JSON file |
| `--format <fmt>` / `-f` | Output format: `text`, `json` |

Auto-handles versioning (creates new or increments existing).

### Delete a property

```bash
confluence property-delete 123456789 my-key --yes
```

## Common workflows

```bash
# Review and respond to comments
confluence comments 123456789 --format json --all
confluence comment 123456789 --parent 998877 --content "Done" --format markdown

# Store metadata on pages
confluence property-set 123456789 review-status --value '{"status":"approved","reviewer":"jane"}'
confluence property-get 123456789 review-status --format json

# Bulk check properties
confluence property-list 123456789 --format json --all
```
