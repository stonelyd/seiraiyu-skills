# Obsidian CLI — advanced commands

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
