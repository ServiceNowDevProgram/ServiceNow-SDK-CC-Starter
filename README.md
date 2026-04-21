# ServiceNow SDK — Claude Code Starter

A template repository containing reusable Claude Code skills for ServiceNow SDK development. Pull these into any new project to get a ready-made AI-assisted workflow out of the box.

## Usage

Copy the `skills/` folder into your project. You can use this Claude Code prompt to do it:

> Use gh repo clone and git sparse-checkout to pull only the skills/ folder from the private repo ServiceNowDevProgram/ServiceNow-SDK-CC-Starter into this project. Skip any files we already have in .claude/skills/. Do not copy READMEs.

## Skills

Skills are slash-command workflows you invoke in Claude Code (e.g., `/git-issues-start`). They work together to form an issue-driven development cycle.

### Issue Workflow

These skills form a pipeline: surface candidate issues, interview each one, work them, then wrap up.

> *(file issues manually)* or `/find-issues` → `/issue-interview` → `/git-issues-start` → `/git-issues-end`

| Skill | What it does |
|-------|-------------|
| *(manual)* | File issues directly on GitHub as single-sentence titles as things come up, or paste a bug-report message into Claude and ask it to convert each item into its own issue. This is the usual entry point — `/find-issues` below is only for surfacing unreported problems. |
| `/find-issues` | **(Optional)** Launches 5 parallel audit agents (bugs, inconsistencies, incomplete work, performance, UI/a11y) to scan the codebase, dedupes their findings, and lets you pick which ones to file as GitHub issues. Use intermittently to surface things no one has reported yet. |
| `/issue-interview` | Walks through a collaborative Q&A on a GitHub issue to flesh out requirements, writes a spec into the issue body, and labels it `ready` for implementation. Supports single-issue, `auto` (batch non-interactive), and `chain` (batch interactive) modes. Also checks the existing `ready` queue for overlapping scope. |
| `/git-issues-start` | Fetches all `ready`-labeled issues (or specific issue numbers), analyzes them, builds a parallelized execution plan, and launches isolated worktree agents to implement each one. Gathers each agent's changes into the main working tree as **uncommitted** changes for your review. |
| `/git-issues-end` | Surveys the uncommitted changes, picks a commit strategy (per-issue / grouped / single) based on file overlap, creates commits that reference their issues via `Closes #N`, pushes (with `--follow-tags`), comments on each issue, and cleans up agent worktrees. |

### Utility

| Skill | What it does |
|-------|-------------|
| `/deploy` | Builds and deploys to a ServiceNow instance. Accepts `dev`, `uat`, or `prod` (defaults to `dev`). Confirms before production, and creates an annotated git tag for UAT/prod deploys. |
| `/push` | Stages all changes, commits, and pushes to remote with `--follow-tags` (carries deploy tags along). |
| `/teams-issue-summary` | Generates a copy-paste-ready Microsoft Teams markdown table summarizing a range of GitHub issues — includes description, who requested it, and the result. |
