# ServiceNow SDK — Claude Code Starter

A template repository containing reusable Claude Code commands and skills for ServiceNow SDK development. Pull these into any new project to get a ready-made AI-assisted workflow out of the box.

## Usage

Copy the `commands/` and `skills/` folders into your project. You can use this Claude Code prompt to do it:

> Use gh repo clone and git sparse-checkout to pull only the commands/ and skills/ folders from the private repo ServiceNowDevProgram/ServiceNow-SDK-CC-Starter into this project. Skip any files we already have in .claude/commands/. Do not copy READMEs.

## Commands

Commands are slash-command workflows you invoke in Claude Code (e.g., `/git-issues-start`). They work together to form an issue-driven development cycle.

### Issue Workflow

These three commands form a pipeline: interview an issue, work it, then wrap it up.

| Command | What it does |
|---------|-------------|
| `/issue-interview` | Walks through a collaborative Q&A session on a GitHub issue to flesh out requirements, writes a spec into the issue body, and labels it `ready` for implementation. |
| `/git-issues-start` | Fetches all `ready`-labeled issues (or specific issue numbers), analyzes them, builds a parallelized execution plan, and launches isolated worktree agents to implement each one. Leaves changes uncommitted for review. |
| `/git-issues-end` | Surveys all worktree changes from a `/git-issues-start` run, commits them with `Closes #N` references, pushes, comments on each issue, and cleans up worktrees and branches. |

### Utility Commands

| Command | What it does |
|---------|-------------|
| `/deploy` | Runs `npm run build` then `npm run deploy` in sequence. Stops if the build fails. |
| `/push` | Stages all changes, commits, and pushes to remote. |
| `/teams-issue-summary` | Generates a copy-paste-ready Microsoft Teams markdown table summarizing a range of GitHub issues — includes description, who requested it, and the result. |

## Skills

Skills are context-rich guides that teach Claude Code how to build specific types of ServiceNow artifacts using the Now SDK. Each skill includes reference documentation, code examples, and knowledge files so Claude can produce correct output without guessing.

| Skill | Description |
|-------|-------------|
| `application-menu` | Application menus and navigator modules |
| `business-rule` | Server-side scripts on record operations |
| `client-script` | Client-side form scripting |
| `creating-workspaces` | Workspace solutions with dashboards, lists, and detail forms |
| `cross-scope-privilege` | Cross-application access permissions |
| `developing-servicenow-apps` | Project setup, authentication, and build/deploy workflow |
| `email-notification` | Automated email notifications and templates |
| `implementing-security` | ACLs, roles, and data access controls |
| `implementing-tests` | Automated Test Framework (ATF) test cases |
| `importing-data` | External data import via data sources and transform maps |
| `module` | Reusable server-side JavaScript modules |
| `platform-view` | Form/list UI controls, actions, policies, and view rules |
| `property` | System properties for app configuration |
| `record` | Generic Record API for seed and metadata records |
| `script-include` | Reusable server-side classes and utilities |
| `scripted-rest-api` | Custom REST API endpoints |
| `service-catalog` | Catalog items, record producers, and variables |
| `table` | Data models with tables, columns, and relationships |
| `ui-page` | React-based UI pages and single page applications |
| `wfa-flow` | Workflow automation flows and triggers |
