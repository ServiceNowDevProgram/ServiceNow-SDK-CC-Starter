---
name: issue-interview
description: Conduct a Q&A session on a GitHub issue, write a structured spec into the issue body, and label it `ready`. Use to prepare an issue before implementation. Supports single-issue (default), `auto` (non-interactive batch), and `chain` (interactive batch) modes.
user-invocable: true
---

Conduct a collaborative interview about a GitHub issue to flesh out requirements, then mark it ready for implementation.

Arguments: $ARGUMENTS (optional — a single issue number e.g. "12", `auto` to process all open non-ready issues non-interactively, `auto 10,15,20-25` to auto-process a specific set, or `chain` / `chain 10,15,20-25` to interactively interview a batch one-by-one)

## Step 1: Determine which issue to work on

**If $ARGUMENTS is a number**, skip to loading that issue directly.

**If $ARGUMENTS starts with "auto"**, enter Auto Mode (see below).

**If $ARGUMENTS starts with "chain"**, enter Chain Mode (see below).

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

---

## Auto Mode

When $ARGUMENTS starts with "auto", enter Auto Mode.

**Building the queue** depends on what follows "auto":

- **`auto` (nothing after):** Fetch all open issues, filter out any with the `ready` label, sort ascending — that is your queue.
  ```
  gh issue list --state open --json number,title,labels --limit 100
  ```

- **`auto <list>` (numbers/ranges after "auto"):** Parse the list — it may contain individual numbers and/or ranges separated by commas (e.g. `10,15,20-25`). Expand ranges into individual numbers. Fetch those specific issues (load each with `gh issue view`). Filter out any that already have the `ready` label. Sort ascending — that is your queue. Do not fetch all open issues; only the specified ones.

  Examples of valid list formats:
  - `auto 170` → issues [170]
  - `auto 10,15,20` → issues [10, 15, 20]
  - `auto 20-25` → issues [20, 21, 22, 23, 24, 25]
  - `auto 10,15,20-25` → issues [10, 15, 20, 21, 22, 23, 24, 25]

Announce the queue to the user:
> Auto mode: found **N** issues to process. Working through them now — I'll only stop if I have a question for you.

Then for each issue in the queue, run the full workflow (Steps 2 through 7 below) with these modifications:

### Auto Mode rules

**Self-sufficiency first:** Before deciding to ask the user anything, exhaust all self-service options:
- Read the issue body thoroughly
- Research the codebase (Step 2.5) — read affected files, grep for related code
- Derive acceptance criteria from the issue description + code evidence

**Only pause and ask the user if:**
- The desired behavior is genuinely ambiguous and cannot be inferred from the issue + code (e.g. two reasonable interpretations exist and the choice has UX or scope implications)
- The issue touches a design decision that requires user intent (e.g. "should this also affect X, or just Y?")
- A critical file or system referenced in the issue doesn't exist and you can't determine the right approach

**Do NOT pause to ask about:**
- Things already stated in the issue body
- Things visible in the code (file paths, function names, current behavior)
- Confirmation of obvious acceptance criteria for clearly-scoped bugs
- Whether to proceed — always proceed unless blocked

**When pausing:** Ask the single most important blocking question, wait for the answer, then continue without asking follow-ups unless truly necessary.

**Skipping user approval of the draft spec:** In auto mode, do not show the draft and ask "Does this look right?" before updating. Write the spec, update the issue, add the `ready` label, and move on. The user can always edit issues manually later.

**Progress reporting:** After each issue is marked ready, print a one-line status:
```
✓ #<N> ready — <title>
```

**After the queue is exhausted**, print a summary:
```
Auto mode complete. Marked N issues ready:
  #<N1> — <title>
  #<N2> — <title>
  ...

Paused for input on: #<Nx> — <question asked>   (omit if none)
```

## Chain Mode

When $ARGUMENTS starts with "chain", enter Chain Mode. Chain Mode runs the **full interactive workflow** (Steps 2 through 7) on each issue in a queue — the same questions, draft approval, and title prompt as single-issue mode. The only difference from single-issue mode is that after finishing one issue, it moves on to the next instead of ending.

**Building the queue** works exactly like Auto Mode:

- **`chain` (nothing after):** Fetch all open issues, filter out any with the `ready` label, sort ascending — that is your queue.
  ```
  gh issue list --state open --json number,title,labels --limit 100
  ```

- **`chain <list>` (numbers/ranges after "chain"):** Parse the list — it may contain individual numbers and/or ranges separated by commas (e.g. `10,15,20-25`). Expand ranges into individual numbers. Fetch those specific issues (load each with `gh issue view`). Filter out any that already have the `ready` label. Sort ascending — that is your queue.

  Examples of valid list formats:
  - `chain 170` → issues [170]
  - `chain 10,15,20` → issues [10, 15, 20]
  - `chain 20-25` → issues [20, 21, 22, 23, 24, 25]
  - `chain 10,15,20-25` → issues [10, 15, 20, 21, 22, 23, 24, 25]

Announce the queue to the user:
> Chain mode: **N** issues queued. I'll walk through each one with a full interview. Say "stop" at any inter-issue prompt to end the chain early.

Then for each issue in the queue:

1. Run Steps 2 through 7 in full — **no auto-mode shortcuts**. Ask questions one at a time, show the draft spec, get user approval, prompt for title rename if needed.
2. After Step 7 completes for that issue, print:
   ```
   ✓ #<N> ready — <title>    (<X> of <N> complete, <Y> remaining)
   ```
3. If more issues remain in the queue, ask:
   > Ready to move on to #<next>: "<next title>"? (y = continue / n = skip this issue / stop = end chain)

   - **y** → proceed to Step 2 for the next issue
   - **n** → skip that next issue (do not interview or mark it ready), then ask about the one after if any remain
   - **stop** → exit chain mode immediately and jump to the final summary

**After the queue is exhausted (or the user typed "stop"):**
```
Chain mode complete. Marked N issues ready:
  #<N1> — <title>
  #<N2> — <title>
  ...

Skipped: #<Nx>, #<Ny>   (omit this line if none were skipped)
Stopped early at: #<Nz>   (omit this line if the user didn't stop early)
```

---

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

## Step 2.6: Check for overlapping ready issues

Other issues already labeled `ready` are queued for implementation. If this issue overlaps with one of them, the implementing agent could duplicate work, cause merge conflicts, or contradict a decision already locked in. Surface that before the interview.

Fetch the currently-ready queue:
```
gh issue list --state open --label ready --json number,title,body --limit 100
```

Scan each for overlap with the current issue. Signals that matter:
- Touches the same files/systems identified in Step 2.5
- Modifies the same subsystem, data table, or UI component
- Has acceptance criteria that could collide or re-state part of this issue's scope
- References the same feature area

For each overlap found, note:
- Issue number + title
- Which area overlaps (e.g. "both edit `src/components/Header.tsx`", "both rework the login flow")
- Whether the overlap is **redundant** (this issue may already be covered), **adjacent** (related but distinct — flag for coordination), or **conflicting** (decisions disagree — needs resolution)

Carry these findings into the rest of the workflow:
- **During the interview (Step 4):** mention overlaps to the user and ask how to handle them (merge, defer, scope-split) — except in auto mode, where you only pause if the overlap is conflicting or would make this issue redundant
- **In the spec (Step 5):** add a `## Related issues` section listing each overlap with a one-line note on the relationship

If no overlaps exist, skip silently.

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

## Related issues
- #<N> — <title> (<relationship: redundant / adjacent / conflicting> — <one-line note>)

## Notes
<Edge cases, out-of-scope items, or implementation hints — omit if none>
```

Omit the `## Related issues` section entirely if Step 2.6 found no overlaps.

Show the draft to the user and ask:
> Does this look right? Any changes before I update the issue and mark it ready?

Incorporate any corrections the user gives, then confirm once more if changes were substantial.

## Step 5.5: Check title adequacy

Compare the current issue title against the approved spec (Summary + Desired behavior + Acceptance criteria).

Ask yourself: *If someone read only this title, would they understand all of the goals this issue now covers?*

A title is **inadequate** if any of these are true:
- It describes only one of multiple goals the spec now covers (e.g. title says "fix X" but the spec also reworks Y)
- It's vague or generic ("improve the page", "fix bug") when the spec is specific
- It references outdated terminology that no longer matches the spec
- It's missing the subsystem/scope prefix used by other issues in this repo (check recent titles via `gh issue list` if unsure of the convention)

If the title is adequate, keep it and continue to Step 6.

If the title is **inadequate**, draft a new title that:
- Fits in ~70 characters
- Names the primary subsystem/area
- Captures the full scope of what will change (not just one sub-goal)
- Matches the style of other recent issue titles in the repo

**Outside auto mode:** show the old and new title side-by-side and ask:
> The current title doesn't fully cover the agreed scope. I'd like to rename it to:
> **\<new title\>**
> Okay to rename? (y/n)

**In auto mode:** rewrite the title without asking, and include it in the progress line (e.g. `✓ #12 ready — <new title>`).

Carry the approved new title (if any) into Step 6.

## Step 6: Update the issue and label it ready

The agent that processes this issue reads only the issue title and body — not comments. All spec content must live in the body.

1. Update the issue body (and title, if Step 5.5 produced a new one) with the full approved spec:
   ```
   gh issue edit <issue-number> --body "<updated body>" [--title "<new title>"]
   ```
   Include `--title` only if Step 5.5 determined the old title was inadequate.

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
- Do not add the `ready` label until the user approves the final spec (except in auto mode)
- Keep the acceptance criteria concrete and testable, not vague
- Research the codebase before asking any questions — never ask the user to describe something you can read in the code
- Always check the `ready` queue (Step 2.6) before the interview — overlapping scope must be flagged, not silently re-specced
- In auto mode: bias heavily toward self-sufficiency — only pause when genuinely blocked, not out of courtesy
- In chain mode: run the full interactive workflow for each issue; only the queueing and inter-issue confirmation prompts are new behaviors
