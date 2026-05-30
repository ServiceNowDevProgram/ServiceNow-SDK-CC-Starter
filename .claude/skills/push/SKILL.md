---
name: push
description: Stage, commit, and push all changes to remote. Use when ready to push to the remote branch.
user-invocable: true
---

Commit all changes and push to remote.

Run the following commands in sequence:
1. `git add -A && git commit` - Stage and commit all changes
2. `git push --follow-tags` - Push to remote (includes any deployment tags)

Report the result of each step.
