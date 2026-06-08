# Handoff — <Topic>

> **For the next session.** You have no memory of the prior conversation. This doc
> is everything you need to continue. Artifacts are referenced by path/URL — open
> them rather than expecting their contents inline.

- **Created:** <YYYY-MM-DD HH:MM> by a prior agent session
- **Next session focus:** <the mission — from user args, or your stated assumption>
- **Repo / location:** <repo name + absolute path>
- **Git state:** branch `<branch>`, <clean | N dirty files | M unpushed commits>

## TL;DR

<3-5 sentences: what we were doing, where it stands, what to do next.>

## The single most important next action

<One concrete, runnable step. Command(s), expected result, where to look first.>

## Context & goal

<Why this work exists. The problem being solved and the desired end state.
Link the source of truth instead of restating it:>
- PRD / spec: <path or URL>
- Plan / ADR: <path or URL>
- Ticket: <Jira key / issue URL>
- PR: <#number / URL>

## What's been done

<Bullets of completed work. Reference commits/diffs, don't paste them.>
- <e.g. Added rate-limit middleware — `git show abc123`>
- <e.g. Schema migration applied to Neon branch `dev/rate-limit`>

## Current state of the work

<What's half-built, what's stubbed, what's known-broken. Point at exact files/lines.>
- `src/.../file.ts:42` — TODO: <what remains>

## Decisions & rationale

<Decisions made and *why*, so they aren't relitigated.>
- Chose **<X>** over **<Y>** because **<Z>**.

## Blockers, dead ends & things already tried

<Save the next agent from repeating failed approaches.>
- Tried <approach> → failed because <reason>. Don't retry without <change>.
- Blocked on <thing>; need <person/access/decision>.

## Open questions

- <Unresolved question the next agent (or the user) must answer.>

## How to resume — environment & commands

<Exact setup needed: cwd, branch checkout, env vars (redacted), build/test commands.>

```bash
cd <path>
git checkout <branch>
<install / build>
<test command — expected result>
```

- Required secrets/config: `<NAME>=[REDACTED — where to get it]`

## Suggested skills

<Skills the next agent should invoke, each with a one-line why and trigger.>
- `<skill>` — <why> (trigger: "<phrase>")

## Notes / scratch

<Anything else useful that didn't fit above.>
