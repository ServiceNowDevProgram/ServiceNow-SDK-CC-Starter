# WFA_FLOW

The Flow API reference provides complete technical specifications for the Flow constructor, configuration properties, and execution callback. This document contains API signatures, parameter definitions, types, and syntax examples. For conceptual guidance, design patterns, best practices, error handling, performance optimization, debugging strategies, and when-to-use information, refer to skill references.

## Flow Constructor

The Flow constructor creates a complete automation unit with configuration, trigger, and execution logic.

### Signature

```typescript
Flow(
  config: FlowConfig,
  trigger: Trigger,
  callback: (params: FlowParams) => void
)
```

### Parameters

**1. config** - Flow configuration object defining metadata and execution context

**2. trigger** - Flow activation point

**3. callback** - Flow logic function containing actions and flow logic

### Import Guidelines

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";
```

**Important:** Do NOT import `Time`, `Duration`, or `TemplateValue` from `@servicenow/sdk/core` - they are available globally from `@servicenow/sdk/global`.

---

## Flow Configuration Properties

The configuration object defines the flow's metadata and execution context.

| Property     | Type                        | Required | Default  | Description                                          |
| ------------ | --------------------------- | -------- | -------- | ---------------------------------------------------- |
| $id          | string                      | Yes      | -        | Unique identifier for the flow (Now.ID['flow_name']) |
| name         | string                      | Yes      | -        | Display name shown in UI                             |
| description  | string                      | No       | -        | Flow purpose and behavior (recommended)              |
| runAs        | 'system' \| 'user'          | No       | 'system' | Execution security context                           |
| runWithRoles | string[]                    | No       | []       | Required roles to execute flow                       |
| flowPriority | 'LOW' \| 'MEDIUM' \| 'HIGH' | No       | 'MEDIUM' | Execution priority for resource allocation           |
| protection   | 'read' \| ''                | No       | ''       | Flow protection level (rarely used)                  |

### Example - Basic Flow Configuration

```typescript
{
  $id: Now.ID['auto_assign_incidents'],
  name: 'Auto-Assign High Priority Incidents',
  description: 'Automatically assigns critical incidents to on-call engineer',
  runAs: 'system',
  flowPriority: 'HIGH'
}
```

---

## Configuration Property Details

### $id (Required)

**Type:** `string`
**Usage:** `Now.ID['unique_flow_identifier']`

Unique identifier for the flow. Must be unique across all flows in the application.

**Example:**

```typescript
{
  $id: Now.ID['auto_assign_incidents'],
  name: 'Auto-Assign High Priority Incidents'
}
```

---

### name (Required)

**Type:** `string`

Display name shown in the ServiceNow UI. Should be descriptive and indicate the flow's purpose.

**Example:**

```typescript
{
  $id: Now.ID['flow'],
  name: 'Escalate High Priority Incidents'
}
```

---

### description (Optional)

**Type:** `string`
**Default:** None

Describes the flow's purpose and behavior. Recommended for documentation and maintainability.

**Example:**

```typescript
{
  $id: Now.ID['flow'],
  name: 'My Flow',
  description: 'Automatically assigns critical incidents to on-call engineer when priority is 1'
}
```

---

### runAs (Optional)

**Type:** `'system' | 'user'`
**Default:** `'system'`

Defines the execution security context for the flow.

**Values:**

- `'system'` - Run with system privileges, bypassing ACLs (default behavior)
- `'user'` - Run with the privileges of the user who triggered the flow

**Guidelines:**

- Use `'system'` (default) when automation must complete regardless of user permissions
- Use `'user'` when flow actions should respect user permissions and ACLs must be enforced

**Example:**

```typescript
// System context - bypasses ACLs (DEFAULT)
{
  $id: Now.ID['system_cleanup'],
  name: 'System Cleanup Flow',
  runAs: 'system'
}

// User context - respects ACLs
{
  $id: Now.ID['user_request'],
  name: 'User Request Flow',
  runAs: 'user'
}
```

---

### runWithRoles (Optional)

**Type:** `string[]`
**Default:** `[]`

Specifies required roles that must be present for the flow to execute. Users triggering the flow must have all specified roles.

**Example:**

```typescript
{
  $id: Now.ID['admin_flow'],
  name: 'Admin Only Flow',
  runWithRoles: ['admin', 'itil']  // User must have both roles
}
```

**Note:** To get role sys_ids, use the `run_query` tool:

```typescript
run_query("sys_user_role", "name=admin");
```

---

### flowPriority (Optional)

**Type:** `'LOW' | 'MEDIUM' | 'HIGH'`
**Default:** `'MEDIUM'`

Defines execution priority for resource allocation and scheduling.

**Guidelines:**

- `'HIGH'` - Time-sensitive, user-facing operations that need immediate processing
- `'MEDIUM'` - Standard automation workflows (default)
- `'LOW'` - Batch processing, cleanup tasks, non-urgent background operations

**Example:**

```typescript
{
  $id: Now.ID['critical_flow'],
  name: 'Critical Incident Handler',
  flowPriority: 'HIGH'  // Process immediately
}
```

---

### protection (Optional)

**Type:** `'read' | ''`
**Default:** `''` (no protection)

Defines flow protection level.

**Values:**

- `'read'` - Flow is read-only and cannot be modified in the UI
- `''` - No protection (default)

**Example:**

```typescript
{
  $id: Now.ID['protected_flow'],
  name: 'Protected System Flow',
  protection: 'read'
}
```

---

## Trigger Parameter

The second parameter defines when the flow should execute.

### Structure

```typescript
wfa.trigger(
  triggerType,              // trigger.record.created, trigger.scheduled.daily, etc.
  config: { $id: string },
  parameters: {...}
)
```

---

## Flow Logic Callback

The third parameter is a callback function containing the flow's automation logic.

### Signature

```typescript
(_params: FlowParams) => void
```

### FlowParams Structure

```typescript
{
  trigger: {
    current: Record,              // Current record (for record triggers)
    previous?: Record,            // Previous state (for update triggers)
    run_start_date_time: DateTime,
    // ... trigger-specific outputs
  }
}
```

### Callback Pattern

```typescript
_params => {
  // Execute actions
  const result = wfa.action(
    action.core.lookUpRecord,
    { $id: Now.ID["lookup"] },
    { table: "incident", conditions: "..." }
  );

  // Conditional logic
  wfa.flowLogic.if({ $id: Now.ID["condition"], condition: "..." }, () => {
    // Actions when condition is true
  });

  // Loops
  wfa.flowLogic.forEach(
    wfa.dataPill(result.Records, "array"),
    { $id: Now.ID["loop"] },
    item => {
      // Process each item
    }
  );
};
```

**Note:** Access dynamic data using `wfa.dataPill(source, type)`.

---

## Complete Flow Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  // 1. Configuration
  {
    $id: Now.ID["escalate_high_priority"],
    name: "Escalate High Priority Incidents",
    description: "Escalates P1 incidents to senior team with notification",
    runAs: "system",
    runWithRoles: ["itil"],
    flowPriority: "HIGH"
  },

  // 2. Trigger
  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_created"] },
    {
      table: "incident",
      condition: "priority=1^active=true",
      run_flow_in: "background"
    }
  ),

  // 3. Flow Logic
  _params => {
    // Update record
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["assign_incident"] },
      {
        table_name: "incident",
        record: wfa.dataPill(_params.trigger.current, "reference"),
        values: TemplateValue({
          assignment_group: "1234567890abcdef",
          state: 2
        })
      }
    );

    // Send notification
    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["notify_team"] },
      {
        ah_to: "senior-team@company.com",
        ah_subject: `Critical: ${wfa.dataPill(_params.trigger.current.number, "string")}`,
        ah_body: "A critical incident requires immediate attention."
      }
    );

    // Log action
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_escalation"] },
      {
        log_level: "info",
        log_message: `Escalated incident ${wfa.dataPill(_params.trigger.current.number, "string")}`
      }
    );
  }
);
```

---

## Notes

- Use `get_table_schema` tool to get field definitions and data types before creating/updating records
- Use `run_query` tool to get sys_ids for roles, users, groups, and other reference fields
- Import guideline: `Time`, `Duration`, and `TemplateValue` are available globally from `@servicenow/sdk/global` - do NOT import from `@servicenow/sdk/core`
