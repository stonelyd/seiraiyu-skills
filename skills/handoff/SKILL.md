---
name: handoff
description: >
  Compact the current conversation into a self-contained handoff document so a
  fresh agent (or a future you) can resume the work without re-reading the whole
  session. Use when the user says "hand off", "write a handoff", "summarize this
  session for another agent", "I'm running low on context", "pause and pick up
  later", or before clearing/compacting a long conversation. If the user passes
  arguments, treat them as the focus of the next session and tailor the doc to it.
argument-hint: "What will the next session focus on? (optional)"
allowed-tools: Bash(date:*) Bash(mktemp:*) Bash(printenv:*) Bash(uname:*) Bash(git:*) Read Write
metadata:
  author: seiraiyu-skills
  version: "1.0"
---

# Handoff

Turn the live conversation into a **handoff document**: everything the next agent
needs to continue, and nothing it can find for itself. Optimize for a cold start —
assume the reader has zero memory of this session.

## Definition of Done

This is complete when:
- [ ] A handoff doc is written to the OS temp directory (never the workspace)
- [ ] The reader could resume work from the doc alone, without this transcript
- [ ] Durable artifacts (PRDs, plans, ADRs, issues, commits, diffs) are **referenced by path/URL**, not copied
- [ ] A "Suggested skills" section names the skills the next agent should invoke
- [ ] Secrets and PII are redacted
- [ ] The final message presents a **paste-ready resume prompt** (`/clear` then paste) naming the doc's **absolute path**

## Step 1 — Anchor the focus

If the user passed arguments, that is the next session's mission — lead with it and
bias every section toward it (the relevant files, the open question, the next action).

If they didn't, infer the most likely continuation from the recent turns and state
your assumption in one line at the top so the reader can correct it.

## Step 2 — Pick the destination

Write to the **OS temp directory**, not the current workspace (the next agent may be
in a different repo, branch, or worktree). Resolve it in this order:

```bash
# Cross-platform temp dir + collision-proof, descriptive filename
DIR="${TMPDIR:-${TEMP:-/tmp}}"
STAMP="$(date +%Y%m%d-%H%M%S)"
FILE="$DIR/handoff-<slug>-$STAMP.md"   # <slug> = 2-4 word kebab-case topic
```

- macOS / Linux / WSL: `$TMPDIR` then `/tmp`.
- Windows shells: `%TEMP%` / `$env:TEMP`.
- Use a descriptive slug (e.g. `handoff-auth-rate-limit-20260608-141030.md`) so the
  user can find it later among other temp files.

## Step 3 — Write the document

Use the template in [references/handoff-template.md](references/handoff-template.md).
Fill every section; delete a heading only if it is genuinely empty.

Guiding rules:

| Do | Don't |
|----|-------|
| Reference artifacts by path/URL (`See plan.md`, `PR #214`, `git show abc123`) | Paste diffs, full files, or long logs |
| Capture **decisions and their rationale** ("chose X over Y because Z") | Re-narrate the play-by-play of the chat |
| State the **exact next action** ("run `npm test auth`, expect 2 failures in …") | Leave the next step vague ("continue the work") |
| List blockers, dead ends, and things already tried | Make the next agent rediscover dead ends |
| Record exact commands, file paths, env/flags needed to resume | Assume the reader shares this session's working dir |
| Note current git branch, dirty state, unpushed commits | Omit where the code actually lives |

Keep it scannable — headers, bullets, tables. A handoff nobody reads is worthless.

## Step 4 — Redact

Before saving, scrub:
- API keys, tokens, passwords, connection strings, `.env` values → `[REDACTED]`
- Personally identifiable information not needed to continue the work
- Internal hostnames / private URLs only if sensitive (keep ticket/PR URLs — they're the point)

When in doubt, redact the value but keep the **name** so the reader knows a secret is needed there (e.g. `DATABASE_URL=[REDACTED — in 1Password/Neon]`).

## Step 5 — Suggested skills

Always include a **Suggested skills** section listing skills the next agent should
invoke, each with a one-line why and (if known) the trigger phrase. Draw from the
skills available in the target environment. Common picks:

- `respond-to-coderabbitai` — if a PR is open and awaiting review resolution
- `jira` — if work is tracked in Jira and the ticket needs updating/transitioning
- `neon` — if the work touches a Neon database branch
- `sop-creator` — if the outcome should be documented as a runbook
- `verify` / `code-review` — before claiming the resumed work is done

Only suggest skills that plausibly apply — an irrelevant list erodes trust.

## Step 6 — Present the resume prompt

The whole point is to start fresh. After writing the doc, present a **paste-ready
prompt** the user can run in a clean context — mirroring how `/plan` ends. Use the
actual file path; the user copies one block after clearing.

```
Handoff written: /tmp/handoff-auth-rate-limit-20260608-141030.md
Covers: rate-limit middleware half-built on branch `feat/rate-limit`; next step is
wiring the Redis store (TODO at src/middleware/rateLimit.ts:42). PR #214 open.

To continue with a clean context, run `/clear` then paste:

Read the handoff at /tmp/handoff-auth-rate-limit-20260608-141030.md and resume the
work. Start with the "single most important next action", and invoke any skills
listed under "Suggested skills".
```

Keep the pasted block self-contained: it must name the absolute path and tell the
fresh agent to start from the doc — that agent has no memory of this session.

## Anti-patterns

- **Dumping the transcript** — a handoff is a synthesis, not a log.
- **Copying content that already lives somewhere durable** — link to it.
- **Writing into the workspace** — pollutes the repo and may be in the wrong one.
- **Vague next steps** — the single most important line is "do this next."
- **Forgetting git state** — the next agent needs branch, dirty files, unpushed work.
