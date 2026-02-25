# Managing Pages

## Search for pages

```bash
# Text search
confluence search "deployment guide" --limit 20

# Raw CQL query
confluence search "type=page AND space=DEV AND title~'API'" --cql
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--limit <n>` | Max results | `10` |
| `--cql` | Treat query as raw CQL | `false` |

## Find by title

```bash
confluence find "Architecture Overview" --space TEAM
```

| Flag | Description |
|------|-------------|
| `--space <key>` | Limit search to specific space |

Returns: id, title, space, URL. Throws error if not found.

## Read a page

```bash
confluence read 123456789 --format markdown
```

| Flag | Description | Default |
|------|-------------|---------|
| `--format <fmt>` | `text`, `html`, `markdown` | `text` |

## Get page info

```bash
confluence info 123456789
```

Returns: title, ID, type, status, space metadata.

## Create a page

```bash
# From file (markdown)
confluence create "Sprint Retro" TEAM --file ./retro.md --format markdown

# Inline content
confluence create "Quick Note" TEAM --content "<p>Hello</p>" --format storage
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--file <path>` | Read content from file | — |
| `--content <text>` | Inline content | — |
| `--format <fmt>` | `storage`, `html`, `markdown` | `storage` |

## Create a child page

```bash
confluence create-child "Sub-section" 123456789 --file ./content.md --format markdown
```

Same options as `create`.

## Update a page

```bash
# Update content
confluence update 123456789 --file ./updated.md --format markdown

# Rename only (preserves content)
confluence update 123456789 --title "New Title"

# Both
confluence update 123456789 --title "New Title" --content "New body" --format markdown
```

### Options

| Flag | Description |
|------|-------------|
| `--title <text>` | New page title |
| `--file <path>` | Read content from file |
| `--content <text>` | Inline content |
| `--format <fmt>` | `storage`, `html`, `markdown` |

Auto-fetches current version and increments. Title-only updates preserve existing content.

## Delete a page

```bash
confluence delete 123456789 --yes
```

`--yes` / `-y` skips the confirmation prompt.

## Edit (roundtrip workflow)

```bash
# 1. Export page in storage format
confluence edit 123456789 --output ./page.xml

# 2. Edit locally with your editor
vim ./page.xml

# 3. Re-upload
confluence update 123456789 --file ./page.xml --format storage
```

## Move a page

```bash
confluence move 123456789 987654321
confluence move 123456789 987654321 --title "Renamed During Move"
```

**Constraint**: Pages can only be moved within the same space.

| Flag | Description |
|------|-------------|
| `--title <text>` | Rename page during move |

## List children

```bash
confluence children 123456789 --recursive --format tree --show-id
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--recursive` / `-r` | List all descendants | `false` |
| `--max-depth <n>` | Max depth for recursive listing | `10` |
| `--format <fmt>` | `list`, `tree`, `json` | `list` |
| `--show-url` | Show page URLs | `false` |
| `--show-id` | Show page IDs | `false` |

## Copy a page tree

```bash
# Preview first
confluence copy-tree 123456789 987654321 "Backup" --dry-run

# Execute
confluence copy-tree 123456789 987654321 "Backup" \
  --exclude "temp*,*draft*" --delay-ms 150
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--max-depth <n>` | Max depth to copy | `10` |
| `--exclude <patterns>` | Comma-separated glob patterns to exclude | — |
| `--delay-ms <ms>` | Delay between page creations (rate limiting) | `100` |
| `--copy-suffix <text>` | Suffix for root page title | `" (Copy)"` |
| `--dry-run` / `-n` | Preview without creating | `false` |
| `--fail-on-error` | Exit on any page failure | `false` |
| `--quiet` / `-q` | Suppress progress output | `false` |

Preserves parent-child hierarchy. Root page gets suffix; children keep original titles.

## List spaces

```bash
confluence spaces
```

Returns: key, name, type for all spaces.

## Common workflows

```bash
# Discover → Read → Export
confluence spaces
confluence search "architecture" --limit 5
confluence read 123456789 --format markdown
confluence export 123456789 --dest ./docs --format markdown

# Create documentation tree
confluence create "Project Docs" TEAM --file ./overview.md --format markdown
# (get returned page ID, e.g. 111)
confluence create-child "Setup Guide" 111 --file ./setup.md --format markdown
confluence create-child "API Reference" 111 --file ./api.md --format markdown

# Backup a page tree
confluence copy-tree 123456789 987654321 "Backup $(date +%Y-%m-%d)" --dry-run
confluence copy-tree 123456789 987654321 "Backup $(date +%Y-%m-%d)"
```
