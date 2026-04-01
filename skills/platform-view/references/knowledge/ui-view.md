# UI_VIEW

# UI_VIEW: Fluent Record API for sys_ui_view

**Table:** `sys_ui_view`

## Record API Properties

| Property      | Type             | Required | Description                                                                                                                                                         |
| ------------- | ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$id`         | String or Number | Yes      | Unique ID for the metadata object in the format `$id: Now.ID[<value>]`, where `<value>` is a string or number. This ID is hashed into a unique sys_ID during build. |
| `table`       | String           | Yes      | Must be `"sys_ui_view"`.                                                                                                                                            |
| `data`        | Object           | Yes      | Contains the view configuration with the following properties:                                                                                                      |
| `data.name`   | String           | Yes      | Unique name for the view. Used to reference the view in forms, lists, and other configurations. Maximum length: 80                                                  |
| `data.title`  | String           | Yes      | Unique title for the view. Maximum length: 80                                                                                                                       |
| `data.roles`  | String[]         | No       | Array of role name strings. Each role is a separate element. Example: `["admin", "itil"]`                                                                           |
| `data.user`   | String           | No       | sys_id or `$id` reference of a specific sys_user record. Makes this view available only to that user.                                                               |
| `data.group`  | String           | No       | sys_id or `$id` reference of a sys_user_group record. Makes this view available to users in that group.                                                             |
| `data.hidden` | Boolean          | No       | If `true`, the view is hidden from view selectors but can still be used programmatically. Default: `false`                                                          |

## Critical Implementation Notes

- **CRITICAL: Both `name` and `title` must each be unique** — query `sys_ui_view` with `name=<proposed_name>^ORtitle=<proposed_title>` before creating. If N > 0, change BOTH and re-query. Never fix just one. Do NOT assume scope-prefixed names are unique — `sys_ui_view` is a global table and `title` must be unique across all scopes.

## Basic Examples

### Public View

```typescript
import { Record } from "@servicenow/sdk/core";

export const defaultView = Record({
  $id: Now.ID["default-view"],
  table: "sys_ui_view",
  data: {
    name: "standard_view", // unique name
    title: "Standard View" // unique title
  }
});
```

### Role-Based View

```typescript
export const adminView = Record({
  $id: Now.ID["admin-view"],
  table: "sys_ui_view",
  data: {
    name: "admin_dashboard", // unique name
    title: "Admin Dashboard", // unique title
    roles: ["admin", "itil"]
  }
});
```

### Hidden View (Portal/API)

```typescript
export const portalView = Record({
  $id: Now.ID["portal-view"],
  table: "sys_ui_view",
  data: {
    name: "service_portal", // unique name
    title: "Service Portal View", // unique title
    hidden: true
  }
});
```

## Access Control Summary

| Access Control             | Who Can See/Select This View                               |
| -------------------------- | ---------------------------------------------------------- |
| No roles/user/group        | Everyone (public view)                                     |
| `roles: ["admin", "itil"]` | Users with admin OR itil role                              |
| `user: userReference`      | Only that specific user                                    |
| `group: groupReference`    | Only members of that group                                 |
| `hidden: true`             | Hidden from platform selector (portal/API/mobile use only) |
