# Managing Projects

## List projects

```bash
neon projects list --output json
```

### Options

- `--org-id` — Filter by organization
- `--recoverable-only` — Show only deleted projects within grace period

## Create a project

```bash
neon projects create --name "my-app" --region-id aws-us-east-2 --set-context
```

### Options

- `--name` — Project name
- `--region-id` — Region: `aws-us-west-2`, `aws-us-east-1`, `aws-us-east-2`, `aws-eu-central-1`, `aws-ap-southeast-1`, `aws-ap-southeast-2`, `azure-eastus2`
- `--org-id` — Organization ID
- `--database` — Database name (default: `neondb`)
- `--role` — Role name (default: `{database_name}_owner`)
- `--cu` — Compute Units, fixed `"2"` or range `"0.5-3"`
- `--set-context` — Set as current project context after creation
- `--psql` — Connect via psql after creation
- `--block-public-connections` — Disallow public internet connections
- `--block-vpc-connections` — Disallow VPC connections

## Get project details

```bash
neon projects get <project-id> --output json
```

## Update a project

```bash
# Rename
neon projects update <project-id> --name "new-name"

# Change compute
neon projects update <project-id> --cu "0.5-4"
```

### Options

- `--name` — New project name
- `--cu` — Compute Units
- `--block-public-connections` / `--block-vpc-connections` — Toggle access

## Delete a project

```bash
neon projects delete <project-id>
```

## Recover a deleted project

```bash
# List recoverable projects first
neon projects list --recoverable-only --output json

# Recover
neon projects recover <project-id>
```

Projects can be recovered within the deletion grace period.

## Examples

```bash
# Create project in EU with custom compute
neon projects create --name "eu-app" --region-id aws-eu-central-1 --cu "1-4" --set-context

# List all projects as JSON
neon projects list --output json

# List org projects
neon projects list --org-id org-abc123 --output json
```
