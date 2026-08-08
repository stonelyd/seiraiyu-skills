# Thread discovery, replies, and resolution

## 1. Discover all unresolved review threads (GraphQL — recommended)

GraphQL returns unresolved threads directly, already filtered to top-level
comments. This is more reliable than filtering REST results.

```bash
REPO="owner/repo"
PR=123

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

echo "=== Found ${#THREADS[@]} unresolved threads ==="
for i in "${!THREADS[@]}"; do
  echo -e "\n--- Thread $((i+1)) ---"
  echo "${THREADS[$i]}" | jq -r '"Comment ID: \(.top_comment_id)\nFile: \(.path)\nAuthor: \(.author)\nThread ID: \(.thread_id)"'
done
```

## 2. Fetch full comment bodies (REST)

GraphQL doesn't include full body text; batch-fetch via REST once you have IDs:

```bash
gh api repos/${REPO}/pulls/${PR}/comments --paginate > /tmp/all_comments.json

for CID in "${COMMENT_IDS[@]}"; do
  echo -e "\n=========================================="
  echo "COMMENT ID: $CID"
  jq -r ".[] | select(.id == $CID) | \"File: \(.path):\(.line // \"null\")\\nAuthor: \(.user.login)\\n\\nBody:\\n\(.body)\\n\"" /tmp/all_comments.json
done

# Or fetch a single comment:
gh api repos/${REPO}/pulls/${PR}/comments/${CID} \
  | jq -r '"File: \(.path):\(.line // \"null\")\nAuthor: \(.user.login)\n\nBody:\n\(.body)"'
```

Key facts:
- GraphQL `databaseId` = REST `id` (don't confuse with GraphQL's global-node `id`).
- `reviewThreads` already excludes threaded replies — don't filter REST output by
  `in_reply_to_id` yourself.

## 3. Commit referencing the comments

```bash
git add "$FILE"
git commit -m "fix: ${SUMMARY}

Addresses PR #${PR} review comments ${CID1}, ${CID2}.

- Rationale: …
- Behavior change: …"
SHA=$(git rev-parse --short HEAD)
```

## 4. Reply in the thread

Use the review-comment **replies** endpoint (it must include the PR number):

```bash
gh api \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  repos/${REPO}/pulls/${PR}/comments/${CID}/replies \
  -f body="Fixed in commit ${SHA}: https://github.com/${REPO}/commit/$(git rev-parse HEAD)

${SUMMARY}"
```

When one commit addresses several comments, reply to every comment in the group
with the same commit link.

## 5. Resolve the thread (optional — GraphQL mutation)

Only if instructed; some teams prefer reviewers resolve their own threads.

```bash
THREAD_ID="gid://github/PullRequestReviewThread/123456789"  # from step 1

gh api graphql -f query='
mutation($id:ID!){ resolveReviewThread(input:{threadId:$id}){ thread { id isResolved } } }
' -F id="$THREAD_ID"
```
