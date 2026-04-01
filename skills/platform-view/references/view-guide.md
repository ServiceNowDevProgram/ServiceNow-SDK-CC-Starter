# UI Views (View Definitions)

## Contents

- [Instructions](#instructions) — uniqueness check, view type selection table, implementation flow
- [Key concepts](#key-concepts) — core sys_ui_view fields, best practices, example
- [References](#references) — detail files for each access pattern
- [Next Steps](#next-steps)

## Instructions

### Critical Guidance when creating view

**1. CRITICAL: Both `name` and `title` must each be unique. Query the database before creating:**

```
table: sys_ui_view
query: name=<proposed_name>^ORtitle=<proposed_title>
```

- N > 0 → change BOTH name and title, re-query. Never fix just one.
- N = 0 → proceed to Step 2.

**Do NOT assume uniqueness.** Scope prefixes, app-specific names, or names from a different application scope do NOT guarantee uniqueness — `sys_ui_view` is a global table and `title` must be unique across ALL scopes. A scoped `name` may still collide with another app's `title`. Always query first.

**2. Choose View Type (CRITICAL - READ THIS FIRST):**

**BEFORE creating any view, you MUST follow these steps in order:**

1. Extract the access pattern keywords from the user request
2. Match against the View Type Selection Table below
3. State your classification: "This is a [GROUP/ROLE/USER/HIDDEN]-based view because..."
4. Only then proceed with implementation
5. Use `get_knowledge_source` tool to get the UI_VIEW complete API information

| If User Says (or semantic equivalent)                                                                            | View Type         | Action                                                                             |
| ---------------------------------------------------------------------------------------------------------------- | ----------------- | ---------------------------------------------------------------------------------- |
| "simple", "basic", "default", "standard", "by default", "normal layout", "basic fields", "straightforward"       | **Default**       | `import { default_view } from "@servicenow/sdk/core"` — **STOP, no Record needed** |
| "admins", "managers", "ITIL", "system administrators", "role" — generic function, NO specific name attached      | **Role-based**    | Load `view-role-based.md`                                                          |
| "team", "department", "group", "project members", "support team", "assignment group", "[any name] group"         | **Group-based**   | Load `view-group-based.md`                                                         |
| "portal", "mobile app", "API", "not in dropdown", "hidden", "should not appear", "not selectable", "system-only" | **Hidden**        | Load `view-hidden.md`                                                              |
| Any named individual — "John", "CRO Maya", "Dr. Smith", "CEO Alice" — title prefix does NOT make it role-based   | **User-specific** | Load `view-user-specific.md`                                                       |
| "everyone", "all users", "no restrictions", "public"                                                             | **Public**        | Create Record — see example below                                                  |

**When view type is Default — use `default_view`, no Record creation needed:**

**Example:**

```typescript
import { Record } from "@servicenow/sdk/core";
import { default_view } from "@servicenow/sdk/core";

// ✓ No custom view Record — use default_view directly as the view reference
export const disasterFormSection = Record({
  $id: Now.ID["disaster-report-section"],
  table: "sys_ui_section",
  data: {
    name: "u_disaster_report",
    view: default_view // standard layout, no custom view needed
  }
});
```

**Pattern Recognition:** Match by meaning, not keywords. Named users (e.g., Abraham, Sarah) or phrases like “not selectable” indicate user-specific or hidden patterns. If the text mentions team, group, or department (e.g., operations team, fulfillment team), always treat it as group-based

**Key differences:**

- **Roles** = Generic function/persona with NO specific person named (e.g., “risk analysts”, “auditors”, “department managers”) — title prefixes like CRO/CEO/CFO alone do NOT make it role-based if a name follows
- **Groups** = Any named team or department including “[Name] group” (e.g., “Governance group”, “operations team”, “fulfillment team”) — the word “group” after any term = Group-based
- **Hidden** = Not in dropdown (portal/API use) - can combine with roles/groups
- **User** = A specific named individual, even if a title precedes the name (e.g., “CRO Maya”, “CEO John”, “Dr. Smith”) — the name is the signal, not the title

**3. Access Control Data Formats:**

- **Public (everyone):** Omit roles/user/group/hidden fields
- **Role-based:** `roles: ["admin", "itil"]` (array of role strings — each role is a separate element)
- **Hidden (portal/API):** `hidden: true`
- **Group-based:** `group: <group_sys_id>` (verify group exists and is active first)
- **User-specific:** `user: <user_sys_id>` (verify user exists and is active first)

**4. Naming Convention:** Use descriptive names
**5. Verify References Exist (if using roles/groups/users):**

- **Roles:** Query `sys_user_role` table - verify roles exist before using
- **Groups:** Query `sys_user_group` table - verify group exists and is active
- **Users:** Query `sys_user` table - verify user exists and is active

### Core Instructions

1. **NEVER Create View for Default Layout:** If user wants the standard/default layout, import `default_view` from `@servicenow/sdk/core`. ONLY create custom view Records when user needs a VARIANT layout (different fields/columns/related-lists than default). Variants can be public (anyone can access) OR restricted (role/user/group-based or portal-hidden).

2. **NEVER Create ACLs for View Access:** The `roles`/`group` field on sys_ui_view handles view access control. ACLs are for table/record/field security, NOT for controlling who can select a view

### Implementation Flow

**A UI View definition alone is non-functional.** To deliver a fully functional and user-interactive experience, it must be supported by the appropriate UI components:

- **Forms** - Require: sys_ui_view + sys_ui_form + sys_ui_section + sys_ui_element
- **Lists** - Require: sys_ui_view + sys_ui_list
- **Related Lists** - Require: sys_ui_view + **FORM COMPONENTS** + RELATIONSHIP

**Note:** Always create the complete functional solution (view + components), NOT just the view definition alone.

**When implementing multiple views:** Treat each view as independent. Complete the uniqueness check → classify type → implement cycle for each view one at a time. Never proceed to implement a view before its own uniqueness query has returned results.

## Key concepts

**UI Views (sys_ui_view)** - Define custom layout variants for forms, lists, and related lists. Views control WHAT fields/columns are shown and WHO can access the layout. Users manually select views from dropdown.

### Core sys_ui_view Fields

**Identity Fields:**

- `name` (required, unique) - Technical name (e.g., "incident_mobile", "change_approval_view")
- `title` (required, unique) - Display name shown in dropdown (e.g., "Mobile View", "Approval View")

**Access Control Fields:**

- `roles` (optional) - Array of role name strings e.g. `["admin"]` or `["admin", "itil"]`. Each role is a separate element. Only users with these roles can access/select this view
- `user` (optional) - Reference to specific user. Only this user can access this view
- `group` (optional) - Reference to specific group. Only group members can access this view
- `hidden` (optional, boolean) - If true, view is hidden from platform selector (used for portals, APIs, mobile apps)

**Usage:**

- If NO access control fields set (roles/user/group empty) → **Public view** (everyone can access)
- If `roles` set → Only users with specified roles can access
- If `user` set → Only that specific user can access (personal view)
- If `group` set → Only group members can access (team view)
- If `hidden: true` → Not visible in platform dropdown (portal/API/mobile only)

### Best Practices

- **Access Control:** Use roles over user-specific views. Start least restrictive, add restrictions as needed.
- **Hidden Views:** Use for Service Portal, APIs, or integrations. Document why hidden.
- **Reusability:** Avoid duplicate views. Use consistent naming for related views.
- **Performance:** Minimize views per table. Prefer role-based over many user-specific views.
- **Uniqueness:** name and title must be unique - Always check before creating (see Instructions section)
- **Manual selection:** Users select from dropdown (for automatic switching, use view-rule-guide)
- **Complete solutions:** View alone is incomplete - Must be combined with form/list/related-list components

### Basic View Example:

```typescript
import { Record } from "@servicenow/sdk/core";

// Public view (everyone can access)
export const mobileView = Record({
  $id: Now.ID["mobile-view"],
  table: "sys_ui_view",
  data: {
    name: "incident_mobile", // descriptive unique name
    title: "Mobile View" // descriptive unique title
  }
});
```

**For complex views, load appropriate reference file (see References section).**

## References

For complex access control patterns, load appropriate reference using `load_skill_resource` with skill name "ui-layout-control":

- Role-Based Views → `references/view-role-based.md`
- Hidden Views (Portal/API) → `references/view-hidden.md`
- Group-Based Views → `references/view-group-based.md`
- User-Specific Views → `references/view-user-specific.md`
- Forms Integration → `references/view-forms-integration.md`

## Next Steps

Use `get_knowledge_source` tool to get complete implementation details:

- **UI_VIEW** - For view definitions and API schemas
- **LIST** - For list layouts (sys_ui_list)
- **RELATIONSHIP** - For related lists (sys_ui_related_list)
