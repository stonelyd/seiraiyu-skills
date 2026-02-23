---
name: neon
description: >
  Manage Neon serverless Postgres projects, branches, databases, and roles.
  Use when the user needs to create or manage Neon projects, branch databases,
  get connection strings, or perform any Neon database administration from the CLI.
allowed-tools: Bash(neon:*)
---

# Neon CLI

Manage Neon serverless Postgres from the terminal. Branching, connection strings, projects, databases, roles.

## Quick start

```bash
# Authenticate
neon auth

# Set project context (avoids --project-id on every command)
neon set-context --project-id <project-id>

# Get connection string for default branch
neon cs

# Create a branch
neon branches create --name feature/my-branch

# List databases
neon databases list

# Schema diff between branches
neon branches schema-diff main feature/my-branch
```

## Commands

| Command | Purpose |
|---------|---------|
| `neon auth` | Authenticate (alias: `login`). |
| `neon me` | Show current user info. |
| `neon set-context` | Set default project/org. `--project-id`, `--org-id`. |
| `neon projects list` | List projects. `--org-id`. |
| `neon projects create` | Create project. `--name`, `--region-id`, `--cu`, `--database`, `--set-context`. |
| `neon projects update <id>` | Update project. `--name`, `--cu`. |
| `neon projects delete <id>` | Delete a project. |
| `neon projects get <id>` | Get project details. |
| `neon projects recover <id>` | Recover deleted project (within grace period). |
| `neon branches list` | List branches. |
| `neon branches create` | Create branch. `--name`, `--parent`, `--type`, `--cu`, `--schema-only`. |
| `neon branches reset <id\|name>` | Reset branch. `--parent`, `--preserve-under-name`. |
| `neon branches restore <target> <source>` | Restore branch. Supports `@timestamp` and `@lsn`. |
| `neon branches rename <id\|name> <new>` | Rename a branch. |
| `neon branches delete <id\|name>` | Delete a branch. |
| `neon branches get <id\|name>` | Get branch details. |
| `neon branches schema-diff [base] [compare]` | Compare schemas. `--database`. |
| `neon branches set-default <id\|name>` | Set default branch. |
| `neon branches add-compute <id\|name>` | Add compute. `--type`, `--cu`. |
| `neon databases list` | List databases. `--branch`. |
| `neon databases create` | Create database. `--branch`. |
| `neon databases delete <db>` | Delete database. `--branch`. |
| `neon roles list` | List roles. `--branch`. |
| `neon roles create` | Create role. `--branch`. |
| `neon roles delete <role>` | Delete role. `--branch`. |
| `neon cs [branch]` | Connection string. `--role-name`, `--database-name`, `--pooled`, `--psql`. |
| `neon ip-allow list` | List IP allowlist. |
| `neon ip-allow add [ips...]` | Add IPs. `--protected-only`. |
| `neon ip-allow remove [ips...]` | Remove IPs. |
| `neon ip-allow reset [ips...]` | Reset allowlist. |
| `neon operations list` | List operations. |
| `neon orgs list` | List organizations. |

## Output format

Default output is `table`. Use `--output json` or `--output yaml` for machine-readable output.

```bash
# JSON output (recommended for agent use)
neon projects list --output json

# Table output (human-readable, default)
neon branches list
```

**Important:** Table format omits some fields. Always use `--output json` when you need complete data.

## Global options

All commands accept: `--output` (json/yaml/table), `--api-key`, `--project-id`, `--config-dir`, `--no-color`, `--no-analytics`.

## Context

Use `neon set-context --project-id <id>` to avoid passing `--project-id` on every command. Context is stored in `~/.config/neonctl/context.json` by default.

## Auth precedence

1. `--api-key` flag (highest)
2. `NEON_API_KEY` env var
3. `~/.config/neonctl/credentials.json` (from `neon auth`)

## Detailed guides

- **Projects** — [references/projects.md](references/projects.md)
- **Branches** — [references/branches.md](references/branches.md)
- **Databases & Roles** — [references/databases-roles.md](references/databases-roles.md)
