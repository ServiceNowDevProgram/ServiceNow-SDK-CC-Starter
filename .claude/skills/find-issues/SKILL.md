---
name: find-issues
description: Audit the codebase for bugs, inconsistencies, incomplete work, performance problems, and UI issues using a parallel agent team. Present findings and let the user pick which to file as GitHub issues.
user-invocable: true
---

Audit the codebase using a parallel agent team, present the findings, and let the user choose which ones to file as GitHub issues.

This is the first step in the issue workflow:
**`/find-issues`** → `/issue-interview` → `/git-issues-start` → `/git-issues-end`

---

## Step 1: Launch parallel audit agents

Dispatch **5 Explore agents simultaneously** — one per audit category. Each agent should search thoroughly and return a list of specific, actionable findings. Do not wait for one to finish before starting another; launch all 5 in a single message.

Before dispatching, briefly scan the repo root (CLAUDE.md if present, `package.json`, top-level `src/` layout) so each agent's prompt can reference the actual frameworks, languages, and project structure in use. Tailor the "Search for" lists below to what the project actually uses (React vs. other UI frameworks, TypeScript vs. plain JS, ServiceNow SDK vs. plain Node, etc.).

---

### Agent 1 — Bugs & edge cases

> Audit this codebase for bugs and edge cases. Read CLAUDE.md (if present) and skim the top-level structure first for context.
>
> Search for:
> - Null/undefined access without guards (optional chaining missing, properties read off values that could be null)
> - Functions that can return null/undefined/false but whose callers don't check
> - Off-by-one errors in loops, slicing, or indexing
> - Unhandled promise rejections and missing `catch` on async operations
> - State that can be stale or inconsistent across navigation / lifecycle boundaries (e.g. values not reset on mount/unmount)
> - Edge cases in save/load, serialization, or API boundaries (missing fields, version mismatches, unexpected shapes)
> - Arithmetic that can produce negative, NaN, or divide-by-zero results in user-visible state
> - `try/catch` blocks whose errors are silently swallowed
>
> For each finding, report:
> - File path and line number
> - What the bug or risk is
> - What scenario triggers it
>
> Be specific and concrete. Skip findings that are clearly intentional defensive patterns.

---

### Agent 2 — Inconsistencies across modules

> Audit this codebase for inconsistencies — patterns that exist in some places but are missing where they should also exist. Read CLAUDE.md (if present) first for context.
>
> Check for:
> - Components/pages that use a shared pattern (e.g. loading state, error boundary, keyboard shortcut, focus management) in some places but not in equivalent ones
> - Cleanup logic (unsubscribing, aborting fetches, clearing timers) present in some components but missing in others with the same lifecycle needs
> - API calls that have retry/error handling in some services but not in equivalent ones
> - Log/instrumentation calls that appear in some flows but are missing in equivalent flows
> - Data records missing fields that peers of the same type have
> - State writes without matching resets (feature A resets on nav-away; feature B doesn't)
>
> For each finding, report:
> - File path(s) and relevant line numbers
> - What pattern exists elsewhere and where it's missing
> - Why it likely matters (silent failure, user-facing inconsistency, resource leak)

---

### Agent 3 — Incomplete features & TODOs

> Audit this codebase for incomplete or placeholder work. Read CLAUDE.md (if present) first for context.
>
> Search for:
> - Comments containing TODO, FIXME, HACK, TEMP, PLACEHOLDER, XXX, or similar markers
> - Functions or code paths that just `return`, `return null`, or `throw new Error("not implemented")` with no real implementation
> - Stubbed mocks/fixtures still wired into production paths
> - Strings like "TBD", "TODO", "placeholder", "lorem ipsum", or empty user-facing copy in data/config files
> - Features described in planning docs (`.planning/`, `docs/`, README) that have no corresponding implementation (spot-check a few)
> - Modules wired up in theory but never imported/called anywhere
>
> For each finding, report:
> - File path and line number (or planning-doc file + section for spec drift)
> - What is incomplete or missing
> - Rough size: small gap vs. large unimplemented feature

---

### Agent 4 — Performance

> Audit this codebase for performance problems. Read CLAUDE.md (if present) first for context.
>
> Search for:
> - Expensive work done inside render/update paths that should be memoized (object/array literals re-created every render, `useMemo`/`useCallback` missing on hot paths, heavy computation inside a render body)
> - Effects (`useEffect` or equivalent) that run more often than needed due to unstable dependencies
> - Network calls or file I/O triggered on every keystroke or mousemove without debouncing
> - `require`/dynamic import inside hot paths that should be hoisted
> - Large lists rendered without virtualization
> - String/array concatenation in tight loops where a builder pattern would be better
> - N+1 query / fetch patterns
>
> For each finding, report:
> - File path and line number
> - What the per-render or per-call cost is
> - Suggested fix (memoize, hoist, debounce, virtualize, batch, etc.)

---

### Agent 5 — UI & accessibility

> Audit this codebase for UI, layout, and accessibility problems. Read CLAUDE.md (if present) first for context.
>
> Search for:
> - Text that can overflow its container (long titles, large numbers rendered without width limits or truncation)
> - Hardcoded pixel positions that break under different viewport sizes or content lengths
> - Interactive elements missing `aria-*` attributes, roles, or keyboard handlers
> - `onClick` on non-button elements without `role="button"` + keyboard event support
> - Color/contrast choices that rely on color alone to convey state
> - Forms without labels, focus outlines disabled without replacement, or inputs without associated error text
> - Missing `alt` on meaningful images, or decorative images without `alt=""`
> - Hit areas (click/hover rects) that don't match visual bounds
> - Missing hover/focus states on interactive elements
>
> For each finding, report:
> - File path and line number
> - What the visual or interaction problem is
> - When/how it manifests (specific screen, viewport, input method)

---

## Step 2: Collect & deduplicate findings

Once all 5 agents return, compile their findings into a single numbered list. Remove duplicates (same file+line reported by multiple agents). Group by category.

Format each finding as:

```
[N] <Category> — <File>:<line>
    <One-sentence description of the problem>
    Trigger: <when/how this manifests>
```

Example:
```
[1] Bug — src/client/services/ExportService.js:84
    Download can fail silently if the blob builder rejects — the catch swallows the error.
    Trigger: Browser denies filesystem write during export.

[2] Inconsistency — src/client/components/DetailPanel.jsx
    Missing loading state — other panels in src/client/components/ show a spinner while fetching.
    Trigger: User opens the detail view before the initial fetch resolves.
```

If an agent returned zero findings in a category, still include that category in the summary as "Clean — no findings" rather than omitting it.

---

## Step 3: Present findings to user

Print the full numbered list, then ask:

> Found **N potential issues** across 5 audit categories above.
>
> Which ones would you like to file as GitHub issues? You can:
> - Enter numbers: `1 3 7` to select specific ones
> - Enter a range: `1-5`
> - Enter `all` to file everything
> - Enter `none` to skip (just review)
>
> For each filed issue, I'll create a draft GitHub issue. You can then run `/issue-interview <number>` to flesh it out before handing it to `/git-issues-start`.

Wait for the user's response.

---

## Step 4: Create GitHub issues

For each selected finding, create a GitHub issue:

```
gh issue create \
  --title "<concise title derived from finding>" \
  --body "$(cat <<'EOF'
## Finding

<Full description from the audit finding>

**File:** `<path>:<line>`
**Category:** <Bug / Inconsistency / Incomplete / Performance / UI>
**Trigger:** <when/how it manifests>

## Notes

Discovered by `/find-issues` audit. Run `/issue-interview <number>` to flesh out acceptance criteria before implementing.
EOF
)"
```

After creating each issue, print its URL and number.

---

## Step 5: Summary

Print a table of all created issues:

| # | GitHub Issue | Category | Title |
|---|--------------|----------|-------|
| 1 | #42 | Bug | Export service swallows filesystem errors |
| 3 | #43 | Inconsistency | Portfolio panel missing loading state |

Then suggest next steps:
> Run `/issue-interview <number>` on any issue that needs more detail before implementation, then `/git-issues-start` when you're ready to build.

---

## Rules
- Launch all 5 audit agents in parallel — do not run them sequentially
- Each agent is read-only: Explore only, no edits
- Be specific: vague findings like "could be improved" are not useful — every finding needs a file, a line (where possible), and a concrete trigger
- Deduplicate before presenting — the same bug reported by two agents should appear once
- Do not create GitHub issues until the user selects which findings to file
- Do not add the `ready` label — that's for `/issue-interview` to do after spec review
- If agents return no findings in a category, note that category as clean rather than omitting it
