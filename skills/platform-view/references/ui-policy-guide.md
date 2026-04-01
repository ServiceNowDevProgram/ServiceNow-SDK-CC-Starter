# UI Policy

## Contents

- [Instructions](#instructions) — core properties quick reference
- [Key concepts](#key-concepts) — Policy Definition, Field Actions, Related List Actions, Scripts, Conditions, UI Type Mapping
- [Common Use Cases](#common-use-cases) — 3 working examples
- [Best Practices](#best-practices)
- [Next Steps](#next-steps) — detail reference files

## Instructions

1. **Core Components:** UI Policies consist of the main policy definition, field actions, and optional related list actions.
2. **Execution Context:** UI Policies execute on the client-side when forms load or when specified conditions change.
3. **Field Actions:**
   - Control visibility using `visible` property (true = show, false = hide, 'ignore' = no change)
   - Control mandatory status using `mandatory` property (true = required, false = optional, 'ignore' = no change)
   - Control editability using `readOnly` property (true = read-only, false = editable, 'ignore' = no change)
   - Clear field values using `cleared` property when conditions are met
   - Set field values and messages using `value`, `fieldMessage`, and `fieldMessageType` properties
   - **IMPORTANT:** Create multiple field actions in the same policy to apply different behaviors to different fields. Each field action targets a specific field and can have different property combinations (e.g., one field mandatory, another field visible).
4. **Related List Actions:**
   - Control related list visibility using `visible` property
   - Use GUID format for system relationships or 'table.field' format for reference fields
   - Each related list action requires a unique `$id`
5. **Conditions:** Use encoded query strings to define when policies apply (e.g., 'urgency=1^priority=1')
6. **Scripts:** For complex logic, set `runScripts: true` and load `ui-policy-scripts.md` for details.

## Key concepts

### UI Policy Components

UI Policies control form behavior through three main components:

- **Policy Definition**: The main configuration that defines when and where the policy applies
- **Field Actions**: Specific actions to apply to individual fields when conditions are met
- **Related List Actions**: Control visibility of related lists on the form

### Policy Definition Properties

Understanding the policy definition properties helps you control when, where, and how your UI policies execute.

#### Active Status

The `active` property determines whether the policy is enabled:

- `true` (default): Policy is active and will execute
- `false`: Policy is disabled and will not execute

Use this to temporarily disable policies without deleting them during testing or troubleshooting.

#### Execution Timing

- `onLoad` (default: true): Execute when the form first loads
- Policies also re-evaluate automatically when fields referenced in conditions change

This ensures policies respond dynamically to user input without requiring page refreshes.

#### Global vs View-Specific

- `global: true` (default): Applies to all form views
- `global: false`: Specify a specific view using the `view` property

Use view-specific policies when different views require different form behaviors (e.g., "Default view" vs "IT view").

#### Reverse Behavior

- `reverseIfFalse: true` (default): Inverts all actions when conditions evaluate to false
- `reverseIfFalse: false`: No action when conditions are false

With `reverseIfFalse: true`, if you set a field mandatory when condition is true, it becomes optional when condition is false. This automatic reversal simplifies policy management.

#### Inheritance

- `inherit: false` (default): Policy only applies to the specified table
- `inherit: true`: Policy applies to tables that extend the specified table

Useful when creating policies on parent tables (like task) that should also apply to child tables (like incident, problem, change).

#### Order and Priority

- `order` (default: 100): Determines execution order when multiple policies apply
- Lower numbers execute first
- Use this to control policy precedence and ensure dependent policies execute in the correct sequence

### Field Actions

Field actions define what happens to specific fields when policy conditions are met. Each action targets a specific field and can control multiple aspects of its behavior.

**Important:** A single UI Policy can have multiple field actions, each targeting a different field with different action types. For example, you can make one field mandatory, make another field visible, and make a third field read-only—all in the same policy based on the same conditions.

#### Visibility Control

Controls whether a field is shown or hidden:

- `visible: true` - Show the field
- `visible: false` - Hide the field
- `visible: 'ignore'` (default) - No change to visibility

When a field is hidden, it remains in the form but is not displayed to the user. Hidden fields retain their values unless `cleared: true` is also set.

#### Mandatory Control

Controls whether a field is required for form submission:

- `mandatory: true` - Field is required (cannot submit without a value)
- `mandatory: false` - Field is optional
- `mandatory: 'ignore'` (default) - No change to mandatory status

Mandatory fields display with a red asterisk (\*) in ServiceNow.

#### Read-Only Control

Controls whether a field can be edited:

- `readOnly: true` - Field is read-only/disabled (cannot be edited)
- `readOnly: false` - Field is editable
- `readOnly: 'ignore'` (default) - No change to read-only status

**Important:** The `readOnly` property maps directly to the database 'disabled' field. Read-only fields appear grayed out in the ServiceNow interface.

#### Field Clearing

- `cleared: true` - Clear the field value when conditions are met
- `cleared: false` (default) - Preserve the field value

Use this to clear dependent fields when parent field changes or reset fields when a form enters a specific state.

#### Field Values and Messages

Optional properties for enhanced user experience:

- `value`: Set a field to a specific value when conditions are met
- `fieldMessage`: Display an informational message near the field
- `fieldMessageType`: Type of message ('info', 'warning', 'error')

**Example:**

```javascript
actions: [
  {
    field: "priority",
    value: "1",
    fieldMessage: "Priority has been automatically set to High",
    fieldMessageType: "info"
  }
];
```

#### Combining Multiple Properties

You can combine multiple properties in a single action:

```javascript
actions: [
  {
    field: "justification",
    visible: true,
    mandatory: true,
    fieldMessage: "Justification is required for emergency changes",
    fieldMessageType: "warning"
  }
];
```

### Related List Actions

Related list actions are an optional feature that controls the visibility of related lists on forms based on policy conditions. Use this when you need to show or hide entire related lists dynamically.

#### Key Properties

- `$id`: Unique identifier for each related list action
- `list`: Related list identifier (GUID or 'table.field' format)
- `visible`: Visibility setting (true, false, or 'ignore')

**Related List Identifier Formats:**

- **GUID format**: For system relationships (e.g., `"b9edf0ca0a0a0b010035de2d6b579a03"`)
- **Table.Field format**: For reference fields (e.g., `"incident.caller_id"`)
- **Table.Table format**: For parent-child relationships (e.g., `"change_request.change_task"`)

**Note:** The plugin automatically adds the `REL:` prefix for GUIDs when writing to ServiceNow.

For detailed examples, finding related list IDs, and best practices, load `ui-policy-related-lists.md`.

### Scripts

To enable scripts in a UI Policy, set `runScripts: true`. When enabled, these properties become available:

- `scriptTrue`: JavaScript code that executes when conditions evaluate to true
- `scriptFalse`: JavaScript code that executes when conditions evaluate to false
- `uiType`: Where scripts execute ('desktop', 'mobile-service-portal', 'all')
- `isolateScript`: Whether to run scripts in isolated scope (recommended: true)

All scripts must be wrapped in `function onCondition() { ... }` format.

For detailed script examples, g_form API usage, and best practices, load `ui-policy-scripts.md`.

### Conditions

Conditions determine when UI policies execute using ServiceNow's encoded query syntax.

#### Combining Conditions

**AND operator (`^`)** — all conditions must be true:

```javascript
conditions: "priority=1^state=2"; // Priority is 1 AND state is 2
conditions: "priority=1^urgency=1^impact=1"; // Multiple ANDs
```

**OR operator (`^OR`)** — at least one condition must be true:

```javascript
conditions: "priority=1^ORpriority=2"; // Priority is 1 OR 2
conditions: "state=6^ORstate=7^ORstate=8"; // Multiple ORs
```

#### Practical Examples

```javascript
conditions: "priority=1"; // Show when priority is critical
conditions: "state!=6^state!=7"; // Show when not closed or resolved
conditions: "category!=NULL"; // Show when category is selected
conditions: "priority<=2^ORurgency=1"; // High priority OR high urgency
conditions: "assignment_group="; // Show when assignment group is empty
```

**Important Notes:**

- Conditions are case-sensitive for field names
- Use actual field names (not labels) from the table dictionary
- Reference field conditions use the sys_id value
- Choice field conditions use the stored numeric/string value (not the display label)
- When conditions change, policies automatically re-evaluate

### UI Type Mapping

UI Policies can target specific interface types:

- `'desktop'` (default): Desktop interface only
- `'mobile-service-portal'`: Mobile and Service Portal interfaces
- `'all'`: All interfaces

The plugin automatically converts between TypeScript string literals and ServiceNow numeric values.

## Common Use Cases

Here are three essential examples to get you started. For comprehensive examples covering all scenarios, load `ui-policy-examples.md`.

### Example 1: Progressive Disclosure

Show additional fields only when relevant based on user selections:

```javascript
export const incidentCategoryPolicy = UiPolicy({
  $id: Now.ID["incident_category_policy"],
  table: "incident",
  shortDescription: "Show subcategory when category is selected",
  onLoad: true,
  conditions: "category!=NULL",
  actions: [
    {
      field: "subcategory",
      visible: true,
      mandatory: true
    }
  ]
});
```

### Example 2: Multiple Field Actions with Different Behaviors

This example demonstrates applying **different action types** to **different fields** in the same policy:

```javascript
export const highValueRequestPolicy = UiPolicy({
  $id: Now.ID["high_value_request_policy"],
  table: "sc_request",
  shortDescription: "Require approver and show urgency for high-value requests",
  onLoad: true,
  conditions: "price>10000",
  actions: [
    {
      field: "approver",
      mandatory: true,
      fieldMessage: "Approver is required for requests exceeding $10,000",
      fieldMessageType: "warning"
    },
    {
      field: "urgency",
      visible: true,
      mandatory: true
    },
    {
      field: "justification",
      visible: true,
      mandatory: true
    }
  ]
});
```

### Example 3: Choice Field-Specific Field Sets (Most Common Pattern)

Show different field sets based on dropdown selection. **CRITICAL:** Choice fields use **stored values** (not display labels) in conditions.

**Scenario:** Show different fields when expense category is "Travel" vs "Meal":

```javascript
import { UiPolicy } from "@servicenow/sdk/core";

// Policy 1: Travel Category - Show travel-specific fields
export const travelExpensePolicy = UiPolicy({
  $id: Now.ID["travel_expense_policy"],
  table: "u_expense_claim",
  shortDescription: "Show travel fields when category is Travel",
  onLoad: true,
  conditions: "u_expense_category=travel", // Use stored VALUE, not label "Travel"
  actions: [
    {
      field: "u_destination",
      visible: true,
      mandatory: true
    },
    {
      field: "u_travel_dates",
      visible: true,
      mandatory: true
    }
  ]
});

// Policy 2: Meal Category - Show meal fields, hide travel fields
export const mealExpensePolicy = UiPolicy({
  $id: Now.ID["meal_expense_policy"],
  table: "u_expense_claim",
  shortDescription: "Show restaurant name and hide travel fields for Meal",
  onLoad: true,
  conditions: "u_expense_category=meal", // Use stored VALUE, not label "Meal"
  actions: [
    {
      field: "u_restaurant_name",
      visible: true,
      mandatory: true
    },
    {
      field: "u_destination",
      visible: false // Explicitly hide competing fields
    },
    {
      field: "u_travel_dates",
      visible: false
    }
  ]
});
```

**How to Find Choice Field Stored Values:**

1. Go to your table's form in ServiceNow
2. Right-click the choice field → Configure → Show Choice List
3. Look at the **Value** column (this is what you use in conditions)
4. The **Label** column is what users see (don't use this in code)

**Common Mistakes:**

- ❌ Using label: `conditions: "category=Travel"` (shows "Travel" to users)
- ✅ Using value: `conditions: "category=travel"` (stored as "travel")
- ❌ Not hiding competing fields when switching between categories
- ✅ Explicitly set `visible: false` for fields that should hide

**Quick Reference for Choice Field Patterns:**

| Pattern              | Condition Example             |
| -------------------- | ----------------------------- |
| Specific value       | `category=hardware`           |
| Multiple values (OR) | `categoryINhardware,software` |
| Not a value          | `category!=hardware`          |
| Any value selected   | `category!=NULL`              |
| No value selected    | `category=NULL`               |

**For more examples including state-based control, field clearing, related list control, complex logic, and mobile-specific policies — load `ui-policy-examples.md`.**

## Best Practices

1. **Use Field Actions First:** Prefer field actions over scripts for simple behaviors
2. **Use 'ignore' Wisely:** Only specify properties you want to change; use 'ignore' for unchanged properties
3. **Consider Reverse Behavior:** Remember that `reverseIfFalse: true` (default) inverts actions when conditions are false
4. **Clear Dependent Fields:** Use `cleared: true` to reset dependent fields when parent conditions change
5. **Provide User Feedback:** Use field messages to explain why fields are mandatory or why values were set
6. **Test Interactions:** Test how multiple policies interact when they affect the same fields
7. **Document GUID Relationships:** When using GUIDs for related lists, add comments explaining what they represent
8. **Use Descriptive IDs:** Use meaningful names in `Now.ID[]` for better code readability

## Next Steps

Use `load_skill_resource` with skill name "ui-layout-control" and the file path to get references:

- For comprehensive examples covering all scenarios, load `references/ui-policy-examples.md`.
- For advanced script handling with g_form API, load `references/ui-policy-scripts.md`.
- For related list visibility control and finding related list IDs, load `references/ui-policy-related-lists.md`.
- For fluent API and coding examples, use `get_knowledge_source` tool to get the UI Policy knowledge source.
- For fluent Table API and coding examples, use `get_knowledge_source` tool to get the Table knowledge source.
