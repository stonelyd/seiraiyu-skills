---
name: confluence
description: >
  Manage Confluence pages, spaces, and content from the CLI.
  Use when the user needs to read, create, update, search, or export Confluence pages,
  manage attachments, handle comments, or automate Confluence workflows from the terminal.
allowed-tools: Bash(confluence:*)
---

# Confluence CLI

Read, create, update, search, and export Confluence pages from the terminal. Supports Cloud and Data Center instances.

## Quick start

```bash
# Initialize (interactive)
confluence init

# Or non-interactive
confluence init \
  --domain company.atlassian.net \
  --api-path "/wiki/rest/api" \
  --auth-type basic \
  --email user@company.com \
  --token YOUR_TOKEN

# Search for pages
confluence search "API documentation" --limit 10

# Read a page
confluence read 123456789 --format markdown

# Create a page
confluence create "Meeting Notes" MYSPACE \
  --file ./notes.md --format markdown

# Export with attachments
confluence export 123456789 --dest ./backup --format markdown
```

## Commands

| Command | Purpose |
|---------|---------|
| `confluence init` | Configure instance. `--domain`, `--api-path`, `--auth-type`, `--email`, `--token`. |
| `confluence spaces` | List all spaces. |
| `confluence search <query>` | Search pages. `--limit`, `--cql` (raw CQL mode). |
| `confluence find <title>` | Find page by title. `--space <key>`. |
| `confluence info <pageId>` | Page metadata (title, ID, type, status, space). |
| `confluence read <pageId>` | Read page content. `--format` (text/html/markdown). |
| `confluence create <title> <space>` | Create page. `--file`, `--content`, `--format` (storage/html/markdown). |
| `confluence create-child <title> <parentId>` | Create child page. `--file`, `--content`, `--format`. |
| `confluence update <pageId>` | Update page. `--title`, `--file`, `--content`, `--format`. |
| `confluence delete <pageId>` | Delete page. `--yes` to skip confirmation. |
| `confluence edit <pageId>` | Export for local editing. `--output <file>`. |
| `confluence move <pageId> <newParentId>` | Move page (same space only). `--title` to rename. |
| `confluence children <pageId>` | List children. `--recursive`, `--max-depth`, `--format` (list/tree/json), `--show-url`, `--show-id`. |
| `confluence copy-tree <src> <target> [title]` | Copy page hierarchy. `--max-depth`, `--exclude`, `--dry-run`, `--delay-ms`. |
| `confluence export <pageId>` | Export page + attachments. `--format`, `--dest`, `--referenced-only`, `--skip-attachments`. |
| `confluence attachments <pageId>` | List/download attachments. `--pattern`, `--download`, `--dest`, `--limit`. |
| `confluence attachment-upload <pageId>` | Upload files. `--file` (repeatable), `--comment`, `--replace`, `--minor-edit`. |
| `confluence attachment-delete <pageId> <attachId>` | Delete attachment. `--yes`. |
| `confluence comments <pageId>` | List comments. `--format` (text/markdown/json), `--location`, `--depth`, `--all`. |
| `confluence comment <pageId>` | Add comment. `--content`, `--file`, `--format`, `--parent` (reply), `--location`. |
| `confluence comment-delete <commentId>` | Delete comment. `--yes`. |
| `confluence property-list <pageId>` | List content properties. `--format` (text/json), `--all`. |
| `confluence property-get <pageId> <key>` | Get property. `--format`. |
| `confluence property-set <pageId> <key>` | Set property. `--value <json>`, `--file`. |
| `confluence property-delete <pageId> <key>` | Delete property. `--yes`. |
| `confluence stats` | Show CLI usage statistics. |

## Authentication

**Basic auth** (Cloud): `confluence init --auth-type basic --email <email> --token <token>`

**Bearer auth** (Data Center PAT): `confluence init --auth-type bearer --token <pat>`

**Environment variables** (override stored config):

| Variable | Alias | Description |
|----------|-------|-------------|
| `CONFLUENCE_DOMAIN` | `CONFLUENCE_HOST` | Instance domain |
| `CONFLUENCE_API_TOKEN` | `CONFLUENCE_PASSWORD` | API token |
| `CONFLUENCE_EMAIL` | `CONFLUENCE_USERNAME` | Email/username |
| `CONFLUENCE_AUTH_TYPE` | — | `basic` or `bearer` |
| `CONFLUENCE_API_PATH` | — | REST API path override |

**API path defaults**: Cloud (`*.atlassian.net`) → `/wiki/rest/api`, self-hosted → `/rest/api`.

**Config location**: `~/.confluence-cli/config.json`

## Page ID

All `<pageId>` arguments accept:
- Numeric ID: `123456789`
- URL with `pageId=` query: `https://domain.atlassian.net/wiki/viewpage.action?pageId=123`
- URL with `/pages/{id}` path: `https://domain.atlassian.net/wiki/spaces/SPACE/pages/123/Title`

## Content formats

| Direction | Formats |
|-----------|---------|
| **Read/Export** | `text` (default for read), `html`, `markdown` (default for export) |
| **Write** (create/update/comment) | `storage` (default), `html`, `markdown` |

## Detailed guides

- **Pages** — [references/pages.md](references/pages.md)
- **Attachments & Export** — [references/attachments.md](references/attachments.md)
- **Comments & Properties** — [references/comments-properties.md](references/comments-properties.md)
