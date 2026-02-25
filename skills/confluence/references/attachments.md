# Attachments & Export

## List attachments

```bash
confluence attachments 123456789
confluence attachments 123456789 --pattern "*.pdf" --limit 10
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--limit <n>` / `-l` | Max attachments to fetch | all |
| `--pattern <glob>` / `-p` | Filter by filename (e.g., `"*.png"`) | — |
| `--download` / `-d` | Download matching attachments | `false` |
| `--dest <dir>` | Download destination directory | `.` |

## Download attachments

```bash
# Download all PNGs
confluence attachments 123456789 --pattern "*.png" --download --dest ./images

# Download all attachments
confluence attachments 123456789 --download --dest ./all-files
```

## Upload attachments

```bash
# Single file
confluence attachment-upload 123456789 --file ./report.pdf

# Multiple files with comment
confluence attachment-upload 123456789 \
  --file ./diagram.png --file ./spec.pdf \
  --comment "Updated diagrams v2"

# Replace existing attachment
confluence attachment-upload 123456789 --file ./logo.png --replace --minor-edit
```

### Options

| Flag | Description |
|------|-------------|
| `--file <path>` / `-f` | File to upload (repeatable for multiple) |
| `--comment <text>` | Comment for the attachment(s) |
| `--replace` | Replace existing attachment with same filename |
| `--minor-edit` | Mark as minor edit |

## Delete an attachment

```bash
confluence attachment-delete 123456789 998877 --yes
```

`--yes` / `-y` skips confirmation.

---

## Export

Export a page with its attachments to a local directory.

```bash
# Full export (markdown + all attachments)
confluence export 123456789 --dest ./backup --format markdown

# HTML export, only referenced attachments
confluence export 123456789 --dest ./backup --format html --referenced-only

# Content only, skip attachments
confluence export 123456789 --format markdown --skip-attachments
```

### Options

| Flag | Description | Default |
|------|-------------|---------|
| `--format <fmt>` | `html`, `text`, `markdown` | `markdown` |
| `--dest <dir>` | Base export directory | `.` |
| `--file <name>` | Content filename | `page.<ext>` |
| `--attachments-dir <name>` | Subdirectory for attachments | `attachments` |
| `--pattern <glob>` | Filter attachments by filename | — |
| `--referenced-only` | Only download attachments referenced in content | `false` |
| `--skip-attachments` | Do not download any attachments | `false` |

### Export structure

```
dest/
├── page.md              # Page content
└── attachments/         # Attachments directory
    ├── diagram.png
    └── spec.pdf
```

## Common workflows

```bash
# Backup page with all attachments
confluence export 123456789 --dest "./backup/$(date +%Y-%m-%d)" --format markdown

# Migrate attachments between pages
confluence attachments 111 --download --dest ./temp
confluence attachment-upload 222 --file ./temp/doc.pdf --file ./temp/img.png

# Clean up old attachments
confluence attachments 123456789 --pattern "*.tmp"
# Note attachment IDs from output, then:
confluence attachment-delete 123456789 <attachId> --yes
```
