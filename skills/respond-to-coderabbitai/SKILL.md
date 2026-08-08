---
name: respond-to-coderabbitai
description: Drive a pull request to merge by resolving CodeRabbit (or any reviewer) PR comments with atomic commits and threaded replies, then iterate across multiple review rounds while a background Monitor tracks both new CodeRabbit comments and CI status. Use when the user mentions CodeRabbit, PR review comments, unresolved review threads, or asks to clear/respond to/address review feedback on a pull request, or to land/merge a PR once CI is green. CodeRabbit typically requires 3-5+ review rounds — never declare done after one pass.
allowed-tools: Bash(gh:*) Bash(git:*) Bash(jq:*) Bash(npm:*) Bash(sleep:*) Bash(date:*) Bash(cat:*) Bash(echo:*) Bash(grep:*) Bash(sed:*) Bash(sort:*) Bash(comm:*) Read Edit Write Monitor TaskStop
compatibility: Requires git, gh CLI (authenticated with repo write scope), jq, and network access to GitHub. A Monitor-style background-task tool is required for the iterative review loop and CI-green gating; without one, run the polling script as a long-running shell job and tail its output between actions.
metadata:
  author: seiraiyu-skills
  version: "3.0"
---

# Respond to CodeRabbit

Drive a PR to merge by clearing all CodeRabbit review threads through however many
rounds it takes, with a persistent Monitor watching for new comments and CI status.

## Completion contract

The job is finished only when ALL THREE hold simultaneously on the latest commit:

1. **Zero unresolved threads** authored by CodeRabbit on the PR.
2. **CodeRabbit has confirmed completion** — explicitly, or by staying silent
   through one full re-review cycle (~5-10 minutes) after you posted an explicit
   `@coderabbitai` confirmation ping.
3. **CI is fully green** — confirmed by the Monitor emitting `CI_ALL_GREEN`, not
   by a one-shot `gh pr checks` snapshot (queued and re-run checks make snapshots
   unreliable).

Expect **3-5 review rounds** on a non-trivial PR, 6-10+ on large or sensitive ones.
Each push triggers a fresh CodeRabbit review within ~1-2 minutes, and it routinely
raises new comments on the new code — including lines you just changed. The most
common failure mode of this skill is stopping after round 1; "I addressed
everything" is a round-1 status update, not a completion claim. While any contract
condition is unmet, the loop is still running.

## The loop

1. **Discover** all unresolved review threads and their comment bodies —
   see [references/discovery.md](references/discovery.md).
2. **Group** comments by logical issue and plan atomic commits: one commit per
   underlying problem, even when it spans several comments; unrelated fixes stay
   in separate commits.
3. **Fix** each group with the minimal change that satisfies the reviewer, run the
   project's checks and tests, and commit. Reference the PR and comment IDs in the
   commit message.
4. **Reply** in every thread of the group with the commit link and a one-line
   summary (endpoint details in [references/discovery.md](references/discovery.md)).
   No silent skips: a comment that won't be fixed gets an acknowledgement and an
   `@coderabbitai please open a GitHub issue` deferral — see the impasse process in
   [references/troubleshooting.md](references/troubleshooting.md).
5. **Push, then immediately launch the persistent Monitor** — script and event
   reference in [references/monitor.md](references/monitor.md). Launch it after
   your first push, before any other post-push action, and keep the same Monitor
   task alive across every round until the PR is merged. It is the only reliable
   signal for new review rounds and CI transitions.
6. **React to Monitor events**: new unresolved threads → run another round from
   step 1; CI failure → fix the root cause as its own atomic commit
   ([references/monitor.md](references/monitor.md) covers log retrieval); threads
   at zero → post the `@coderabbitai` confirmation ping and keep watching.
7. **Merge** only when the completion contract is satisfied and the PR is
   mergeable (`gh pr view --json mergeable,mergeStateStatus` shows
   `MERGEABLE`/`CLEAN`):

   ```bash
   gh pr merge "$PR" --repo "$REPO" --squash --delete-branch
   git checkout main && git pull --ff-only
   ```

   Then stop the Monitor (`TaskStop`) — it is always the last thing torn down.

## Guardrails

- Do not merge without the user's go-ahead unless they explicitly said
  "merge when green."
- Never bypass failing checks: no `--no-verify`, no `--admin`, no force-pushing
  the PR branch to "fix" a check.
- Silence is not completion — only an explicit CodeRabbit confirmation, or a full
  quiet re-review cycle after a deliberate ping, satisfies condition 2.

## References

- [references/discovery.md](references/discovery.md) — GraphQL/REST thread
  discovery, comment bodies, threaded replies, resolving threads.
- [references/monitor.md](references/monitor.md) — the persistent Monitor script,
  event/reaction table, CI failure recovery.
- [references/troubleshooting.md](references/troubleshooting.md) — API quirks,
  comment-filtering pitfalls, and the impasse escalation process.
