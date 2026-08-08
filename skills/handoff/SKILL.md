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
  version: "2.0"
---

# Handoff

Turn the live conversation into a **handoff document**: everything the next agent
needs to continue, and nothing it can find for itself. Optimize for a cold start —
assume the reader has zero memory of this session.

## Two non-negotiable rules

These are the two ways this skill fails in practice. Both are absolute:

1. **The file goes in the OS temp directory. It NEVER goes in the workspace.**
   Not `docs/`, not the project root, not a `handoff/` folder, not anywhere inside
   the repository — no exceptions, and never `git add` it. A handoff is session
   scratch, not a project artifact; writing it into the repo pollutes the working
   tree and may land in the wrong repo entirely. If at any point a handoff file
   exists inside the workspace (including one you just wrote by mistake), delete
   it and rewrite it to the temp directory before continuing.

2. **Your turn ends with the paste-ready pickup prompt — every time.**
   Writing the file is NOT completion. The skill is complete only when the last
   thing in your final message is the fenced pickup block from the "Final message
   format" section below, containing the doc's absolute path. If you are about to
   end your turn and that block is not the last thing you wrote, the handoff is
   unfinished — write it now.

## Step 1 — Anchor the focus

If the user passed arguments, that is the next session's mission — lead with it and
bias every section toward it (the relevant files, the open question, the next action).

If they didn't, infer the most likely continuation from the recent turns and state
your assumption in one line at the top so the reader can correct it.

## Step 2 — Resolve the temp path

```bash
# Cross-platform temp dir + collision-proof, descriptive filename
DIR="${TMPDIR:-${TEMP:-/tmp}}"
STAMP="$(date +%Y%m%d-%H%M%S)"
FILE="$DIR/handoff-<slug>-$STAMP.md"   # <slug> = 2-4 word kebab-case topic
```

macOS/Linux/WSL resolves to `$TMPDIR` or `/tmp`; Windows shells use `%TEMP%`.
Use a descriptive slug (e.g. `handoff-auth-rate-limit-20260608-141030.md`) so the
user can find it among other temp files. Before writing, confirm the resolved path
is outside the workspace — if it somehow isn't, use `/tmp` directly.

## Step 3 — Write the document

Use the template in [references/handoff-template.md](references/handoff-template.md).
Fill every section; delete a heading only if it is genuinely empty.

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

Before saving, scrub API keys, tokens, passwords, connection strings, and `.env`
values, plus PII not needed to continue the work. Keep ticket/PR URLs — they're the
point. When in doubt, redact the value but keep the **name** so the reader knows a
secret is needed there (e.g. `DATABASE_URL=[REDACTED — in 1Password/Neon]`).

## Step 5 — Suggested skills

Include a **Suggested skills** section listing skills the next agent should invoke,
each with a one-line why. Draw from the skills available in the target environment
(e.g. `respond-to-coderabbitai` for an open PR, `jira` for ticket updates, `neon`
for database branches). Only suggest skills that plausibly apply — an irrelevant
list erodes trust.

## Step 6 — Final message format (mandatory)

After the file is written, your final message MUST follow this exact shape and MUST
be the last thing in your turn — no trailing commentary after the fenced block:

1. One line: `Handoff written: <absolute path>`
2. 1-3 sentences summarizing what the doc covers.
3. The literal line: `To continue with a clean context, run /clear then paste:`
4. A fenced code block the user can copy verbatim, following this pattern:

```
Read the handoff at <absolute path> and resume the work. Start with the
"single most important next action", and invoke any skills listed under
"Suggested skills".
```

The fenced block must name the **absolute path** and be fully self-contained — the
fresh agent has no memory of this session. Do not paraphrase the block away, do not
merely mention that the user "can resume later", and do not end the turn without it.

## Anti-patterns

- **Dumping the transcript** — a handoff is a synthesis, not a log.
- **Copying content that already lives somewhere durable** — link to it.
- **Writing into the workspace** — see rule 1. Delete and relocate if it happened.
- **Ending without the pickup block** — see rule 2. The prompt IS the deliverable.
- **Vague next steps** — the single most important line is "do this next."
- **Forgetting git state** — the next agent needs branch, dirty files, unpushed work.
