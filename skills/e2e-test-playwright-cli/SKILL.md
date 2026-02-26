---
name: e2e-test-playwright-cli
description: Comprehensive end-to-end testing command. Launches parallel sub-agents to research the codebase (structure, database schema, potential bugs), then uses the Playwright CLI (@playwright/cli) to test every user journey — taking screenshots, capturing traces, validating UI/UX, and querying the database to verify records. Run after implementation to validate everything before code review.
disable-model-invocation: true
---

# End-to-End Application Testing (Playwright CLI)

## Pre-flight Check

### 1. Platform Check

playwright-cli supports **Linux, macOS, WSL, and Windows**. Check the platform for informational purposes:
```bash
uname -s
```
All platforms are supported. No platform gate is needed.

### 2. Frontend Check

Verify the application has a browser-accessible frontend. Check for:
- A `package.json` with a dev/start script serving a UI
- Frontend framework files (pages/, app/, src/components/, index.html, etc.)
- Web server configuration

If no frontend is detected:
> "This application doesn't appear to have a browser-accessible frontend. E2E browser testing requires a UI to visit. For backend-only or API testing, a different approach is needed."

Stop execution if no frontend is found.

### 3. playwright-cli Installation

Check if playwright-cli is installed:
```bash
playwright-cli --version
```

If the command is not found, install it automatically:
```bash
npm install -g @playwright/cli@latest
```

After installation (or if it was already installed), ensure the browser engine is set up:
```bash
playwright-cli install-browser
```

Verify installation succeeded:
```bash
playwright-cli --version
```

If installation fails, stop with:
> "Failed to install playwright-cli. Please install it manually with `npm install -g @playwright/cli@latest && playwright-cli install-browser`, then re-run this command."

## Phase 1: Parallel Research

Launch **three sub-agents simultaneously** using the Task tool. All three run in parallel.

### Sub-agent 1: Application Structure & User Journeys

> Research this codebase thoroughly. Return a structured summary covering:
>
> 1. **How to start the application** — exact commands to install dependencies and run the dev server, including the URL and port it serves on
> 2. **Authentication/login** — if the app has protected routes, how to create a test account or log in (credentials from .env.example, seed data, or sign-up flow)
> 3. **Every user-facing route/page** — each URL path and what it renders
> 4. **Every user journey** — complete flows a user can take (e.g., "sign up → create profile → view public page"). For each journey, list the specific steps, interactions (clicks, form fills, navigation), and expected outcomes
> 5. **Key UI components** — forms, modals, dropdowns, pickers, toggles, and other interactive elements that need testing
> 6. **Environment setup** — check for `.env.example`, database seeding scripts, migration commands, or any other setup steps needed before the app can run
>
> Be exhaustive. Testing will only cover what you identify here.

### Sub-agent 2: Database Schema & Data Flows

> Research this codebase's database layer. Read `.env.example` to understand environment variables for database connections. DO NOT read `.env` directly. Return a structured summary covering:
>
> 1. **Database type and connection** — what database is used (Postgres, MySQL, SQLite, etc.) and the environment variable name for the connection string (from .env.example)
> 2. **Full schema** — every table, its columns, types, and relationships
> 3. **Data flows per user action** — for each user-facing action (form submit, button click, etc.), document exactly what records are created, updated, or deleted and in which tables
> 4. **Validation queries** — for each data flow, provide the exact query to verify records are correct after the action

### Sub-agent 3: Bug Hunting

> Analyze this codebase for potential bugs, issues, and code quality problems. Focus on:
>
> 1. **Logic errors** — incorrect conditionals, off-by-one errors, missing null checks, race conditions
> 2. **UI/UX issues** — missing error handling in forms, no loading states, broken responsive layouts, accessibility problems
> 3. **Data integrity risks** — missing validation, potential orphaned records, incorrect cascade behavior
> 4. **Security concerns** — SQL injection, XSS, missing auth checks, exposed secrets
>
> Return a prioritized list with file paths and line numbers.

**Wait for all three sub-agents to complete before proceeding.**

## Phase 2: Start the Application

Using Sub-agent 1's startup instructions:

1. Install dependencies if needed
2. Set up the environment — if Sub-agent 1 identified a `.env.example`, copy it to `.env` and fill in required values. Run database seeding or migrations if needed.
3. Start the dev server **in the background** (e.g., `npm run dev &`)
4. Poll for server readiness before continuing:
   ```bash
   for i in $(seq 1 30); do
     curl -s -o /dev/null -w "%{http_code}" http://localhost:PORT | grep -q "200\|304" && break
     sleep 1
   done
   ```
   Replace `PORT` with the actual port from Sub-agent 1's research.
5. Open the app with `playwright-cli open <url> -s=e2e-test` and confirm it loads. The `-s=e2e-test` flag creates a named session to prevent conflicts.
6. Take an initial screenshot: `playwright-cli screenshot --filename=e2e-screenshots/00-initial-load.png`

## Phase 3: Create Task List

Using the user journeys from Sub-agent 1 and findings from Sub-agent 3, create a task (using TaskCreate) for each user journey. Each task should include:

- **subject:** The journey name (e.g., "Test profile creation flow")
- **description:** Steps to execute, expected outcomes, database records to verify, and any related bug findings from Sub-agent 3
- **activeForm:** Present continuous (e.g., "Testing profile creation flow")

Also create a final task: "Responsive testing across devices."

## Phase 4: User Journey Testing

For each task, mark it `in_progress` with TaskUpdate and execute the following.

### 4a. Browser Testing

Use the Playwright CLI for all browser interaction. Commands are organized by function:

**Core Interaction:**
```
playwright-cli open <url> [-s=name] [--headed]    # Navigate + open session
playwright-cli snapshot                            # Get element refs (e2609)
playwright-cli click <ref>                         # Click element by ref
playwright-cli fill <ref> "text"                   # Clear field and type
playwright-cli type "text"                         # Type at current focus
playwright-cli select <ref> "option"               # Select dropdown option
playwright-cli check <ref>                         # Check a checkbox
playwright-cli uncheck <ref>                       # Uncheck a checkbox
playwright-cli press Enter                         # Press a key
playwright-cli screenshot --filename=<path>        # Save screenshot (named arg)
```

**Navigation:**
```
playwright-cli goto <url>                          # Navigate to URL
playwright-cli go-back                             # Browser back
playwright-cli go-forward                          # Browser forward
playwright-cli reload                              # Reload page
```

**Diagnostics:**
```
playwright-cli console                             # All console messages
playwright-cli console error                       # Only error-level messages
playwright-cli network                             # Network requests log
playwright-cli eval "() => location.href"          # Get current URL
playwright-cli eval "el => el.textContent" <ref>   # Get element text
```

**Tabs:**
```
playwright-cli tab-list                            # List open tabs
playwright-cli tab-new                             # Open new tab
playwright-cli tab-select <index>                  # Switch to tab
playwright-cli tab-close                           # Close current tab
```

**Tracing:**
```
playwright-cli tracing-start                       # Start recording trace
playwright-cli tracing-stop                        # Stop and save trace
```

**State Persistence:**
```
playwright-cli state-save <file>                   # Save browser state (cookies, storage)
playwright-cli state-load <file>                   # Restore browser state
```

**Session Management:**
```
playwright-cli close                               # End browser session
```

**Refs become invalid after navigation or DOM changes.** Always re-snapshot after page navigation, form submissions, or dynamic content updates (modals, tabs, theme changes).

For each step in a user journey:

1. Snapshot to get current refs
2. Perform the interaction
3. Wait and retry if needed — if an expected element is not found in the snapshot, wait 2-3 seconds and re-snapshot. DOM updates (animations, lazy loading, API responses) may not be instant.
4. **Take a screenshot** — save to a descriptive path under `e2e-screenshots/` organized by journey (e.g., `e2e-screenshots/profile-creation/03-form-submitted.png`)
5. **Analyze the screenshot** — use the Read tool to view the screenshot image. Check for visual correctness, UX issues, broken layouts, missing content, error states
6. Check `playwright-cli console error` and `playwright-cli network` periodically for JavaScript errors and failed API calls

**Key capabilities to leverage:**
- **Named sessions** (`-s=e2e-test`) — use throughout to prevent session conflicts
- **State persistence** — after completing login, run `playwright-cli state-save auth-state.json`. For subsequent journeys that need auth, run `playwright-cli state-load auth-state.json` instead of repeating login
- **Tracing** — run `playwright-cli tracing-start` at the beginning of each journey and `playwright-cli tracing-stop` at the end. Traces are invaluable for debugging failures
- **Network monitoring** — run `playwright-cli network` after interactions to watch for failed API calls (4xx/5xx)
- **Multi-tab testing** — use tab commands to test flows that open new tabs

Be thorough. Go through EVERY interaction, EVERY form field, EVERY button. The goal is that by the time this finishes, every part of the UI has been exercised and screenshotted.

### 4b. Database Validation

After any interaction that should modify data (form submits, deletions, updates):

1. Query the database to verify records. Use the environment variable from Sub-agent 2's research for the connection string and the schema docs to know what to check.
   - **Postgres:** use `psql` directly — e.g., `psql "$DATABASE_URL" -c "SELECT theme FROM profiles WHERE username = 'testuser'"`
   - **SQLite:** use `sqlite3` directly — e.g., `sqlite3 db.sqlite "SELECT theme FROM profiles WHERE username = 'testuser'"`
   - **Other databases:** write a small ad hoc script in the application's language, run it, then delete it
2. Verify:
   - Records created/updated/deleted as expected
   - Values match what was entered in the UI
   - Relationships between records are correct
   - No orphaned or duplicate records

### 4c. Issue Handling

When an issue is found (UI bug, database mismatch, JS error):

1. **Document it:** what was expected vs what happened, screenshot path, relevant DB query results
2. **Fix the code** — make the correction directly
3. **Re-run the failing step** to verify the fix worked
4. **Take a new screenshot** confirming the fix

### 4d. Responsive Testing

For the responsive testing task, revisit key pages using device presets:

- **Mobile:** `playwright-cli open <url> -s=e2e-test --device=iPhone12`
- **Tablet:** `playwright-cli open <url> -s=e2e-test --device=iPadPro11`
- **Desktop:** default viewport (no device flag needed)

At each device preset, screenshot every major page. Analyze for layout issues, overflow, broken alignment, and touch target sizes on mobile.

After completing each journey, mark its task as `completed` with TaskUpdate.

## Phase 5: Cleanup

After all testing is complete:
1. Stop any active traces: `playwright-cli tracing-stop`
2. Close the browser session: `playwright-cli close`
3. Stop the dev server background process:
   - **Linux/macOS/WSL:** `kill %1` or `kill $(lsof -t -i:PORT)`
   - **Windows:** `taskkill /F /PID <pid>` or `npx kill-port PORT`

## Phase 6: Report

### Text Summary (always output)

Present a concise summary:

```
## E2E Testing Complete

**Journeys Tested:** [count]
**Screenshots Captured:** [count]
**Traces Recorded:** [count]
**Issues Found:** [count] ([count] fixed, [count] remaining)

### Issues Fixed During Testing
- [Description] — [file:line]

### Remaining Issues
- [Description] — [severity: high/medium/low] — [file:line]

### Bug Hunt Findings (from code analysis)
- [Description] — [severity] — [file:line]

### Artifacts
- Screenshots: `e2e-screenshots/`
- Traces: saved via `tracing-stop` (load in Playwright Trace Viewer)
- Saved states: `auth-state.json` (and any other state files)
```

### Markdown Export (ask first)

After the text summary, ask the user:

> "Would you like me to export the full testing report to a markdown file? It includes per-journey breakdowns, all screenshot references, database validation results, and detailed findings — useful as context for follow-up fixes or GitHub issues."

If yes, write a detailed report to `e2e-test-report.md` in the project root containing:
- Full summary with stats
- Per-journey breakdown: steps taken, screenshots, database checks, issues found
- All issues with full details, fix status, and file references
- Bug hunt findings from the code analysis sub-agent
- Recommendations for any unresolved issues

## Quick Reference: agent-browser → playwright-cli

| agent-browser | playwright-cli | Notes |
|---|---|---|
| `open <url>` | `open <url> [-s=name]` | Named sessions prevent conflicts |
| `snapshot -i` | `snapshot` | Returns element refs like `e2609` |
| `click @eN` | `click <ref>` | Same concept, different ref format |
| `fill @eN "text"` | `fill <ref> "text"` | Same |
| `select @eN "opt"` | `select <ref> "opt"` | Same |
| `press Enter` | `press Enter` | Same |
| `screenshot <path>` | `screenshot --filename=<path>` | Named argument |
| `screenshot --annotate` | *(not available)* | Use `snapshot` + screenshot combo |
| `set viewport W H` | `--device=iPhone12` | Device presets or config file |
| `wait --load networkidle` | *(built-in)* | Navigation waits automatically |
| `console` | `console [level]` | Filter by level (error, warn) |
| `errors` | `console error` | Equivalent |
| `get text @eN` | `eval "el => el.textContent" <ref>` | Via eval |
| `get url` | `eval "() => location.href"` | Via eval |
| *(N/A)* | `network` | **New:** network request log |
| *(N/A)* | `tab-list/new/select/close` | **New:** multi-tab support |
| *(N/A)* | `tracing-start/stop` | **New:** trace recording |
| *(N/A)* | `state-save/load <file>` | **New:** state persistence |
| *(N/A)* | `goto <url>` | **New:** navigate without new session |
| *(N/A)* | `go-back/go-forward/reload` | **New:** browser navigation |
| `close` | `close` | Same |
