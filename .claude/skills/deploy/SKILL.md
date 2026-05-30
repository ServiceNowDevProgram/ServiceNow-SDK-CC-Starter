---
name: deploy
description: Build and deploy the ServiceNow app to an instance. Supports dev/uat/prod environments with a confirmation gate for prod and an annotated git tag for each UAT/prod deploy.
user-invocable: true
---

Build and deploy the ServiceNow app to an instance.

Arguments: $ARGUMENTS (optional — target environment: `dev`, `uat`, or `prod`. Defaults to `dev` if omitted.)

## Step 1: Resolve target

Parse $ARGUMENTS to determine the deployment target. Map each target to the credential alias and instance URL configured for this project (see `.snc/config.json` or equivalent — update the mapping below once per project):

- If empty or `dev` → alias = `dev`, instance = `<dev-instance>`
- If `uat` → alias = `uat`, instance = `<uat-instance>`
- If `prod` → alias = `prod`, instance = `<prod-instance>`
- If anything else → stop and report: "Unknown target '$ARGUMENTS'. Use: dev, uat, or prod."

Report: "Deploying to **<environment>** (<instance URL>)"

## Step 2: Build

First, check the recent conversation history. If the most recent significant activity was a successful `npm run build` (or equivalent) AND no source files have been edited since that build, skip this step and report: "Recent build detected — skipping rebuild." Then proceed to Step 3.

Otherwise, run:
```
npm run build
```

If the build fails, stop and report the error. Do not proceed to deploy.

## Step 3: Confirm for production

If the target is `prod`, warn the user:

> **Production deployment.** This will update the live instance at <prod-instance>. Proceed?

Wait for confirmation before continuing. If the user declines, stop.

If the target is `dev` or `uat`, proceed without confirmation.

## Step 4: Deploy

Run:
```
now-sdk install --auth <alias>
```

Where `<alias>` is the resolved credential alias from Step 1.

Report the result. If successful, report:

> Successfully deployed to **<environment>** (<instance URL>)

## Step 5: Tag (UAT and Prod only)

If the target is `uat` or `prod`, create an annotated git tag for audit trail:

```
git tag -a <env>/<YYYY-MM-DD>-<short-hash> -m "Deploy to <env>"
```

For example: `prod/2026-04-15-b0b8089`

If a tag with that exact name already exists (multiple deploys same day), append a sequence number: `prod/2026-04-15-b0b8089-2`

Report the tag name. Do NOT push the tag automatically — that happens when the user runs `/push` (which uses `--follow-tags`).
