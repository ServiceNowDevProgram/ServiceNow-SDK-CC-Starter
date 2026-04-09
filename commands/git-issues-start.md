Fetch open GitHub issues labeled `ready` and work through them using worktree-isolated agents, then merge all changes back into main for immediate deployment.

Arguments: $ARGUMENTS (optional — space- or comma-separated issue numbers to limit scope, e.g. "3 7 12". Explicit numbers bypass the `ready` label requirement.)

## Step 1: Gather issues

If $ARGUMENTS is non-empty, fetch all open issues and filter to only the specified issue numbers (no label requirement):
```
gh issue list --state open --json number,title,body,labels,assignees --limit 100
```

If $ARGUMENTS is empty, fetch only issues labeled `ready`:
```
gh issue list --state open --label "ready" --json number,title,body,labels,assignees --limit 100
```

If no issues are found, stop and report that there are no `ready` issues to work on.

## Step 2: Analyze each issue

For every issue, determine:
- Which files likely need to change (use Grep/Glob to verify they exist)
- Complexity: small (1–2 files), medium (3–5 files), large (6+ files or cross-layer)
- Dependencies on other issues (shared files = must run sequentially)

Project area map:
- **Frontend components**: `src/client/components/` — React JSX + paired CSS (Name.jsx + Name.css)
- **Frontend services**: `src/client/services/` — API service modules (plain JS)
- **Frontend pages**: `src/client/` root — top-level page JSX + HTML entry points
- **Server logic**: `src/server/script-includes/`, `src/server/rest-handlers/`, `src/server/rest-api/` — ServiceNow server-side JS
- **Fluent definitions**: `src/fluent/` — TypeScript .now.ts files defining tables, seed data, business rules, navigation, form layouts
- **Fluent script includes**: `src/fluent/script-includes/` — .now.ts wrappers for server script includes
- **Build/config**: `package.json`, root config files

## Step 3: Build execution plan

Group issues into batches:
- Issues touching completely different files can run in parallel
- Issues sharing files must run sequentially within the same batch
- Order batches so dependencies resolve first

Log the full execution plan:

```
## Execution Plan

### Batch 1 (parallel)
- #3: Fix draft timer reset → src/client/components/DraftTimer.jsx
- #7: Add export button → src/client/components/PortfolioPanel.jsx, src/client/services/ExportService.js

### Batch 2 (after Batch 1)
- #12: Update scoring display → src/client/components/ScoreBar.jsx, src/fluent/seed-data.now.ts
⚠️ Sequential — shares seed-data.now.ts with a prior dependency
```

Proceed immediately without asking for approval.

## Step 4: Execute batches

For each batch:

1. Comment on each issue being started:
   ```
   gh issue comment <number> --body "Starting work on this issue."
   ```

2. For each issue in the batch, launch a sub-agent with `isolation: "worktree"` using this prompt structure:

   > You are fixing a bug / implementing a feature for AICTsdk — "AI Control Tower: CoE Draft & Defend," a competitive ServiceNow scoped application used at Knowledge 26.
   >
   > ## Issue #<NUMBER>: <TITLE>
   > <BODY>
   >
   > ## Project Conventions
   > - **Frontend**: React 18 JSX with plain CSS. Components live in `src/client/components/` as paired files: `Name.jsx` + `Name.css`. No TypeScript on the client. CSS uses plain classes — no modules, no CSS-in-JS. Import the CSS file at the top of the JSX file.
   > - **Services**: API modules in `src/client/services/` (plain JS). Follow existing patterns in nearby files.
   > - **Server**: ServiceNow Script Includes and REST handlers in `src/server/`. Server-side JavaScript using GlideRecord API and scoped app patterns.
   > - **Fluent layer**: TypeScript `.now.ts` files in `src/fluent/` using @servicenow/sdk 4.4.0 Fluent API (createTable, createBusinessRule, createScriptInclude, etc.).
   > - **CSS file size**: Keep individual CSS files under ~13KB to avoid SDK extraction bugs on Windows.
   >
   > ## Files to Read First
   > - <list of relevant files from your analysis>
   >
   > ## Task
   > 1. Read and understand the relevant code
   > 2. Make the minimal change needed to address this issue
   > 3. Do NOT refactor surrounding code or add unrelated improvements
   > 4. Run `npm run build` to verify compilation succeeds
   > 5. Do NOT commit — leave changes uncommitted in the worktree
   >
   > When done, report: which files were modified and a brief description of what changed.

3. Wait for all agents in the batch to finish before starting the next batch.

4. After each batch, comment results on each issue:
   ```
   gh issue comment <number> --body "$(cat <<'EOF'
   Changes ready for review.

   **Files modified:**
   - `path/to/file`: what changed

   **Approach:**
   Brief explanation of the fix/implementation.

   **Testing notes:**
   What to verify after deploying with `npm run deploy`.
   EOF
   )"
   ```

## Step 5: Merge all changes into main

After all batches complete:

1. List all worktrees: `git worktree list`

2. For each worktree (excluding the main working tree):
   - Extract the issue number from the branch name (pattern: `issue-<number>-<slug>`)
   - Stage all changes in the worktree:
     ```
     git -C <path> add -A
     ```
   - Commit in the worktree:
     ```
     git -C <path> commit -m "$(cat <<'EOF'
     Issue #<number>: <concise description of what changed>

     Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
     EOF
     )"
     ```

3. Determine merge order — if worktrees have overlapping files, merge sequentially and resolve conflicts.

4. For each worktree branch, merge into main:
   ```
   git merge <branch> --no-ff --no-edit
   ```
   If a merge conflict occurs, resolve it using context from both issues and note the resolution in the summary.

   **Important:** Do NOT use `Closes #N` or `Fixes #N` anywhere — issues must stay open until the user validates and runs `/git-issues-end`.

5. Clean up each worktree after merging:
   ```
   git worktree remove <path>
   git branch -d <branch>
   ```

6. Run `npm run build` on main to verify everything compiles cleanly.
   - If the build fails, diagnose and fix, then create an additional commit: "Fix build error after merging issue #<number>"
   - Run `npm run build` again to confirm it passes.

## Step 6: Present summary

Print a results table:

| Issue | Title | Status | Files Changed | Build |
|-------|-------|--------|---------------|-------|

List any issues that were skipped (ambiguous, conflicted, or failed) with reasons.

All changes are now in main and worktrees have been cleaned up.

Next steps:
1. Run `/deploy` to build and deploy to the ServiceNow instance
2. Test each change on the instance
3. If further changes are needed, re-run `/git-issues-start` with specific issue numbers (e.g. `/git-issues-start 3 7`)
4. When all changes are verified, run `/git-issues-end` to push and close issues

## Rules
- Do NOT ask for approval or confirmation at any point — just do the work
- Use `isolation: "worktree"` for each parallel agent to prevent conflicts
- If an issue is ambiguous or clearly depends on out-of-scope changes, skip it and flag it in the summary
- Prefer minimal, focused fixes — do not let agents refactor or over-engineer
- Do NOT push to remote — leave that for `/git-issues-end`
- Do NOT use `Closes` or `Fixes` in commit messages — issues must stay open until the user explicitly runs `/git-issues-end`
- Always clean up worktrees and branches after merging into main
