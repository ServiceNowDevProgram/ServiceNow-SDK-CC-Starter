# ServiceNow SDK — Claude Code Starter

A template repository containing reusable Claude Code skills for ServiceNow SDK development. Pull these into any new project to get a ready-made AI-assisted workflow out of the box.

## Usage

Copy the `skills/` folder into your project. You can use this Claude Code prompt to do it:

> Use gh repo clone and git sparse-checkout to pull only the skills/ folder from the private repo ServiceNowDevProgram/ServiceNow-SDK-CC-Starter into this project. Skip any files we already have in .claude/skills/. Do not copy READMEs.

## Skills

Skills are slash-command workflows you invoke in Claude Code (e.g., `/git-issues-start`). They work together to form an issue-driven development cycle.

### Issue Workflow

These three skills form a pipeline: interview an issue, work it, then wrap it up.

| Skill | What it does |
|-------|-------------|
| `/issue-interview` | Walks through a collaborative Q&A session on a GitHub issue to flesh out requirements, writes a spec into the issue body, and labels it `ready` for implementation. |
| `/git-issues-start` | Fetches all `ready`-labeled issues (or specific issue numbers), analyzes them, builds a parallelized execution plan, and launches isolated worktree agents to implement each one. Merges all changes into main when done. |
| `/git-issues-end` | Scans commits for issue references, pushes to remote, closes each issue, removes labels, and posts closing comments. |

### Utility

| Skill | What it does |
|-------|-------------|
| `/deploy` | Runs `npm run build` then `npm run deploy` in sequence. Stops if the build fails. |
| `/push` | Stages all changes, commits, and pushes to remote. |
| `/teams-issue-summary` | Generates a copy-paste-ready Microsoft Teams markdown table summarizing a range of GitHub issues — includes description, who requested it, and the result. |
