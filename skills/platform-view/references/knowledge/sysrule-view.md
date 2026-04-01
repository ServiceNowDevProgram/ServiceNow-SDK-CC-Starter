# SYSRULE_VIEW

# SYSRULE_VIEW: Fluent Record API for sysrule_view

**Table:** `sysrule_view`

## Record API Properties

| Property                         | Type    | Required | Description                                                                                                                                                         |
| -------------------------------- | ------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$id`                            | String  | Yes      | Unique ID for the metadata object in the format `$id: Now.ID[<value>]`, where `<value>` is a string or number. This ID is hashed into a unique sys_ID during build. |
| `table`                          | String  | Yes      | Must be `"sysrule_view"`.                                                                                                                                           |
| `data`                           | Object  | Yes      | Contains the view rule configuration with the following properties:                                                                                                 |
| `data.name`                      | String  | Yes      | Descriptive name for the rule.                                                                                                                                      |
| `data.table`                     | String  | Yes      | Target table name where rule applies (e.g., `"incident"`).                                                                                                          |
| `data.view`                      | String  | No       | View name from sys_ui_view (use `name`, not `title`). Required unless using advanced script.                                                                        |
| `data.condition`                 | String  | No       | Encoded query string ending with `^EQ`. Example: `"priority=1^state=2^EQ"`                                                                                          |
| `data.match_conditions`          | String  | No       | How to evaluate multiple conditions: `"ALL"` or `"ANY"`. Default: `"ALL"`                                                                                           |
| `data.device_type`               | String  | No       | Device type filter: `"browser"`, `"mobile"`, or `"tablet"`.                                                                                                         |
| `data.active`                    | Boolean | No       | Whether rule is active. Default: `true`                                                                                                                             |
| `data.overrides_user_preference` | Boolean | No       | Override user's manual view selection. Default: `true`                                                                                                              |
| `data.advanced`                  | Boolean | No       | Enable custom script. Default: `false`                                                                                                                              |
| `data.script`                    | String  | No       | JavaScript for complex logic (when `advanced: true`). Must set `answer` variable.                                                                                   |
| `data.order`                     | Number  | No       | Evaluation order (lower numbers evaluated first). Default: `100`                                                                                                    |

## Encoded Query Operators

Common operators for `data.condition` property:

```
=          Equal to
!=         Not equal
^          AND
^OR        OR
>, <, >=, <=    Comparison
LIKE       Contains
STARTSWITH Starts with
ENDSWITH   Ends with
ISEMPTY    Is empty
ISNOTEMPTY Is not empty
```

**Note:** All conditions must end with `^EQ` operator.

## Advanced Script Structure

When `data.advanced: true`, use this script structure:

```typescript
script: `(function overrideView(view, is_list) {
  // Parameters:
  //   view (string) - Current view name
  //   is_list (boolean) - true for list, false for form
  //   current - Current record (forms only)
  //   answer - Set to view name or null

  // Your logic here
  answer = 'view_name';
})(view, is_list);`;
```

**Requirements:**

- Wrapped in IIFE: `(function overrideView(view, is_list) { ... })(view, is_list);`
- Set `answer` variable to view name (string) or `null`
- Use view's `name` field from sys_ui_view, not `title`
- `current` object only available for forms, not lists

## Examples

### Example 1: Device-Based View Switching

```typescript
import { Record } from "@servicenow/sdk/core";

// Automatically switch to mobile view on mobile devices
export const mobileViewRule = Record({
  $id: Now.ID["incident-mobile-view-rule"],
  table: "sysrule_view",
  data: {
    name: "Incident Mobile View Rule",
    table: "incident",
    view: "mobile", // Must exist in sys_ui_view
    device_type: "mobile",
    active: true,
    overrides_user_preference: true
  }
});
```

### Example 2: Condition-Based View Switching

```typescript
// Automatically show critical view for high-priority incidents
export const criticalViewRule = Record({
  $id: Now.ID["incident-critical-view-rule"],
  table: "sysrule_view",
  data: {
    name: "Critical Priority View Rule",
    table: "incident",
    view: "critical", // Must exist in sys_ui_view
    condition: "priority=1^ORpriority=2^EQ", // Priority 1 or 2
    active: true,
    overrides_user_preference: true,
    order: 100
  }
});
```

### Example 3: Role-Based View Switching (Advanced Script)

```typescript
// Show different views based on user role
export const roleBasedViewRule = Record({
  $id: Now.ID["incident-role-view-rule"],
  table: "sysrule_view",
  data: {
    name: "Role-Based View Rule",
    table: "incident",
    view: null, // Set dynamically by script
    advanced: true,
    active: true,
    overrides_user_preference: true,
    script: `(function overrideView(view, is_list) {
      var user = gs.getUser();

      if (user.hasRole('admin')) {
        answer = 'admin_view';
      } else if (user.hasRole('itil')) {
        answer = 'agent_view';
      } else {
        answer = 'ess';
      }
    })(view, is_list);`
  }
});
```
