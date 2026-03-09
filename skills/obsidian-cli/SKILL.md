---
name: obsidian-cli
description: >
  Interact with Obsidian vaults via the command line - creating, reading, searching, moving notes,
  managing properties, tasks, plugins, sync, and daily notes. Use when the user needs vault operations
  that benefit from Obsidian's link resolution, templates, or plugin features rather than direct file I/O.
allowed-tools: Bash(obsidian:*)
---

# Obsidian CLI

Control Obsidian from the terminal. The CLI communicates with the running Obsidian desktop app via IPC.

**Announce:** "Using obsidian-cli to [action]."

**Prerequisite:** Obsidian desktop must be running. The CLI launches the Electron app per invocation, so commands take a few seconds.

## When to use

**Use obsidian-cli when:**
- Operating on vault content that benefits from Obsidian's link resolution (wikilinks, backlinks, templates)
- Managing properties/frontmatter (type-aware: text, list, number, checkbox, date, tags)
- Working with tasks, search, sync, or plugins
- Need Obsidian-specific features (templates, daily notes, bookmarks, bases)

**Use direct file I/O (Read/Write/Edit) when:**
- Simple file reads/writes where link resolution isn't needed
- Bulk operations where CLI per-command overhead is too slow
- Obsidian desktop isn't running

## Quick start

```bash
# Read a note
obsidian read file="Note Name"

# Create a note
obsidian create name="Title" path=Projects/

# Create with template
obsidian create name="Title" template="Template Name"

# Search vault
obsidian search query="term"

# Today's daily note
obsidian daily

# Append to daily
obsidian daily:append content="text"

# List files in folder
obsidian files folder=Projects/

# Vault info
obsidian vault
```

## Syntax

```
obsidian <command> [options] [vault=<name>]
```

- `file=<name>` resolves by name (like wikilinks)
- `path=<path>` is exact (folder/note.md)
- Most commands default to the active file when file/path is omitted
- Quote values with spaces: `name="My Note"`
- Use `\n` for newline, `\t` for tab in content values
- Output formats: `format=json|tsv|csv|md|paths|tree|yaml` (varies by command)

## Commands

| Task | Command |
|------|---------|
| Read note | `obsidian read file="Note Name"` |
| Create note | `obsidian create name="Title" path=Projects/` |
| Create with template | `obsidian create name="Title" template="Template Name"` |
| Append to note | `obsidian append file="Note" content="text"` |
| Prepend to note | `obsidian prepend file="Note" content="text"` |
| Search vault | `obsidian search query="term"` |
| Search with context | `obsidian search:context query="term"` |
| Move/rename | `obsidian move file="Note" to=Archive/` |
| Delete (trash) | `obsidian delete file="Note"` |
| Delete (permanent) | `obsidian delete file="Note" permanent` |
| Open in Obsidian | `obsidian open file="Note"` |
| Daily note | `obsidian daily` |
| Read daily | `obsidian daily:read` |
| Append to daily | `obsidian daily:append content="text"` |
| List files | `obsidian files` |
| List files in folder | `obsidian files folder=Projects/` |
| File count | `obsidian files total` |
| Folder tree | `obsidian folders format=tree` |
| Vault info | `obsidian vault` |

## Properties (frontmatter)

```bash
obsidian properties file="Note"                          # Read all
obsidian property:read name=status file="Note"           # Read one
obsidian property:set name=status value=active file="Note"  # Set
obsidian property:set name=tags value="pkm,obsidian" type=tags file="Note"
obsidian property:remove name=draft file="Note"          # Remove
```

Supported types: `text`, `list`, `number`, `checkbox`, `date`, `datetime`, `tags`

## Tags & links

```bash
obsidian tags                           # All vault tags
obsidian tags sort=count                # By frequency
obsidian tag name=pkm                   # Notes with tag
obsidian links file="Note"              # Outgoing links
obsidian backlinks file="Note"          # Incoming links
obsidian unresolved                     # Broken wikilinks
obsidian orphans                        # No incoming links
obsidian deadends                       # No outgoing links
```

## Tasks

```bash
obsidian tasks                          # All tasks
obsidian tasks todo                     # Incomplete only
obsidian tasks done                     # Completed only
obsidian tasks file="Project"           # Tasks in file
obsidian tasks daily                    # Today's tasks
obsidian tasks format=json              # Structured output
obsidian task ref="path:line" toggle    # Toggle status
obsidian task ref="path:line" done      # Mark done
```

## Search

```bash
obsidian search query="term"                    # Full-text
obsidian search query="term" path=Projects/     # In folder
obsidian search query="term" limit=10 case      # Case-sensitive, limited
obsidian search:context query="term"            # With line context
obsidian search:open query="term"               # Open in Obsidian UI
obsidian search query="term" format=json        # JSON output
```

## Templates

```bash
obsidian templates                              # List available
obsidian template:read name="Template"          # Read content
obsidian template:read name="Template" resolve title="My Note"  # With variables
obsidian template:insert name="Template"        # Insert into active file
obsidian create name="Note" template="Template" # Create with template
```

## Sync & history

```bash
obsidian sync:status                    # Sync state
obsidian sync on|off                    # Resume/pause
obsidian sync:history file="Note"       # Sync versions
obsidian sync:read file="Note" version=3
obsidian sync:restore file="Note" version=3
obsidian history file="Note"            # Local versions
obsidian history:read file="Note" version=2
obsidian history:restore file="Note" version=2
```

## Plugins

```bash
obsidian plugins                        # List installed
obsidian plugins:enabled                # List enabled
obsidian plugin id=dataview             # Plugin info
obsidian plugin:enable id=dataview
obsidian plugin:disable id=calendar
obsidian plugin:install id=plugin-id enable
obsidian plugin:uninstall id=plugin-id
obsidian plugin:reload id=my-plugin     # Dev reload
```

## Bases (databases)

```bash
obsidian bases                                  # List bases
obsidian base:views file="Base"                 # List views
obsidian base:query file="Base" format=json     # Query base
obsidian base:query file="Base" view="Active"   # Query specific view
obsidian base:create file="Base" name="Item"    # Create item
```

## Bookmarks

```bash
obsidian bookmarks                              # List all
obsidian bookmark file="Note"                   # Bookmark file
obsidian bookmark file="Note" subpath="Heading" # Bookmark heading
obsidian bookmark search="query"                # Bookmark search
obsidian bookmark url="https://..."             # Bookmark URL
```

## Developer commands

```bash
obsidian eval code="app.vault.getFiles().length"
obsidian dev:screenshot path=~/screenshot.png
obsidian dev:console limit=50
obsidian dev:console level=error
obsidian dev:errors
obsidian dev:dom selector=".workspace-leaf" total
obsidian dev:css selector=".markdown-preview-view"
```

## Utility commands

```bash
obsidian version                        # App version
obsidian reload                         # Reload vault
obsidian restart                        # Restart app
obsidian vaults                         # List known vaults
obsidian recents                        # Recently opened files
obsidian random:read                    # Read random note
obsidian wordcount file="Note"          # Word/char count
obsidian outline file="Note"            # Heading structure
obsidian commands                       # All command IDs
obsidian command id=command-id          # Execute command
```

## Tips

- **Performance:** Each CLI call boots Electron (~3-5s). Batch reads with direct file I/O when speed matters.
- **Multi-vault:** Use `vault=<name>` to target a specific vault.
- **Piping:** Use `format=json` for structured output: `obsidian search query="term" format=json | jq '.[]'`
- **Help per command:** `obsidian help <command>` for detailed options.
- **Wikilink resolution:** `file=` resolves like `[[wikilinks]]` — use when you know the note name but not the path.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| CLI hangs | Ensure Obsidian desktop is running |
| `file=` not found | Use exact note name (no .md extension) or switch to `path=` |
| Slow bulk operations | Use direct file I/O for batch work, CLI for Obsidian-specific features |
| Missing output | Add `format=json` or `format=text` for explicit output |
| Spaces in values | Quote: `name="My Note"` |
