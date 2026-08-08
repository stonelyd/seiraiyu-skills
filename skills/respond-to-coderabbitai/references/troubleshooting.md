# Troubleshooting & impasse escalation

## API and query issues

- **GraphQL syntax errors** — don't use heredocs or multi-line strings in
  `-f query=`; keep the query a single argument.
- **"Expected VAR_SIGN" errors** — variable names in the query must match the
  `-F` parameters exactly (`$owner`, `$name`, `$pr`).
- **404 on replies endpoint** — the path must include `/pulls/${PR}/comments/${CID}/replies`
  and the token needs write scope.
- **GraphQL query fails** — check `gh auth status -t` and repo access; reduce
  `first:100` if hitting limits.
- **Empty/null bodies from GraphQL** — expected; fetch full text via REST
  (`gh api repos/${REPO}/pulls/${PR}/comments --paginate`).

## Comment filtering pitfalls

- **All REST comments have `in_reply_to_id` set** — use GraphQL `reviewThreads`,
  which returns only top-level threads.
- **GraphQL shows 0 threads but REST shows many** — GraphQL is correct; REST
  includes threaded replies.
- **IDs don't match** — GraphQL `databaseId` = REST `id`; GraphQL's own `id` is a
  global node ID.

## Code issues

- **Outdated comments** — if the line moved, fix at the new location and reply in
  the same thread explaining the mapping.
- **Other bots** — fine to fix and reply; optionally filter authors with
  `| select(.author.login != "coderabbitai[bot]")`.

## Impasse escalation

Escalate when a thread can't be resolved after 2-3 exchanges and one of these
holds: the change needs codebase-wide refactoring, there's a legitimate technical
disagreement, it's out of scope for this PR, or it's an improvement rather than a
required fix. Don't block a PR indefinitely on nice-to-haves — but always
acknowledge the suggestion's validity and create traceability.

1. **Reply acknowledging and deferring**, asking CodeRabbit to open an issue:

```bash
gh api -H "Accept: application/vnd.github+json" \
  repos/${REPO}/pulls/${PR}/comments/${CID}/replies \
  -f body="Acknowledged. This is a valid improvement but requires codebase-wide refactoring that's out of scope for this PR.

@coderabbitai please open a GitHub issue to track this improvement so it can be properly addressed in a dedicated PR.

Marking as resolved for this PR."
```

2. **Resolve the thread** (if permitted) with the GraphQL mutation in
   [discovery.md](discovery.md).

Examples: "Use AuthenticatedRequest interface" (requires refactoring all routes) →
acknowledge, explain the middleware already guarantees safety, defer to issue.
"Add comprehensive error handling" (large scope) → acknowledge value, note PR
scope, request follow-up issue.
