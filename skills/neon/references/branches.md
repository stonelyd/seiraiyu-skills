# Managing Branches

Neon branches are copy-on-write clones of your database. They're instant, cheap, and the core workflow for dev/test/preview environments.

## List branches

```bash
neon branches list --output json
```

## Create a branch

```bash
neon branches create --name dev/feature-auth
```

### Options

- `--name` — Branch name
- `--parent` — Parent branch name, id, or point-in-time (`main@2024-01-01T00:00:00Z`, `main@0/1234`)
- `--compute` / `--no-compute` — Create with or without compute (default: with)
- `--type` — Compute type: `read_write` (default) or `read_only`
- `--cu` — Compute Units
- `--suspend-timeout` — Seconds before auto-suspend (0=global default, -1=never, max=604800)
- `--schema-only` — Branch schema without data
- `--expires-at` — Expiration date (RFC3339, e.g. `2024-12-31T23:59:59Z`)
- `--psql` — Connect via psql after creation

### Examples

```bash
# Branch from main
neon branches create --name staging

# Branch from specific point in time
neon branches create --name rollback --parent "main@2024-06-15T12:00:00Z"

# Schema-only branch (no data)
neon branches create --name test-schema --schema-only

# Branch with expiration
neon branches create --name temp-review --expires-at "2024-12-31T23:59:59Z"
```

## Get branch details

```bash
neon branches get dev/feature-auth --output json
```

## Rename a branch

```bash
neon branches rename old-name new-name
```

## Set default branch

```bash
neon branches set-default staging
```

## Set/remove expiration

```bash
# Set expiration
neon branches set-expiration temp-branch --expires-at "2024-12-31T23:59:59Z"

# Remove expiration (omit --expires-at)
neon branches set-expiration temp-branch
```

## Add compute to a branch

```bash
neon branches add-compute my-branch --type read_only --cu "0.5-2"
```

### Options

- `--type` — `read_only` (default) or `read_write`
- `--cu` — Compute Units
- `--name` — Optional compute name

## Reset a branch

Reset a branch to match its parent or a specific state.

```bash
# Reset to parent
neon branches reset dev/feature --parent
```

### Options

- `--parent` — Reset to parent branch state
- `--preserve-under-name` — Save current state under a new branch name before reset

## Restore a branch

Restore a branch from another branch or a point in time.

```bash
# Restore from another branch
neon branches restore target-branch source-branch

# Restore to a point in time
neon branches restore main "main@2024-06-15T12:00:00Z"

# Restore to a specific LSN
neon branches restore main "main@0/1A2B3C"

# Restore from own history
neon branches restore my-branch "^self@2024-06-15T12:00:00Z"

# Restore from parent
neon branches restore my-branch "^parent"
```

### Options

- `--preserve-under-name` — Save current state before restoring

### Special source syntax

- `branch@timestamp` — Point-in-time (RFC3339)
- `branch@lsn` — Specific LSN (e.g. `0/1A2B3C`)
- `^self@timestamp` or `^self@lsn` — From own history
- `^parent` — From parent branch head

## Schema diff

Compare schemas between branches.

```bash
# Compare two branches
neon branches schema-diff main dev/feature

# Compare branch to a point in time
neon branches schema-diff main "dev/feature@2024-06-15T12:00:00Z"

# Compare to parent
neon branches schema-diff main "^parent"

# Specify database
neon branches schema-diff main dev/feature --database mydb
```

### Options

- `--database`, `--db` — Database to compare (defaults to project default)

Alias: `neon branches sd`

## Delete a branch

```bash
neon branches delete dev/feature-auth
```

## Connection string for a branch

```bash
# Default connection string
neon cs my-branch

# Pooled connection (for serverless)
neon cs my-branch --pooled

# For Prisma
neon cs my-branch --prisma

# Point-in-time connection
neon cs "main@2024-01-01T00:00:00Z"

# Connect directly via psql
neon cs my-branch --psql
```

### Options

- `--role-name` — Role for connection
- `--database-name` — Database for connection
- `--pooled` — Use connection pooling (recommended for serverless)
- `--prisma` — Format for Prisma
- `--psql` — Open psql directly
- `--endpoint-type` — `read_write` or `read_only`
- `--extended` — Show extended connection details
- `--ssl` — SSL mode: `require` (default), `verify-ca`, `verify-full`, `omit`
