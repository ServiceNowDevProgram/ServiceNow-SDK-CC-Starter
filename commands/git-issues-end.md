Survey all worktree changes from a previous /git-issues-start run, commit them with proper issue-closing references, push, and clean up.

## Step 1: Survey changes

1. List all worktrees: `git worktree list`
2. For each worktree (excluding the main working tree):
   - Run `git -C <path> status` and `git -C <path> diff --stat`
   - Extract the issue number from the branch name (pattern: `issue-<number>-<slug>`)
   - Note all modified, added, or deleted files
3. Check the main working tree too: `git status` and `git diff --stat`

## Step 2: Map changes to issues

Cross-reference changed files with bot comments from /git-issues-start:
```
gh issue view <number> --comments
```

Build a mapping table:

| Issue | Branch | Files Changed | Overlaps With |
|-------|--------|---------------|---------------|

## Step 3: Decide commit strategy

Choose automatically based on how changes are structured:

- **Per-issue commits** (preferred): one commit per issue with `Closes #N`. Use when issues touch different files.
- **Grouped commits**: combine issues that share modified files into one commit referencing all: `Closes #N, Closes #M`. Use when files overlap.
- **Single commit**: everything in one commit. Use only when total changes are trivially small.

Log the chosen strategy and reasoning.

## Step 4: Create commits

For each commit:

1. Merge the worktree branch: `git merge <branch> --no-ff --no-edit`
   - Resolve any merge conflicts and note what was resolved.

2. If the auto-generated merge commit message does not match project style, amend it to follow the convention below.

**Commit message format:**
```
<Verb> <concise description of what changed>

Closes #N

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Style rules:
- Imperative verb + description (e.g., "Fix draft timer reset on page reload", "Add portfolio export button")
- No conventional-commit prefixes (no "feat:", "fix:", "chore:")
- No emojis
- `Closes #N` on its own line after a blank line
- `Co-Authored-By` on the final line

Example for a single issue:
```
Fix draft timer not resetting when resuming a session

Closes #3

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Example for grouped issues:
```
Update event scoring values and portfolio budget validation

Closes #8
Closes #11

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Use a HEREDOC to create the commit:
```bash
git commit -m "$(cat <<'EOF'
<message here>
EOF
)"
```

## Step 5: Verify build

Run `npm run build` to confirm the merged changes compile cleanly.

If the build fails:
- Diagnose and fix the issue
- Create an additional commit: "Fix build error after merging issue #N"
- Run `npm run build` again to confirm it passes

## Step 6: Push and comment

1. Push: `git push`
2. For each closed issue, comment with the commit hash and remove the `ready` label:
   ```
   gh issue comment <number> --body "Resolved in commit <hash>."
   gh issue edit <number> --remove-label "ready"
   ```

## Step 7: Clean up worktrees

For each worktree created by /git-issues-start:
```
git worktree remove <path>
git branch -d <branch>
```

If a branch can't be deleted with `-d` (not fully merged), investigate before using `-D`.

## Step 8: Final summary

Print a summary table:

| Issue | Title | Commit | Status |
|-------|-------|--------|--------|

Report totals: issues closed, commits created, branches cleaned up.

## Rules
- Stage specific files per commit where possible — avoid `git add -A` unless all remaining changes belong to the same commit
- If a file was changed for multiple issues, put it in the commit for the primary issue
- Always push after committing — the user calling /git-issues-end means they have reviewed the changes
- Do NOT close issues manually with `gh issue close` — the `Closes #N` in the commit message handles closure on push
- If some changes look wrong or incomplete, flag them to the user instead of committing
