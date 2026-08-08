# The persistent Monitor

Launch one Monitor task immediately after the first push of fix commits — before
any other post-push action — and keep the same task alive across every round until
the PR is merged. It watches CodeRabbit activity and CI in one place; manual
one-shot polling misses queued runs, late reviews, and re-checks.

## Monitor script

Save as `/tmp/pr_watch.sh`, then launch via the Monitor tool with
`persistent: true` and a description like `"PR ${PR} CodeRabbit threads + CI"`.
Polls every ~30s, emits one line per actionable change.

```bash
REPO="owner/repo"
PR=123
prev_unresolved=""
prev_checks=""
prev_review_count=""

while true; do
  # Unresolved CodeRabbit threads
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

  # CodeRabbit review submissions (detects a new review round)
  review_count=$(gh api "repos/${REPO}/pulls/${PR}/reviews" --jq \
    '[.[] | select(.user.login=="coderabbitai[bot]")] | length' 2>/dev/null || echo "0")

  # CI checks
  checks=$(gh pr checks "$PR" --repo "$REPO" --json name,bucket 2>/dev/null \
    | jq -r '.[] | "\(.name): \(.bucket)"' | sort)

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

## Reacting to events

| Event line | Action |
|---|---|
| `UNRESOLVED_THREADS: N` (N>0, increased) | New review round landed. Re-run discovery, group, fix, commit, reply, push. |
| `UNRESOLVED_THREADS: 0` | All threads addressed this round. Post the confirmation ping below. |
| `NEW_CODERABBIT_REVIEW: total=X` | Fresh review submitted — expect new comments; re-run discovery. |
| `CI_FAILURE_DETECTED` | Fetch failed logs (below), fix root cause, commit, push. |
| `CI_ALL_GREEN` + threads at 0 + CodeRabbit confirmed | Proceed to merge. |

## Confirmation ping

When unresolved threads hit 0, ask CodeRabbit explicitly, then keep the Monitor
running through at least one full re-review cycle (~5-10 minutes):

```bash
gh pr comment "$PR" --repo "$REPO" --body \
"@coderabbitai I believe all review comments have been addressed. Could you confirm the review is complete or flag any remaining concerns?"
```

CodeRabbit responds with new comments (loop continues) or a confirmation.

## CI failure recovery

On `CI_FAILURE_DETECTED`:

```bash
RUN_ID=$(gh run list --repo "$REPO" --branch "$(git branch --show-current)" \
  --limit 1 --json databaseId,status,conclusion \
  --jq '.[] | select(.conclusion=="failure") | .databaseId')

gh run view "$RUN_ID" --repo "$REPO" --log-failed > /tmp/ci_failure.log

# Diagnose, fix, commit as its own atomic commit:
git add <files>
git commit -m "fix(ci): <root cause summary>

Addresses CI failure in run ${RUN_ID}."
git push
```

The Monitor re-emits `CI_ALL_GREEN` once the next run passes. Stop the Monitor
(`TaskStop`) only after the PR is merged and pulled locally.
