# Managing Databases & Roles

## Databases

### List databases

```bash
neon databases list --output json
neon databases list --branch dev/feature --output json
```

### Create a database

```bash
neon databases create --branch main
```

### Options

- `--branch` — Target branch name or ID (defaults to default branch)
- `--project-id` — Project ID (uses context if set)

### Delete a database

```bash
neon databases delete mydb --branch main
```

## Roles

### List roles

```bash
neon roles list --output json
neon roles list --branch dev/feature --output json
```

### Create a role

```bash
neon roles create --branch main
```

### Options

- `--branch` — Target branch name or ID
- `--project-id` — Project ID (uses context if set)

### Delete a role

```bash
neon roles delete myrole --branch main
```

## IP Allow List

Restrict project access by IP address.

### List allowed IPs

```bash
neon ip-allow list --output json
```

### Add IPs

```bash
neon ip-allow add 203.0.113.0/24 198.51.100.1
```

#### Options

- `--protected-only` — Apply allowlist only to protected branches

### Remove IPs

```bash
neon ip-allow remove 203.0.113.0/24
```

### Reset allowlist

```bash
# Replace entire list
neon ip-allow reset 10.0.0.0/8 172.16.0.0/12
```

## Operations

### List operations

```bash
neon operations list --output json
```

Shows recent operations (creates, deletes, computes) for the current project.

## Organizations

### List organizations

```bash
neon orgs list --output json
```
