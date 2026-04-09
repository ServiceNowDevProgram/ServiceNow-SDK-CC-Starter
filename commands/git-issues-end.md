Push committed changes to remote, close all issues that were worked on, remove labels, and post closing comments.

## Step 1: Identify issues that were worked on

Scan recent commits ahead of the remote for issue references:
```
git log --oneline --grep="Issue #" origin/main..HEAD
```

Extract all issue numbers from commits matching the `Issue #N` pattern.

If no issue-referencing commits are found ahead of origin/main, also check for open issues with the `ready` label as a fallback:
```
gh issue list --state open --label "ready" --json number,title --limit 100
```

If still no issues found, report that there are no issues to finalize and stop.

Present the list of issues to be closed:

| Issue | Title | Referenced in Commit |
|-------|-------|---------------------|

## Step 2: Push to remote

```
git push
```

If the push fails (e.g., behind remote), report the error and stop — do not force push.

## Step 3: Close issues and clean up

For each issue identified in Step 1:

1. Close the issue:
   ```
   gh issue close <number>
   ```

2. Remove the `ready` label:
   ```
   gh issue edit <number> --remove-label "ready"
   ```

3. Post a closing comment:
   ```
   gh issue comment <number> --body "Resolved and deployed. Closed via /git-issues-end."
   ```

## Step 4: Final summary

Print a summary table:

| Issue | Title | Status | Commit |
|-------|-------|--------|--------|

Report totals: issues closed, pushed to remote.

## Rules
- Do NOT force push under any circumstances
- Do NOT create new commits — all commits were already made by `/git-issues-start`
- If some issues look wrong or incomplete based on commit history, flag them to the user instead of closing
- Close issues via `gh issue close`, not via commit message keywords
