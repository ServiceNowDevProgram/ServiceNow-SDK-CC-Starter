# Advanced View Rules (Script-Based)

Complex view rules using custom scripts (`advanced: true`) for role-based logic, multi-criteria conditions, and dynamic switching.

## Contents

- [Script Variables](#script-variables) — available variables in script context
- [Role-Based Switching](#role-based-switching) — switching views based on user roles
- [Complex Conditional Logic](#complex-conditional-logic-forms) — multi-criteria, forms-only logic
- [GlideElement Methods](#glideelement-methods) — accessing record field values
- [Performance Best Practices](#performance-best-practices)
- [Debugging with Logging](#debugging-with-logging)
- [Common Mistakes](#common-mistakes)

## Script Variables

When using `advanced: true`, these variables are available:

| Variable  | Type        | Availability | Description                          |
| --------- | ----------- | ------------ | ------------------------------------ |
| `view`    | string      | Always       | Current view name                    |
| `is_list` | boolean     | Always       | true for lists, false for forms      |
| `current` | GlideRecord | Forms only   | Current record (undefined for lists) |
| `answer`  | string/null | Always       | Set this to the view name            |
| `gs`      | GlideSystem | Always       | GlideSystem API                      |

**CRITICAL:** Always check `!is_list && typeof current !== 'undefined'` before accessing `current`.

## Role-Based Switching

```typescript
import { Record } from "@servicenow/sdk/core";

export const roleBasedRule = Record({
  $id: Now.ID["role-rule"],
  table: "sysrule_view",
  data: {
    name: "Role-Based View Rule",
    table: "incident",
    view: null, // Set by script
    advanced: true,
    active: true,
    overrides_user_preference: true,
    script: `(function overrideView(view, is_list) {
      var user = gs.getUser();

      // Role hierarchy (check highest priority first)
      if (user.hasRole('admin')) {
        answer = 'admin_view';
      } else if (user.hasRole('manager')) {
        answer = 'manager_view';
      } else if (user.hasRole('agent')) {
        answer = 'agent_view';
      } else {
        answer = 'ess';
      }
    })(view, is_list);`
  }
});
```

## Complex Conditional Logic (Forms)

```typescript
export const complexRule = Record({
  $id: Now.ID["complex-rule"],
  table: "sysrule_view",
  data: {
    name: "Complex Conditional Rule",
    table: "incident",
    view: null,
    advanced: true,
    active: true,
    overrides_user_preference: true,
    script: `(function overrideView(view, is_list) {
      // CRITICAL: Check if form and current exists
      if (!is_list && typeof current !== 'undefined') {
        var priority = current.priority.toString();
        var state = current.state.toString();

        // Multi-field logic
        if (priority === '1' && state === '2') {
          answer = 'critical_active_view';
        } else if (priority === '1' && state === '7') {
          answer = 'critical_closed_view';
        } else {
          answer = null;  // Use default
        }
      }
    })(view, is_list);`
  }
});
```

## GlideElement Methods

When accessing `current` record fields:

```javascript
// CORRECT - Use .toString() for comparisons
if (current.priority.toString() === "1") {
  answer = "critical_view";
}

// Reference fields
current.assignment_group.toString(); // Returns sys_id
current.assignment_group.getDisplayValue(); // Returns display name
current.assignment_group.nil(); // Check if empty
```

## Performance Best Practices

1. **Minimize Database Queries** - Use `current` record, not new GlideRecord lookups
2. **Cache User Info** - Use `gs.getUser()` (cached), not repeated queries
3. **Early Returns** - Exit when condition met
4. **Check Context First** - Always verify `!is_list && current` before using current
5. **Use Session Properties** - Prefer session over repeated property lookups

## Debugging with Logging

```typescript
script: `(function overrideView(view, is_list) {
  try {
    gs.info('View: ' + view + ', Is list: ' + is_list);

    var user = gs.getUser();
    gs.info('User: ' + gs.getUserName() + ', Has admin: ' + user.hasRole('admin'));

    if (user.hasRole('admin')) {
      answer = 'admin_view';
    } else {
      answer = null;
    }
  } catch (e) {
    gs.error('View Rule Error: ' + e.message);
    answer = null;
  }
})(view, is_list);`;
```

## Common Mistakes

- **current unchecked**: May be undefined → check `!is_list && typeof current !== 'undefined'`
- **answer not set**: Rule won’t work → set `answer = 'view_name'` or `null`
- **Missing .toString()**: Comparison fails → use `current.priority.toString() === "1"`
- **Syntax errors**: Script fails → fix quotes, semicolons, syntax
- **Using view title**: View not found → use view name field
- **Heavy GlideRecord queries**: Slows performance → use current or cached data
