---
name: respond-to-coderabbitai
description: Drive a pull request to merge by resolving CodeRabbit (or any reviewer) PR comments with atomic commits and threaded replies, then iterate across multiple review rounds while monitoring CI. Use when the user mentions CodeRabbit, PR review comments, unresolved review threads, or asks to clear/respond to/address review feedback on a pull request, or to land/merge a PR once CI is green.
allowed-tools: Bash(gh:*) Bash(git:*) Bash(jq:*) Bash(npm:*) Bash(sleep:*) Bash(date:*) Bash(cat:*) Bash(echo:*) Bash(grep:*) Bash(sed:*) Bash(sort:*) Bash(comm:*) Read Edit Write Monitor TaskStop
compatibility: Requires git, gh CLI (authenticated with repo write scope), jq, and network access to GitHub. The iterative monitoring loop relies on a Monitor-style background-task tool; agents without one should run the polling script as a long-running shell job instead.
metadata:
  author: seiraiyu-skills
  version: "2.0"
---

## Purpose

Enable Claude Code to **drive a PR to merge** by clearing all CodeRabbit review threads through however many rounds it takes, monitoring CI continuously, fixing failures, and merging when everything is green.

> ## ⚠️ READ THIS FIRST — DO NOT STOP AFTER ONE PASS
>
> **CodeRabbit will almost always go 3–5 rounds of review on a non-trivial PR. Plan for it, budget for it, and do not declare the task complete until CodeRabbit itself has confirmed completion or fallen silent for a full re-review cycle after an explicit `@coderabbitai` ping.**
>
> The single most common failure mode of this skill is **stopping too early** — fixing the first batch of comments, pushing, and walking away while CodeRabbit is still re-reviewing. That is **not done**. Each push triggers a fresh review. New comments on the new code are the rule, not the exception. The job is finished when:
>
> 1. There are **zero unresolved threads** authored by CodeRabbit, AND
> 2. CodeRabbit has either explicitly confirmed completion or stayed silent through one full re-review cycle (~5–10 minutes) after you pinged `@coderabbitai`, AND
> 3. CI is fully green on the latest commit.
>
> If any of those three conditions is not met, **the loop is still running**. Keep the Monitor task alive and keep iterating. Do not summarize, do not hand off, do not move on.

**The full lifecycle is iterative.** CodeRabbit typically requires **3–5 rounds** of review before it is content (occasionally more on large PRs). After each round of fixes, CodeRabbit re-reviews and **almost always raises new comments on the new code** — including comments on lines you just changed in response to a previous comment. The skill:

1. Resolves all current unresolved threads with atomic commits + threaded replies.
2. Pushes, then **uses the Monitor tool** to watch the PR for new CodeRabbit reviews and CI results.
3. Repeats round-by-round until CodeRabbit explicitly confirms the review is complete (and an explicit `@coderabbitai` ping is used to ask if needed).
4. When CI is green AND CodeRabbit has no remaining concerns, merges the PR and pulls the result locally.
5. If CI fails at any point, fixes the failure as another atomic commit and continues the loop.

**Key principles:**
- **Atomic commits per logical issue.** One commit per problem, even if it addresses multiple comments.
- **One Monitor task running across all rounds.** Do not stop and restart it between rounds.
- **Never declare done after a single pass.** CodeRabbit's first re-review almost always finds something. Plan to iterate 3–5 times.
- **Every comment must be addressed** — either with a fix + threaded reply linking the commit, or with an explicit acknowledgement and an `@coderabbitai please open a GitHub issue` deferral. No silent skips.
- **Silence ≠ done.** "No new events for 30 seconds" is not a completion signal; only an explicit CodeRabbit confirmation, or silence after a deliberate ping, counts.

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

## Iterative review loop with Monitor

**This is the heart of the skill. Read it carefully.**

CodeRabbit reviews are **never** one-shot on a non-trivial PR. After you push fixes, CodeRabbit re-runs within a minute or two and frequently raises **new** comments — sometimes on the very lines you just edited, sometimes on adjacent code it didn't flag the first time, sometimes nitpicks that only surfaced after the bigger issues were fixed. **Expect 3–5 rounds.** Treating the first round as the last round is the #1 way this skill fails.

Use the **Monitor tool** to drive the loop without manual polling. The Monitor stays armed across **every** round; do not stop it until the PR is merged.

### When to start the monitor

Start the monitor **after pushing your first round of fix commits**. The monitor should:
- Watch for new CodeRabbit reviews and unresolved threads on the PR.
- Watch CI check status (success / failure / pending).
- Emit a line per state change so you can react.

### Monitor command

Use a single persistent Monitor task that polls the PR every ~30s and emits one line per actionable change. Replace `${REPO}` and `${PR}`:

```bash
# Save as /tmp/pr_watch.sh, then invoke via the Monitor tool
REPO="owner/repo"
PR=123
prev_unresolved=""
prev_checks=""
prev_review_count=""

while true; do
  # Unresolved threads (count + IDs)
  unresolved=$(gh api graphql -f query='
    query($o:String!,$n:String!,$p:Int!){
      repository(owner:$o,name:$n){
        pullRequest(number:$p){
          reviewThreads(first:100){ nodes{ isResolved id comments(first:1){ nodes{ author{login} } } } }
        }
      }
    }' -F o="${REPO%%/*}" -F n="${REPO##*/}" -F p="$PR" 2>/dev/null \
    | jq -r '.data.repository.pullRequest.reviewThreads.nodes[]
        | select(.isResolved==false)
        | select(.comments.nodes[0].author.login=="coderabbitai")
        | .id' | sort)

  # CodeRabbit review submissions (to detect a new review round)
  review_count=$(gh api "repos/${REPO}/pulls/${PR}/reviews" --jq \
    '[.[] | select(.user.login=="coderabbitai[bot]")] | length' 2>/dev/null || echo "0")

  # CI checks
  checks=$(gh pr checks "$PR" --repo "$REPO" --json name,bucket 2>/dev/null \
    | jq -r '.[] | "\(.name): \(.bucket)"' | sort)

  # Emit on changes
  if [ "$unresolved" != "$prev_unresolved" ]; then
    cnt=$(echo "$unresolved" | grep -c . || true)
    echo "UNRESOLVED_THREADS: $cnt"
    prev_unresolved="$unresolved"
  fi
  if [ "$review_count" != "$prev_review_count" ] && [ -n "$prev_review_count" ]; then
    echo "NEW_CODERABBIT_REVIEW: total=$review_count"
  fi
  prev_review_count="$review_count"

  if [ "$checks" != "$prev_checks" ]; then
    echo "CI_STATE_CHANGE:"
    echo "$checks" | sed 's/^/  /'
    if echo "$checks" | grep -q ": failure"; then echo "CI_FAILURE_DETECTED"; fi
    if [ -n "$checks" ] && ! echo "$checks" | grep -qE ": (pending|failure)"; then
      echo "CI_ALL_GREEN"
    fi
    prev_checks="$checks"
  fi

  sleep 30
done
```

Launch with `Monitor` using `persistent: true` and a description like `"PR ${PR} CodeRabbit threads + CI"`. The monitor stays armed across rounds; you do not need to restart it after each push.

### Reacting to monitor events

| Event line | Action |
|---|---|
| `UNRESOLVED_THREADS: N` (N>0, increased) | New review round landed. Re-run discovery (step 1), group, fix, commit, reply, push. |
| `UNRESOLVED_THREADS: 0` | All threads addressed for this round. Proceed to "Confirm review complete". |
| `NEW_CODERABBIT_REVIEW: total=X` | A fresh review was submitted — expect new comments; re-run discovery. |
| `CI_FAILURE_DETECTED` | Fetch logs (`gh run view --log-failed`), fix root cause, commit, push. |
| `CI_ALL_GREEN` + threads at 0 + CodeRabbit confirmed | Proceed to merge. |

### Confirm review is complete with an explicit ping

When unresolved threads hit 0 and you believe you've addressed everything, **post an explicit comment asking CodeRabbit to confirm**, then keep the monitor running for at least one more poll cycle (~60s) to catch any reply or new review:

```bash
gh pr comment "$PR" --repo "$REPO" --body \
"@coderabbitai I believe all review comments have been addressed. Could you confirm the review is complete or flag any remaining concerns?"
```

CodeRabbit responds either with new review comments (loop continues) or with a confirmation. Only treat the review as complete once it explicitly says so or makes no new comments after the ping + one full review cycle.

---

## CI monitoring and failure recovery

The same Monitor task watches CI. On `CI_FAILURE_DETECTED`:

```bash
# Identify the failing run
RUN_ID=$(gh run list --repo "$REPO" --branch "$(git branch --show-current)" \
  --limit 1 --json databaseId,status,conclusion \
  --jq '.[] | select(.conclusion=="failure") | .databaseId')

# Get failed step logs
gh run view "$RUN_ID" --repo "$REPO" --log-failed > /tmp/ci_failure.log

# Diagnose, fix, commit as its own atomic commit:
git add <files>
git commit -m "fix(ci): <root cause summary>

Addresses CI failure in run ${RUN_ID}."
git push
```

Do **not** disable, skip, or `--no-verify` past failing checks. Fix the root cause. The monitor will re-emit `CI_ALL_GREEN` once the next run passes.

---

## Auto-merge when green

Only merge when **all** of the following hold:

1. `UNRESOLVED_THREADS: 0` from the monitor.
2. `CI_ALL_GREEN` from the monitor (no `pending` or `failure` checks).
3. CodeRabbit has either explicitly confirmed completion or made no new comments after the explicit `@coderabbitai` ping and a full re-review cycle.
4. PR is mergeable (`gh pr view ${PR} --json mergeable,mergeStateStatus` shows `MERGEABLE` / `CLEAN`).

Then merge and pull:

```bash
# Prefer squash unless the repo convention says otherwise
gh pr merge "$PR" --repo "$REPO" --squash --delete-branch

# Pull the merged result locally
git checkout main   # or the repo's default branch
git pull --ff-only
```

Stop the monitor (`TaskStop`) once the PR is merged.

**Safety guardrails:**
- Do **not** merge if the user has not authorized merge for this PR. Auto-mode permits the loop but a destructive/shared-state action like merge still needs the user's go-ahead unless they explicitly said "merge when green."
- Never bypass branch protection (`--admin`) without explicit permission.
- Never force-push to the PR branch to "fix" a failing check.

---

## Success criteria

* All CodeRabbit review threads across **every round** are either:
  1. replied with a commit link and resolved (or awaiting reviewer auto-resolve), or
  2. acknowledged with a request for CodeRabbit to open a tracking issue (impasse path).
* CodeRabbit has confirmed completion (explicitly, or by silence after the `@coderabbitai` ping + one full review cycle).
* **All tests pass locally**: `npm run check`, `npm run test`, and `npm run test:components` (if applicable).
* **CI is fully green** on the latest commit (no pending, no failures).
* PR is merged and the default branch is pulled locally.
* Monitor task is stopped.
