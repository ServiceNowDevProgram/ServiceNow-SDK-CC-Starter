# UI Policy Examples

This reference provides comprehensive, practical examples of common UI Policy scenarios. Each example demonstrates key concepts and best practices for implementing UI Policies in ServiceNow.

## Table of Contents

1. [State-Based Field Control](#state-based-field-control) - Read-only fields for specific states
2. [Field Clearing on Condition Change](#field-clearing-on-condition-change) - Clear dependent fields when conditions change
3. [Combined Field and Related List Control](#combined-field-and-related-list-control) - Control both fields and related lists
4. [Default Values with Validation](#default-values-with-validation) - Set values and display field messages
5. [Complex Conditional Logic](#complex-conditional-logic) - Multiple conditions with AND/OR operators

## State-Based Field Control

**Use Case:** Make fields read-only when records reach certain states to prevent accidental changes to closed or resolved records.

**Scenario:** Lock key fields when an incident is closed to maintain data integrity.

```javascript
import { UiPolicy } from "@servicenow/sdk/core";

export const closedIncidentPolicy = UiPolicy({
  $id: Now.ID["closed_incident_policy"],
  table: "incident",
  shortDescription: "Make fields read-only for closed incidents",
  onLoad: true,
  conditions: "state=7", // Closed state
  actions: [
    {
      field: "short_description",
      readOnly: true
    },
    {
      field: "description",
      readOnly: true
    },
    {
      field: "priority",
      readOnly: true
    },
    {
      field: "category",
      readOnly: true
    },
    {
      field: "assignment_group",
      readOnly: true
    }
  ]
});
```

**Key Points:**

- Uses `state=7` to target closed incidents (adjust based on your state values)
- `readOnly: true` prevents field editing while keeping values visible
- Multiple fields can be locked with a single policy
- Users can still view data but cannot modify it
- Consider which fields should remain editable even when closed (e.g., work notes for documentation)

---

## Field Clearing on Condition Change

**Use Case:** Automatically clear dependent fields when parent conditions change to prevent invalid data combinations.

**Scenario:** Clear resolution fields when an incident is reopened to ensure resolution data doesn't persist incorrectly.

```javascript
import { UiPolicy } from "@servicenow/sdk/core";

export const incidentReopenPolicy = UiPolicy({
  $id: Now.ID["incident_reopen_policy"],
  table: "incident",
  shortDescription: "Clear resolution fields when incident is reopened",
  onLoad: true,
  conditions: "state!=6^state!=7", // Not Resolved and Not Closed
  actions: [
    {
      field: "resolution_code",
      cleared: true
    },
    {
      field: "close_notes",
      cleared: true
    },
    {
      field: "resolved_at",
      cleared: true
    },
    {
      field: "resolved_by",
      cleared: true
    }
  ]
});
```

**Key Points:**

- Uses `state!=6^state!=7` to target non-resolved/non-closed states
- `cleared: true` removes existing field values
- Prevents stale resolution data from appearing on reopened incidents
- Combines multiple conditions with AND operator (`^`)
- Automatically executes when state changes, not just on load

---

## Combined Field and Related List Control

**Use Case:** Control both field behavior and related list visibility in a single policy for comprehensive form management.

**Scenario:** For high-priority incidents, make work notes mandatory and show the incident tasks related list.

```javascript
import { UiPolicy } from "@servicenow/sdk/core";

export const highPriorityPolicy = UiPolicy({
  $id: Now.ID["high_priority_policy"],
  table: "incident",
  shortDescription: "Show additional details for high priority incidents",
  onLoad: true,
  conditions: "priority=1^ORpriority=2", // Priority 1 (Critical) OR 2 (High)
  actions: [
    {
      field: "work_notes",
      visible: true,
      mandatory: true
    },
    {
      field: "impact",
      mandatory: true
    }
  ],
  relatedListActions: [
    {
      $id: Now.ID["show_incident_tasks"],
      list: "incident_task.parent",
      visible: true
    },
    {
      $id: Now.ID["show_affected_cis"],
      list: "task_ci.task",
      visible: true
    }
  ]
});
```

**Key Points:**

- Combines field actions and related list actions in one policy
- Uses OR operator (`^OR`) to match multiple priority levels
- Each related list action requires a unique `$id`
- `list` property uses `table.field` format for reference relationships
- Demonstrates managing multiple aspects of form behavior together

---

## Default Values with Validation

**Use Case:** Automatically set field values and provide validation messages when conditions are met.

**Scenario:** For security-related incidents, set priority to High and display guidance messages.

```javascript
import { UiPolicy } from "@servicenow/sdk/core";

export const securityIncidentPolicy = UiPolicy({
  $id: Now.ID["security_incident_policy"],
  table: "incident",
  shortDescription: "Set defaults and show messages for security incidents",
  onLoad: true,
  conditions: "categoryLIKEsecurity",
  actions: [
    {
      field: "priority",
      value: "2", // Set to High
      readOnly: true,
      fieldMessage: "Priority automatically set to High for security incidents",
      fieldMessageType: "info"
    },
    {
      field: "assignment_group",
      mandatory: true,
      fieldMessage: "Security incidents must be assigned immediately",
      fieldMessageType: "warning"
    },
    {
      field: "impact",
      value: "2", // Set to Medium
      mandatory: true
    }
  ]
});
```

**Key Points:**

- `value` property sets field values automatically
- `readOnly: true` locks the auto-set value to prevent changes
- `fieldMessage` provides user guidance
- `fieldMessageType` controls message styling ('info', 'warning', 'error')
- Combines value setting, locking, and messaging in one action

---

## Complex Conditional Logic

**Use Case:** Implement policies with sophisticated condition logic using multiple operators.

**Scenario:** Show escalation fields for incidents that are open, high priority, and assigned to specific groups.

```javascript
import { UiPolicy } from "@servicenow/sdk/core";

export const escalationPolicy = UiPolicy({
  $id: Now.ID["escalation_policy"],
  table: "incident",
  shortDescription: "Show escalation fields for open high-priority incidents",
  onLoad: true,
  conditions: "stateIN1,2,3^priority<=2^assignment_group!=NULL",
  actions: [
    {
      field: "escalation",
      visible: true,
      mandatory: false
    },
    {
      field: "escalation_reason",
      visible: true
    },
    {
      field: "escalated_to",
      visible: true
    }
  ]
});
```

**Key Points:**

- `stateIN1,2,3`: Uses IN operator to match multiple open states
- `priority<=2`: Uses comparison operator for priority levels
- `assignment_group!=NULL`: Ensures incident is assigned
- All conditions connected with AND operator (`^`)
- Demonstrates combining different condition operators in one policy

---

## Best Practices Summary

Based on these examples, follow these best practices:

1. **Keep Policies Focused:** Create separate policies for distinct behaviors rather than complex multi-purpose policies
2. **Use Descriptive IDs:** Use meaningful names in `Now.ID[]` that indicate the policy's purpose
3. **Document Conditions:** Add comments explaining complex condition logic
4. **Consider Reverse Behavior:** Remember `reverseIfFalse: true` (default) inverts actions automatically
5. **Provide User Guidance:** Use field messages to explain why fields are required or values are set
6. **Test Interactions:** Test how multiple policies interact when they affect the same fields
7. **Optimize for UX:** Progressive disclosure improves usability by showing fields only when needed
8. **Clear Dependent Data:** Use `cleared: true` to prevent invalid data combinations
9. **Lock Historical Data:** Use `readOnly: true` to protect data integrity in closed records
10. **Plan for Mobile:** Consider mobile users and create appropriate mobile-specific policies when needed
