# WFA_FLOW_ACTIONS_CONTROL

# Workflow Automation Flow Actions - Control

The Control Actions API reference provides complete technical specifications for flow execution control and logging actions. This document contains API signatures, parameter definitions, types, and syntax examples. For conceptual guidance, when-to-use recommendations, and best practices, refer to skill references.

## action.core.log

Write custom messages to the flow execution log.

### Input Parameters

| Parameter   | Type   | Default | Mandatory | Description                                         |
| ----------- | ------ | ------- | --------- | --------------------------------------------------- |
| log_level   | string | "info"  | Yes       | Log level - must be one of: "info", "warn", "error" |
| log_message | string | -       | Yes       | Message to log (max 255 characters)                 |

### Output Fields

None

### Usage Examples

**Basic Logging:**

```typescript
wfa.action(
  action.core.log,
  { $id: Now.ID["log_start"] },
  {
    log_level: "info",
    log_message: `Processing incident ${wfa.dataPill(_params.trigger.current.number, "string")}`
  }
);
```

**Logging with Data Pills:**

```typescript
// ✅ CORRECT - Use data pills directly in template literal
wfa.action(
  action.core.log,
  { $id: Now.ID["log_details"] },
  {
    log_level: "info",
    log_message: `Incident ${wfa.dataPill(_params.trigger.current.number, "string")} - Priority: ${wfa.dataPill(_params.trigger.current.priority, "string")}, Assigned to: ${wfa.dataPill(_params.trigger.current.assigned_to.name, "string")}`
  }
);
```

**Warning Level:**

```typescript
// ✅ CORRECT - Use data pill directly in template literal
wfa.action(
  action.core.log,
  { $id: Now.ID["log_warning"] },
  {
    log_level: "warn",
    log_message: `Query returned ${wfa.dataPill(lookupResult.Count, "integer")} records - exceeds expected threshold`
  }
);
```

**Error Level:**

```typescript
wfa.action(
  action.core.log,
  { $id: Now.ID["log_error"] },
  {
    log_level: "error",
    log_message: `Failed to send email - email address not configured`
  }
);
```

**Logging in Conditional Logic:**

```typescript
// ✅ CORRECT - Use data pill directly in condition
wfa.flowLogic.if(
  {
    $id: Now.ID["check_priority"],
    condition: `${wfa.dataPill(_params.trigger.current.priority, "integer")}=1`
  },
  () => {
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_critical"] },
      {
        log_level: "info",
        log_message: "Critical priority detected - escalating to management"
      }
    );
  }
);
```

**Logging Action Results:**

```typescript
const createResult = wfa.action(
  action.core.createRecord,
  { $id: Now.ID["create_task"] },
  {
    table_name: "task",
    values: TemplateValue({
      short_description: "Follow-up task"
    })
  }
);

wfa.action(
  action.core.log,
  { $id: Now.ID["log_result"] },
  {
    log_level: "info",
    log_message: `Created task: ${wfa.dataPill(createResult.record, "string")}`
  }
);
```

**Logging in forEach Loop:**

```typescript
const records = wfa.action(
  action.core.lookUpRecords,
  { $id: Now.ID["get_records"] },
  {
    table: "incident",
    conditions: "active=true^priority=1",
    max_results: 10
  }
);

wfa.flowLogic.forEach(
  wfa.dataPill(records.Records, "array.object"),
  { $id: Now.ID["process_incidents"] },
  item => {
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_iteration"] },
      {
        log_level: "info",
        log_message: `Processing incident: ${wfa.dataPill(item.number, "string")}`
      }
    );
  }
);
```

---
