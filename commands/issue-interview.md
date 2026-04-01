Conduct a collaborative interview about a GitHub issue to flesh out requirements, then mark it ready for implementation.

Arguments: $ARGUMENTS (optional — a single issue number, e.g. "12"; if omitted, a list of open issues is shown first)

## Step 1: Determine which issue to work on

**If $ARGUMENTS is provided**, skip to loading that issue directly.

**If $ARGUMENTS is empty or not provided**, run:
```
gh issue list --state open --json number,title,labels --limit 50
```

Display the results as a numbered list:
```
Open issues:
  #<number>  <title>  [<labels>]
  ...
```

Then ask:
> Which issue would you like to talk through? (Enter the issue number)

Wait for the user to reply with a number, then use that as the issue number for the rest of the command.

## Step 2: Load the issue

Run:
```
gh issue view <issue-number> --json number,title,body,labels,comments
```

Read the full issue: title, body, existing labels, and any prior comments.

## Step 2.5: Research the codebase

Before talking to the user, investigate the codebase to understand what already exists that's relevant to this issue. This prevents asking the user questions you could answer yourself by reading the code.

Based on the issue title and body, identify keywords, component names, screen names, or feature areas mentioned. Then:

1. Search for relevant files using `Glob` and `Grep` — look for components, pages, scripts, or records related to the issue subject
2. Read the most relevant files to understand current behavior, data structures, and UI
3. Note what you found: what currently exists, how it works, and what gaps or uncertainties remain

Use this research to:
- Skip interview questions whose answers are already visible in the code
- Ask more specific, informed questions about gaps the code doesn't answer
- Identify affected files ahead of time so the spec can reference them

Do not ask the user to describe something you can see in the code.

## Step 3: Summarize and open the interview

Present a brief summary to the user:

```
## Issue #<N>: <title>

**Current description:**
<body or "(no description yet)">

**Labels:** <labels or "none">
```

Then say something like:
> Let's talk through this so we can get it fully specced before handing it off to an agent. I'll ask questions — answer as much or as little as you know, and say "done" when you're ready to finalize.

## Step 4: Interview loop

Ask questions one at a time (not all at once). Use your codebase research to skip questions whose answers are already clear from the code. Tailor remaining questions to what's genuinely missing or ambiguous. Cover:

- **Goal**: What is the user trying to accomplish? What problem does this solve?
- **Behavior**: What should happen? What should the UI/API/system do differently?
- **Scope**: What files or areas of the app are likely affected?
- **Acceptance criteria**: How will we know it's done? What does "working" look like?
- **Edge cases**: Any tricky scenarios, error states, or edge inputs to handle?
- **Out of scope**: Anything that might seem related but shouldn't be part of this issue?

Keep the conversation natural — if the user's answers make later questions irrelevant, skip them. If an answer raises a new question, ask it.

Stop asking when:
- The user says "done", "that's it", "ready", or similar
- You have enough to write a clear, actionable issue spec

## Step 5: Propose updated issue body

Draft a revised issue body using this structure:

```markdown
## Summary
<One or two sentences: what this is and why it matters>

## Desired behavior
<What should happen — be specific about UI, API responses, data changes, etc.>

## Acceptance criteria
- [ ] <criterion 1>
- [ ] <criterion 2>
- [ ] ...

## Notes
<Edge cases, out-of-scope items, or implementation hints — omit if none>
```

Show the draft to the user and ask:
> Does this look right? Any changes before I update the issue and mark it ready?

Incorporate any corrections the user gives, then confirm once more if changes were substantial.

## Step 6: Update the issue and label it ready

The agent that processes this issue reads only the issue title and body — not comments. All spec content must live in the body.

1. Update the issue body with the full approved spec:
   ```
   gh issue edit <issue-number> --body "<updated body>"
   ```

2. Add the `ready` label (create it if it doesn't exist):
   ```
   gh issue edit <issue-number> --add-label "ready"
   ```
   If the label doesn't exist yet, create it first:
   ```
   gh label create ready --color 0E8A16 --description "Fully specced and ready for implementation"
   ```

## Step 7: Confirm

Tell the user:
> Issue #<N> is updated and labeled **ready**. You can include it in the next `/git-issues-start` run.

## Rules

- Ask one question at a time — don't dump a list of questions at the user
- Don't make up details — only write what the user confirmed
- If the existing issue body is already thorough, say so and only ask about gaps
- Do not add the `ready` label until the user approves the final spec
- Keep the acceptance criteria concrete and testable, not vague
- Research the codebase before asking any questions — never ask the user to describe something you can read in the code
