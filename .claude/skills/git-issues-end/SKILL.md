---
name: git-issues-end
description: Survey changes, create commits referencing their issues, push to remote, comment on each issue, and clean up agent worktrees. Use after the user has reviewed the changes from `/git-issues-start`.
user-invocable: true
---

After the user has reviewed changes from `/git-issues-start`, commit all changes with proper issue-closing references, push, comment on each issue, and clean up agent worktrees.

## Phase 1: Survey the changes

1. **Check working tree state:**
   ```
   git status
   git diff
   git diff --stat
   ```

2. **Fetch the open issues** to map changes back to issue numbers:
   ```
   gh issue list --state open --limit 50 --json number,title
   ```

3. **Read recent issue comments** to recall what was done for each issue:
   ```
   gh issue view <NUMBER> --comments
   ```
   Look for the bot comments posted by `/git-issues-start` that document what files were changed for which issue.

4. **Map each changed file to its issue(s).** Build a table like:
   ```
   | File                                    | Issue(s)   |
   |-----------------------------------------|------------|
   | src/client/components/Header.jsx        | #3         |
   | src/client/components/Toolbar.jsx       | #7         |
   | src/client/services/ExportService.js    | #7         |
   | src/client/components/DetailPanel.jsx   | #12        |
   | src/fluent/seed-data.now.ts             | #12, #15   |
   ```

## Phase 2: Decide commit strategy

Choose the best approach based on how the changes are structured:

**Option A: One commit per issue** (preferred when files don't overlap)
- Groups files by issue
- Each commit references its issue with `Closes #N`
- Clean history, easy to revert individual fixes

**Option B: Grouped commits** (when files overlap across issues)
- Group related issues that share files into a single commit
- Reference all related issues: `Closes #18, Closes #19`

**Option C: Single commit** (when changes are small or tightly coupled)
- One commit referencing all issues
- Simpler but harder to revert individually

Pick the best option automatically based on file overlap — no need to ask the user.

## Phase 3: Create commits

For each commit, use a clear imperative summary and reference the issue(s) it closes:

```
<short imperative summary>

<what changed and why>

Closes #N

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

**Steps for each commit:**
1. Stage only the files belonging to that commit: `git add <specific files>`
2. Create the commit with a HEREDOC:
   ```bash
   git commit -m "$(cat <<'EOF'
   Fix portfolio export button missing on mobile

   Wire up the export handler on small viewports — the button was
   rendered but the onClick was gated behind the desktop breakpoint.

   Closes #7

   Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
   EOF
   )"
   ```
3. Verify with `git log -1`

## Phase 4: Push and comment

After all commits are created:

1. **Push immediately:**
   ```
   git push --follow-tags
   ```
   The user calling `/git-issues-end` means they've reviewed and approved the changes. `--follow-tags` carries any deploy tags along. The `Closes #N` references in the commits will auto-close the linked issues as soon as the push lands on the default branch.

2. **Comment on each issue** with the final resolution:
   ```
   gh issue comment <NUMBER> --body "Fix committed and pushed: <short commit hash>"
   ```

3. **Show the final commit log:**
   ```
   git log --oneline -<N>
   ```

## Phase 5: Cleanup

1. **List all worktrees:**
   ```
   git worktree list
   ```

2. **Remove any agent worktrees** (paths matching `.claude/worktrees/agent-*` or similar). Use `--force` because the worktrees contain the uncommitted changes that were already applied to main:
   ```
   git worktree remove --force <worktree-path>
   ```
   Run this for each leftover agent worktree.

3. Confirm with a final `git worktree list` — only the main worktree should remain.

## Rules
- Use `Closes #N` (not `Fixes #N`) for consistency — GitHub accepts both
- Stage specific files per commit — never use `git add -A` or `git add .`
- If a file was changed for multiple issues, put it in the commit for the primary issue
- Always push after committing — the user has already reviewed the changes
- Do NOT manually close issues with `gh issue close` — let the `Closes #N` in the commit message handle it on push
- If some changes look wrong or incomplete, flag them to the user instead of committing
