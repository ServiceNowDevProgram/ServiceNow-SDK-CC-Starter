---
name: git-issues-start
description: Fetch `ready`-labeled GitHub issues, analyze complexity, and implement each in a parallel isolated worktree. Use to kick off a batch implementation run.
user-invocable: true
---

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
- #3: Fix header alignment → src/client/components/Header.jsx
- #7: Add export action → src/client/components/Toolbar.jsx, src/client/services/ExportService.js

### Batch 2 (after Batch 1)
- #12: Add new seed fields → src/client/components/DetailPanel.jsx, src/fluent/seed-data.now.ts
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

   > You are fixing a bug / implementing a feature for this ServiceNow scoped application. Read CLAUDE.md (if present) at the repo root for project-specific context and conventions.
   >
   > ## Issue #<NUMBER>: <TITLE>
   > <BODY>
   >
   > ## Project Conventions (typical ServiceNow SDK starter — override per-project in CLAUDE.md)
   > - **Frontend components**: `src/client/components/` — typically paired files like `Name.jsx` + `Name.css`. Follow whatever pattern already exists in nearby files.
   > - **Frontend services**: `src/client/services/` — API/service modules. Follow existing patterns.
   > - **Server**: `src/server/script-includes/`, `src/server/rest-handlers/`, `src/server/rest-api/` — ServiceNow server-side JavaScript using GlideRecord and scoped-app patterns.
   > - **Fluent layer**: `src/fluent/` — TypeScript `.now.ts` files using `@servicenow/sdk` Fluent API (createTable, createBusinessRule, createScriptInclude, etc.).
   > - **Build/config**: `package.json`, root config files.
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

## Step 5: Gather changes into main working tree (uncommitted)

After all batches complete, transfer each worktree's changes into the main working tree as **uncommitted** changes so the user can review everything in one place before `/git-issues-end` commits and pushes.

1. List all worktrees: `git worktree list`

2. For each worktree (excluding the main working tree):
   - Stage all changes in the worktree so untracked files are included in the diff:
     ```
     git -C <worktree-path> add -A
     ```
   - Produce a patch of the staged changes (use `--binary` so binary assets apply cleanly):
     ```
     git -C <worktree-path> diff --cached --binary > /tmp/issue-<number>.patch
     ```
   - Apply the patch to the main working tree (without staging):
     ```
     git apply /tmp/issue-<number>.patch
     ```
     If `git apply` fails because of overlap with an earlier batch's changes, resolve manually using context from both issues.

3. **Do NOT stage, commit, or remove the worktrees here.** The user will review the uncommitted diff, and `/git-issues-end` will create commits, push, and clean up the worktrees.

4. Run `npm run build` on main to verify the combined uncommitted changes compile.
   - If the build fails, diagnose and fix. Leave the fix uncommitted alongside the rest — `/git-issues-end` will commit it.

## Step 6: Present summary

Print a results table:

| Issue | Title | Status | Files Changed | Build |
|-------|-------|--------|---------------|-------|

List any issues that were skipped (ambiguous, conflicted, or failed) with reasons.

All changes are now **uncommitted in the main working tree**. Worktrees are intact for `/git-issues-end` to clean up.

Next steps:
1. Review the uncommitted diff (`git status`, `git diff`) to validate each change
2. Optionally run `/deploy` against `dev` to smoke-test on an instance
3. If something is wrong, edit files directly or discard pieces with `git checkout -- <file>`; re-run `/git-issues-start <number>` for a specific issue if you want to redo it
4. When satisfied, run `/git-issues-end` — it will create issue-referencing commits, push, auto-close issues via `Closes #N`, and remove the worktrees

## Rules
- Do NOT ask for approval or confirmation at any point — just do the work
- Use `isolation: "worktree"` for each parallel agent to prevent conflicts
- If an issue is ambiguous or clearly depends on out-of-scope changes, skip it and flag it in the summary
- Prefer minimal, focused fixes — do not let agents refactor or over-engineer
- Do NOT commit, push, or merge — leave that for `/git-issues-end`
- Do NOT remove worktrees — `/git-issues-end` removes them after committing
- Agents inside worktrees must leave their changes uncommitted (see the agent prompt in Step 4)
