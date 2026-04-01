Fetch open GitHub issues labeled `ready` and work through them systematically using worktree-isolated agents for parallel work.

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

To understand the project structure, read CLAUDE.md (if it exists) and explore the repo's directory layout using Glob/Grep. Identify which areas of the codebase each issue is likely to touch.

## Step 3: Build execution plan

Group issues into batches:
- Issues touching completely different files can run in parallel
- Issues sharing files must run sequentially within the same batch
- Order batches so dependencies resolve first

Log the full execution plan:

```
## Execution Plan

### Batch 1 (parallel)
- #3: Fix timer reset → src/components/Timer.js
- #7: Add export button → src/components/Panel.js, src/services/Export.js

### Batch 2 (after Batch 1)
- #12: Update display logic → src/components/Score.js, src/config/settings.js
⚠️ Sequential — shares settings.js with a prior dependency
```

Proceed immediately without asking for approval.

## Step 4: Execute batches

For each batch:

1. Comment on each issue being started:
   ```
   gh issue comment <number> --body "Starting work on this issue."
   ```

2. For each issue in the batch, launch a sub-agent with `isolation: "worktree"` using this prompt structure:

   > ## Issue #<NUMBER>: <TITLE>
   > <BODY>
   >
   > ## Project Context
   > Read CLAUDE.md at the repo root for project description and conventions. If it doesn't exist, explore the repo structure to understand the tech stack and patterns before making changes.
   >
   > ## Files to Read First
   > - <list of relevant files from your analysis>
   >
   > ## Task
   > 1. Read CLAUDE.md and understand the project conventions
   > 2. Read and understand the relevant code
   > 3. Make the minimal change needed to address this issue
   > 4. Do NOT refactor surrounding code or add unrelated improvements
   > 5. Run `npm run build` to verify compilation succeeds
   > 6. Do NOT commit — leave changes uncommitted in the worktree for review
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

## Step 5: Present summary

Print a results table:

| Issue | Title | Status | Files Changed | Build |
|-------|-------|--------|---------------|-------|

List any issues that were skipped (ambiguous, conflicted, or failed) with reasons.

Provide testing instructions:
1. Review changes in each worktree (list worktree paths)
2. For each worktree, run `npm run build` to verify compilation
3. Deploy to test the changes
4. When satisfied, run `/git-issues-end` to commit, push, and clean up

## Rules
- Do NOT ask for approval or confirmation at any point — just do the work
- Use `isolation: "worktree"` for each parallel agent to prevent conflicts
- If an issue is ambiguous or clearly depends on out-of-scope changes, skip it and flag it in the summary
- Prefer minimal, focused fixes — do not let agents refactor or over-engineer
- Do NOT commit — leave that for `/git-issues-end`
