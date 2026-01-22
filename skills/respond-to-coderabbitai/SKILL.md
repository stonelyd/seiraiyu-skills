---
name: Auto‑Resolve PR Review Comments
version: 0.2.0
description: >-
  Read **unresolved** pull‑request review threads, group by logical issue, implement fixes
  with atomic commits per issue, and reply to each comment with the commit that addresses it.
audience: Claude Code (Developer Agent)
intent_examples:
  - "Fix all unresolved PR review comments in PR #5, group by logical issue, create atomic commits, and reply with commit links."
  - "Address coderabbitai[bot] comments, group related fixes into atomic commits, do not push, and respond to each thread with its commit."
requirements:
  - Access to local git repo checked out on a working branch that targets the PR.
  - GitHub CLI (`gh`) authenticated with write scopes (classic: `repo`; fine‑grained: Pull Requests: read+write, Contents: read+write).
  - `jq` available for JSON processing.
config:
  env:
    REPO: "owner/repo (e.g., stonelyd/altium-design-review)"
    PR: "PR number (e.g., 5)"
    BRANCH: "Name of working branch that the PR uses"
  tools:
    - Bash
    - git
    - gh
    - jq
    - Editor
safety:
  - Never force‑push without explicit instruction.
  - Run tests/lint before committing when available.
  - Only modify files that are in the PR's branch.
---

## Purpose

Enable Claude Code to **systematically clear unresolved PR review threads**: analyze all threads, group by logical issue, implement atomic fixes, create one commit per logical issue, and **post threaded replies** linking each comment to the commit that addresses it.

**Key principle**: Create atomic commits that make sense as standalone changes. Group related comments that address the same underlying issue into a single commit for clean git history.

## Use these superpowers

* **/superpowers:write-plan – Create implementation plan**

  * Use this to analyze all comments, group by logical issue, and plan atomic commits.

Example:

```bash
/superpowers:write-plan
Goal: Clear all unresolved review threads in PR ${PR} for ${REPO}.
Approach: Analyze all comments and group by logical issue. Create atomic commits per issue.
Deliverables:
  - Grouping plan showing which comments address the same logical issue
  - One commit per logical issue (e.g., "Add archived filters" for 3 related comments)
  - Threaded replies to ALL comments in each group with the commit link
Constraints: Atomic commits that make sense standalone; preserve style; run tests before each commit.
```

---

## Operational recipe

### 1) Discover **all unresolved** review comments (GraphQL - Recommended)

> **RECOMMENDED**: Use GraphQL to get unresolved review threads directly. This is more reliable than filtering REST API results.

```bash
REPO="stonelyd/altium-design-review"  # Update with your repo
PR=5                                    # Update with your PR number

# Get all unresolved review threads with full comment details
readarray -t THREADS < <(
gh api graphql -f query='
query($owner:String!, $name:String!, $pr:Int!) {
  repository(owner:$owner, name:$name) {
    pullRequest(number:$pr) {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          comments(first: 50) {
            nodes {
              databaseId
              path
              originalPosition
              diffHunk
              isMinimized
              author { login }
            }
          }
        }
      }
    }
  }
}' \
-F owner="${REPO%%/*}" -F name="${REPO##*/}" -F pr="$PR" \
| jq -c '.data.repository.pullRequest.reviewThreads.nodes[]
| select(.isResolved==false)
| {thread_id:.id,
   top_comment_id:(.comments.nodes[0].databaseId),
   path:(.comments.nodes[0].path),
   author:(.comments.nodes[0].author.login),
   diff:(.comments.nodes[0].diffHunk)}')

# Display the threads
echo "=== Found ${#THREADS[@]} unresolved threads ==="
for i in "${!THREADS[@]}"; do
  echo -e "\n--- Thread $((i+1)) ---"
  echo "${THREADS[$i]}" | jq -r '"Comment ID: \(.top_comment_id)\nFile: \(.path)\nAuthor: \(.author)\nThread ID: \(.thread_id)"'
done
```

**Alternative: REST API for comment bodies**

> Use REST API to get full comment bodies after you have the comment IDs from GraphQL.

```bash
# Save all comments to file for processing (includes threaded replies)
gh api repos/${REPO}/pulls/${PR}/comments --paginate > /tmp/all_comments.json

# Extract specific comment details
for CID in 2466210587 2466210599 2466210608; do
  echo -e "\n=========================================="
  echo "COMMENT ID: $CID"
  cat /tmp/all_comments.json | jq -r ".[] | select(.id == $CID) | \"File: \(.path):\(.line // .original_line // \"null\")\\nAuthor: \(.user.login)\\n\\nBody:\\n\(.body)\\n\""
  echo "=========================================="
done
```

### 2) Get full comment bodies for each unresolved thread

After getting thread IDs from GraphQL, fetch full comment bodies from REST API:

```bash
# Method 1: Batch fetch all comments to file (recommended for many comments)
gh api repos/${REPO}/pulls/${PR}/comments --paginate > /tmp/all_comments.json

# Extract full body for specific comment IDs
for CID in "${COMMENT_IDS[@]}"; do
  echo -e "\n=========================================="
  echo "COMMENT ID: $CID"
  cat /tmp/all_comments.json | jq -r ".[] | select(.id == $CID) | \"File: \(.path):\(.line // \"null\")\\nAuthor: \(.user.login)\\n\\nBody:\\n\(.body)\\n\""
  echo "=========================================="
done

# Method 2: Fetch individual comment (for single/few comments)
CID=2466210587
gh api repos/${REPO}/pulls/${PR}/comments/${CID} | jq -r '"File: \(.path):\(.line // \"null\")\nAuthor: \(.user.login)\n\nBody:\n\(.body)"'
```

**Key learnings:**
- GraphQL `databaseId` matches REST API `id` field
- GraphQL doesn't include full comment body text - use REST API for that
- Always verify `in_reply_to_id` is `null` (GraphQL filters this automatically via `reviewThreads`)

### 3) Implement the fix

* Open the referenced file/section.
* Apply the **minimal** change that satisfies the reviewer's request.
* **MANDATORY**: Run all tests before committing:
  1. `npm run check` - TypeScript type checking and linting
  2. `npm run test` - Unit tests
  3. `npm run test:components` - Component tests (if changes affect UI components)
* Fix any test failures or type errors before proceeding to commit.

### 4) Create atomic commits per logical issue

**CRITICAL**: Create **one commit per logical issue/fix**. Group related comments that address the same issue into a single atomic commit.

**Commit strategy:**
- **Single issue = Single commit**: If multiple comments point to the same underlying problem, fix it in one commit
- **Different issues = Separate commits**: Keep unrelated fixes in separate commits for clean history
- **Atomic changes**: Each commit should be self-contained and make sense on its own

**Examples:**
- ✅ Good: One commit fixing "Add archived filter" that addresses 3 comments about missing archived filters in different functions
- ✅ Good: Separate commits for "Add dimension validation", "Remove unused variable", "Fix exit code handling"
- ❌ Bad: One mega-commit fixing all 15 unrelated review comments
- ❌ Bad: 15 tiny commits when 3 comments are about the same validation issue

```bash
FILE="path/from/context"
SUMMARY="Add archived=false filters to match functions"

git add "$FILE"
COMMIT_MSG=$(cat <<EOF
fix: ${SUMMARY}

Addresses PR #${PR} review comments ${CID1}, ${CID2}, ${CID3}.

- Rationale: …
- Behavior change: …
EOF
)

git commit -m "$COMMIT_MSG"
SHA=$(git rev-parse --short HEAD)

```

### 5) Reply in the thread with the commit

Use the **review comment reply** endpoint (note: it **must** include the PR number):

```bash
COMMENT_URL="https://github.com/${REPO}/pull/${PR}#discussion_r${CID}"
MSG="Fixed in commit ${SHA}: https://github.com/${REPO}/commit/$(git rev-parse HEAD)
\nSummary: ${SUMMARY}\nRefs: ${COMMENT_URL}"

gh api \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  repos/${REPO}/pulls/${PR}/comments/${CID}/replies \
  -f body="$MSG"
```

### 6) (Optional) Resolve the thread (GraphQL mutation)

> Only do this if instructed; some teams prefer reviewers resolve their own threads.

```bash
THREAD_ID="gid://github/PullRequestReviewThread/123456789"  # from step 1

gh api graphql -f query='
mutation($id:ID!){ resolveReviewThread(input:{threadId:$id}){ thread { id isResolved } } }
' -F id="$THREAD_ID"
```

---

## End‑to‑end automation loop (pseudo‑bash)

```bash
# 1) First, analyze all threads and group by logical issue
# Example grouping:
# - Group A: Comments 1,2,3 all about "Add archived filters"
# - Group B: Comment 4 about "Remove unused variable"
# - Group C: Comments 5,6 about "Add dimension validation"

# 2) For each logical group of related comments:
for group in "${ISSUE_GROUPS[@]}"; do
  # Get comment IDs in this group
  comment_ids=($(echo "$group" | jq -r '.comment_ids[]'))
  issue_summary=$(echo "$group" | jq -r '.summary')

  # 3) Implement all fixes for this logical issue
  # ... make changes to address all comments in this group ...

  # 4) MANDATORY: Run all tests
  npm run check    # Type checking and linting
  npm run test     # Unit tests
  # Run component tests if UI changes were made:
  # npm run test:components

  # 5) Create one atomic commit for this logical issue
  git add <affected_files>
  git commit -m "fix: ${issue_summary}

Addresses PR #${PR} review comments ${comment_ids[@]}.

- Rationale: …
- Changes: …"
  SHA=$(git rev-parse --short HEAD)
  FULL_SHA=$(git rev-parse HEAD)

  # 6) Reply to ALL comments in this group with the same commit link
  for cid in "${comment_ids[@]}"; do
    gh api -H "Accept: application/vnd.github+json" \
      repos/${REPO}/pulls/${PR}/comments/${cid}/replies \
      -f body="Fixed in commit ${SHA}: https://github.com/${REPO}/commit/${FULL_SHA}

${issue_summary}

Refs: https://github.com/${REPO}/pull/${PR}#discussion_r${cid}"
  done
done
```

---

## Prompts Claude Code should follow

* "Identify all **unresolved** review threads for PR ${PR}. Analyze and group comments by logical issue/fix."
* "Create a plan showing how comments will be grouped into atomic commits. Example: 'Commit 1: Add archived filters (addresses comments #1, #2, #3)', 'Commit 2: Remove unused variable (comment #4)'."
* "For each logical issue group, propose the **minimal** changes needed. Ask for confirmation if non‑trivial."
* "**MANDATORY**: After making changes for each group, run `npm run check` and `npm run test` (and `npm run test:components` if UI changes were made). Fix all failures before committing."
* "Create **one atomic commit per logical issue**. Group related comments that fix the same underlying problem."
* "After each commit, post a **threaded reply** to ALL comments in that group with the same commit link, summary, and rationale."

---

## Troubleshooting

### API and Query Issues

* **GraphQL syntax errors** → Don't use heredoc syntax or multi-line strings in `-f query=` parameter. Keep query on single line or use proper escaping.
* **"Expected VAR_SIGN" GraphQL errors** → Variable names in GraphQL query must match `-F` parameters exactly. Use `$owner`, `$name`, `$pr` (lowercase) consistently.
* **404 on replies endpoint** → Ensure endpoint includes `/pulls/${PR}/comments/${CID}/replies` and token has write scopes.
* **GraphQL query fails** → Check `gh auth status -t` and that the repo is accessible; reduce `first:100` if hitting limits.
* **Empty/null results from GraphQL** → GraphQL may not include full comment body. Use REST API to get full comment text: `gh api repos/${REPO}/pulls/${PR}/comments --paginate`

### Comment Filtering Issues

* **All comments have `in_reply_to_id` set** → Use GraphQL `reviewThreads` which automatically filters to top-level comments. Don't filter REST API by `in_reply_to_id == null`.
* **GraphQL returns 0 threads but REST shows many** → GraphQL is correct - REST API includes threaded replies. Use GraphQL for accurate count.
* **Comment IDs don't match** → GraphQL `databaseId` = REST API `id`. Don't confuse with GraphQL's `id` field (which is a global node ID).

### Code Issues

* **Outdated comments** → If the code line moved, apply fix at the new location; still reply in the same thread explaining the mapping.
* **Bots (e.g., coderabbitai[bot])** → It's fine to fix and reply; optionally filter authors with `| select(.author.login != "coderabbitai[bot]")`.

---

## Handling Impasses

When a review comment cannot be resolved after multiple back-and-forth exchanges (typically 2-3 rounds), follow this escalation process:

### When to Escalate

* The suggested change would require significant codebase-wide refactoring
* There's a legitimate technical disagreement about the approach
* The fix is out of scope for the current PR
* The suggestion is a "nice to have" improvement rather than a required fix

### Escalation Process

1. **Reply acknowledging the suggestion** and explain why it can't be addressed in this PR:

```bash
gh api -H "Accept: application/vnd.github+json" \
  repos/${REPO}/pulls/${PR}/comments/${CID}/replies \
  -f body="Acknowledged. This is a valid improvement but requires codebase-wide refactoring that's out of scope for this PR.

@coderabbitai please open a GitHub issue to track this improvement so it can be properly addressed in a dedicated PR.

Marking as resolved for this PR."
```

2. **Ask CodeRabbit to create an issue**: Include `@coderabbitai please open a GitHub issue` in your reply. CodeRabbit can automatically create tracked issues for deferred work.

3. **Resolve the thread** (if you have permission):

```bash
THREAD_ID="gid://github/PullRequestReviewThread/123456789"
gh api graphql -f query='
mutation($id:ID!){ resolveReviewThread(input:{threadId:$id}){ thread { id isResolved } } }
' -F id="$THREAD_ID"
```

### Example Impasse Scenarios

| Scenario | Response |
|----------|----------|
| "Use AuthenticatedRequest interface" (requires refactoring all routes) | Acknowledge, explain middleware guarantees safety, ask CodeRabbit to create issue |
| "Add comprehensive error handling" (large scope) | Acknowledge value, note current PR scope, request issue for follow-up |
| "Refactor to use newer pattern" (not a bug fix) | Acknowledge as improvement, defer to issue for dedicated PR |

### Key Points

* **Don't block PRs indefinitely** on suggestions that are improvements rather than fixes
* **Always acknowledge** the validity of the suggestion
* **Create traceability** by having CodeRabbit open an issue
* **Document the decision** in your reply for future reference

---

## Success criteria

* All previously unresolved threads are either:

  1. replied with a commit link and awaiting reviewer confirmation, or
  2. acknowledged with a request for CodeRabbit to create a tracking issue, or
  3. (if allowed) resolved via mutation after successful verification.
* **All tests pass**: `npm run check`, `npm run test`, and `npm run test:components` (if applicable) all succeed.
* CI is green; no style/lint regressions; commit messages are clear and traceable.
