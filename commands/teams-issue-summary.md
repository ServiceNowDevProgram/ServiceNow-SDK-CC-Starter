Generate a Microsoft Teams message with a summary table of GitHub issues in a given range or list.

Arguments: $ARGUMENTS — a range like `10-20`, a comma-separated list like `10,12,15`, or a mix like `10-12,15,18`

## Step 1: Parse the issue numbers

Expand `$ARGUMENTS` into an ordered list of individual issue numbers:
- `10-20` → 10, 11, 12, … 20
- `10,12,15` → 10, 12, 15
- `10-12,15,18` → 10, 11, 12, 15, 18

If `$ARGUMENTS` is empty, ask the user:
> What issue range or numbers would you like to summarize? (e.g. `10-20` or `10,12,15`)

## Step 2: Fetch each issue

For each issue number, run:
```
gh issue view <N> --json number,title,body,state,author,comments,url,closedAt
```

If an issue doesn't exist or the command fails, skip it and note it as "Not found" in the table.

## Step 3: Extract the four fields for each issue

For each issue, determine:

### Issue # (with link)
Format as a markdown hyperlink: `[#N](url)`

### Short description
Write a 5–10 word summary capturing the core of the issue. Use the title as the starting point, then refine it with context from the body if needed. Keep it plain — no jargon, no markdown formatting in the cell.

### Who originally vocalized the request
Use this priority order:
1. Look for a line in the issue body matching `Requested by:`, `Requested By:`, `Feedback from:`, or `Feedback From:` — use whatever name or handle follows (strip parenthetical context like dates)
2. If not found but the issue body references or builds on another issue's feedback, attribute it to the same person who vocalized the original request
3. If neither applies, use `@<author.login>` (the GitHub user who opened the issue)

### Result
- **If closed**: Synthesize a brief outcome (under 10 words) from the last comment or closing comment. If a PR was linked, note "Shipped: <short description>". Examples: "Fixed styling on draft board card", "Shipped: new intake form layout"
- **If open**: Write "Open" or "In progress" depending on whether any activity has happened

## Step 4: Build the Teams message

Output the following as plain text the user can copy directly into Teams. Do NOT wrap it in a code block — Teams renders markdown natively.

```
## Issue Summary

| Issue | Description | Requested By | Result |
|-------|-------------|--------------|--------|
| [#N](url) | Short description | @handle | Result text |
```

Add a blank line before the table and after. If any issues were skipped (not found), add a note below the table:
```
> Issues not found: #X, #Y
```

## Step 5: Confirm

After outputting the message, tell the user:
> Ready to paste into Teams. The table uses standard markdown — it will render in a Teams channel or chat message.

## Rules

- Keep description and result cells short — Teams table cells don't wrap gracefully
- Do not invent details that aren't in the issue body or comments
- If the issue body is empty and the title is vague, use the title as-is for the description
- Preserve the order the user specified (range order for ranges, list order for lists)
- Do not add the `ready` label or modify any issues — this is read-only
