---
name: creating-workspaces
description: Guide for creating Workspaces. Use when the user requests to create a workspace.
user-invocable: true
---
> [!NOTE]
> This skill was authored for an agent runtime that provides two tools not
> available here. When you encounter references to these tools, resolve them
> as follows:
>
> **`get_knowledge_source`** — When instructed to use this tool with a
> knowledge source name like `BUSINESS_RULE`, read the file at
> `references/knowledge/<name>.md` where `<name>` is the lowercase,
> hyphenated version (e.g. `BUSINESS_RULE` →
> `references/knowledge/business-rule.md`).
>
> **`load_skill_resource`** — When instructed to use this tool with a file
> path like `references/column.md`, read that file directly from the
> `references/` directory within this skill.


# Creating Workspaces

## When to use this skill

- When creating new workspaces
- When the user asks about workspace configuration or best practices

## Purpose

A Workspace provides a complete, out-of-the-box solution for managing business entities through standardized CRUD (Create, Read, Update, Delete) workflows. When you need a full business process interface with dashboard page, list management, and detailed forms, a Workspace can help with that.

## What is a Workspace?

A Workspace automatically generates a complete set of pages for managing a business entity:

- **Dashboard Page**: Overview dashboard with key metrics and recent activity
- **List Page**: Searchable, filterable table view with bulk operations
- **Detail/Form Page**: Full CRUD form with related records and actions

Workspaces follow ServiceNow's standard UX patterns and provide consistent user experience across all business entities.

## Instructions

When a user requests a workspace, follow these steps:

### Step 1: Understand the Requirement

- Identify the required tables.
  - First, check whether they already exist on the platform.
  - If not, look for them within the project.
  - Create new tables only if they cannot be found in either location.
- Gather details about the tables’ columns.

### Step 2: Instantiate and configure the UX List Menu Configuration fluent plugin

- Use provided reference to configure the UX List Menu Configuration fluent plugin inside list-menu.now.ts.

### Step 3: Instantiate and configure the Workspace fluent plugin

- Use provided reference to configure the Workspace fluent plugin in workspace.now.ts
  - Ensure you also created an ACL to secure the Workspace route
  - Ensure that you associate the UX List Menu configuration to the Workspace

### Step 4: YOU MUST Instantiate and configure the PAR Dashboard fluent plugin

- Use provided reference to configure the PAR Dashboard fluent plugin inside dashboard.now.ts
- Associate the Dashboard configuration to the Workspace by configuring visibilities on the Dashboard configuration.
- YOU MUST ensure that the dashboard is created, even though the Dashboard plugin references the Workspace plugin. This is mandatory or else the Workspace will not function correctly.

### Step 5: Verify Integration

- Ensure the UxListMenuConfig is properly referenced in Workspace
- Verify that the Workspace is referenced in Dashboard visibilities
- Confirm ACL field matches workspace path pattern: `{path}.*`
- Check that all roles are properly defined and referenced

### Step 6: Run Diagnostics

- Run diagnostics to ensure the workspace is properly configured

### Step 7: Build and Install the Workspace

- Build and install the workspace.

### Step 8: Get Workspace Sys_ID

After building and installing:

- Read `src/fluent/generated/keys.ts` file
- Extract the actual `id` value from the `sys_ux_page_registry` key entry for the workspace
- Use this sys_id for the UI Builder URL in the final summary

### Step 9: Provide a Summary to the User

- Summarize the workspace configuration and provide the user with:
  - clickable link with the URL to access the workspace
  - clickable link with the URL to edit the workspace in UI Builder with the ACTUAL sys_id
    - **MANDATORY**: Replace {workspace_sys_id} with the real sys_id from the `sys_ux_page_registry` id you found in keys.ts
    - **NEVER** provide placeholder URLs with {workspace_sys_id} - always provide the complete, working URL
- Both of these links can be relative links since they are already on the correct ServiceNow instance.

## File Organization

- Example of a project with two workspaces, one for incident tracking and one for asset management
- When creating a new workspace, you should always end up with these three now.ts files.

```
src/
  fluent/
    workspaces/
      incident-tracker/
        workspace.now.ts
        list-menu.now.ts
        dashboard.now.ts
      asset-manager/
        workspace.now.ts
        list-menu.now.ts
        dashboard.now.ts
```

## Workspace URL Structure

When a Workspace is created, it generates a URL following this pattern:

```
/now/{path}/{landingPath}
```

**Example 1**: If `path: 'my-example'` the workspace URL will be:

```
/now/my-example/home
```

**Example 2**: If `path: 'my-other-example'` and `landingPath: 'main'`, the workspace URL will be:

```
/now/my-other-example/main
```

## UI Builder URL for Workspace

You can edit the workspace in UI Builder using the following URL:

```
/now/builder/ui/experience/{workspace_sys_id}
```

## References

- For Fluent API details for Workspaces, use `get_knowledge_source` to fetch the `UX_WORKSPACE` knowledge source
- For Fluent API details for UX List Menu Configurations, use `get_knowledge_source` to fetch the `UX_LIST_MENU_CONFIG` knowledge source
- For Fluent API details for Dashboards, use `get_knowledge_source` to fetch the `PAR_DASHBOARD` knowledge source
