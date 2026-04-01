# List Control

## Contents

- [Critical Instructions](#critical-instructions)
- [Quick Guide](#quick-guide)
- [Best Practices](#best-practices)
- [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
- [Next Steps](#next-steps)

## Critical Instructions

When developing List Controls with Record API, follow these guidelines to ensure proper functionality. Failure to follow these requirements may result in build or runtime errors.

1. Each list control must have a unique `$id` using `Now.ID['value']` format
2. Set `table: 'sys_ui_list_control'`
3. Provide valid `name` (table name this list control applies to)
4. For related lists, use `related_list` in format `table.field` or `REL:sys_id`
5. Use condition scripts with `Now.include()` for external script files
6. Role fields accept array of role names in comma-separated string format : ["admin", "itil"]
7. Use `active: true` to enable list control

## Quick Guide

Here are some basic examples to get you started quickly:

### Example 1: Student information for a particular department record should be read only.

**NOTE**: When a simple reference field exists (e.g., `table.field`), the relationship `sys_id` is not required. For example, "Student" can be displayed on the Department form to show all associated students by leveraging the implicit relationship between the `department` table and the `student` table. The list is configured to appear in the default view of the `department` form's related list.

```typescript
import { Record } from "@servicenow/sdk/core";

export const departmentStudentListControl = Record({
  $id: Now.ID["department_student_list_control"],
  table: "sys_ui_list_control",
  data: {
    name: "department",
    related_list: "student.department",
    omit_new_button: true,
    omit_edit_button: true,
    list_edit_type: "disabled"
  }
});
```

### Example 2: Option to add new incident should be visible to admin and itil roles and edit option should be visible to admin role only.

**NOTE**: Verify roles exists in the system. If roles do not exist, create them first. Use `role` skill to create roles.

```typescript
export const roleBasedAccess = Record({
  $id: Now.ID["role-based-access"],
  table: "sys_ui_list_control",
  data: {
    name: "incident",
    new_roles: ["admin", "itil"], // Only admin and itil can create
    edit_roles: ["admin"] // Only admin can edit
  }
});
```

### Example 3: When a incident is resolved or closed, option to add new child incident and edit child incident should not be disabled.

```typescript
import { Record } from "@servicenow/sdk/core";

export const incidentChildConditionalControl = Record({
  $id: Now.ID["incident-conditional-button"],
  table: "sys_ui_list_control",
  data: {
    name: "incident",
    related_list: "incident.parent_incident",
    new_condition: Now.include("../scripts/incidentStateIsResolvedOrClosed.js"),
    edit_condition: Now.include("../scripts/incidentStateIsResolvedOrClosed.js")
  }
});
```

Condition script (`incidentStateIsResolvedOrClosed.js`):

```javascript
var answer;
if (parent.state == 6 || parent.state == 7) {
  answer = true; // hide when Resolved (6) or Closed (7)
} else {
  answer = false; // show otherwise
}
answer;
```

**How it works**:

- List Control condition scripts are evaluated on the server, and the resulting answer determines the UI behavior (e.g., whether New/Edit is shown) in the related list.
- `parent` object provides access to parent record's fields
- `answer = true` hides the button; `answer = false` shows it

## Best Practices

1. **Don't combine `omit_*_button: true` with role permissions `*_roles` for the same capability**: The omit flag will override role permissions and hide the button for everyone.
2. **Button Visibility Logic**: Hide button if: `omit_*_button == true` **OR** `*_condition` evaluates to true, this is OR logic, not AND. If either property is true, the button is hidden.
3. **Performance**: Use `omit_count: true` for large tables (>10,000 records).
4. **Conditional list controls with scripts**: Apply list controls with specific conditions, use `*_condition` with script files via `Now.include()` rather than broad rules.
5. **Role-based list controls**: Always use role fields for role-based list behavior control.
6. **Naming**: Use descriptive `$id` values and valid table names in `name` field.

## Anti-Patterns to Avoid

**Don't do this:**

```typescript
// Bad - omit flag overrides role permissions
export const badControl = Record({
  $id: Now.ID["bad-control"],
  table: "sys_ui_list_control",
  data: {
    name: "incident",
    omit_new_button: true, // Hides for everyone
    new_roles: ["admin"] // This is ignored!
  }
});
```

**Avoid conflicting controls for same table**

## Next Steps

### List Control API Reference

- For complete List Control API properties and syntax, use `get_knowledge_source` tool with knowledge source name "list_control" to get the list_control knowledge source.

### Detailed Guides

- For comprehensive implementation examples, use `load_skill_resource` with skill name "ui-layout-control" and file path `references/list-control-examples.md`

### Related Skills

- For creating tables, activate the `table` skill to create tables
- For role-based access control, activate the `role` skill to create roles
