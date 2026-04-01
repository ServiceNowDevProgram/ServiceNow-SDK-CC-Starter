# Sequential Steps Pattern

Multiple actions executed in order - Building complex automation through chained operations.

## Table of Contents

- [When to Use](#when-to-use)
- [Pattern Structure](#pattern-structure)
- [Example 1: Create-Lookup-Update Chain](#example-1-create-lookup-update-chain)
- [Example 2: Escalation Chain](#example-2-escalation-chain)
- [Data Flow Between Actions](#data-flow-between-actions)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)

## When to Use

- Multi-step processes in sequence
- Each action depends on previous output
- Orchestrating multiple table operations

## Pattern Structure

```
Trigger → Action 1 → Action 2 → Action 3 → ... → Action N
```

## Example 1: Create-Lookup-Update Chain

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["incident_auto_assignment"],
    name: "Incident Auto-Assignment",
    description: "Creates incident, finds group, assigns automatically"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["request_created"] },
    {
      table: "sc_request",
      condition: "active=true",
      run_flow_in: "background"
    }
  ),

  _params => {
    const newIncident = wfa.action(
      action.core.createRecord,
      { $id: Now.ID["create_incident"] },
      {
        table_name: "incident",
        values: TemplateValue({
          short_description: wfa.dataPill(
            _params.trigger.current.short_description,
            "string"
          ),
          caller_id: wfa.dataPill(
            _params.trigger.current.opened_by,
            "reference"
          ),
          urgency: "2",
          impact: "2"
        })
      }
    );

    const assignmentGroup = wfa.action(
      action.core.lookUpRecord,
      { $id: Now.ID["find_group"] },
      {
        table: "sys_user_group",
        conditions: "name=Software Support^active=true"
      }
    );

    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["assign_incident"] },
      {
        table_name: "incident",
        record: wfa.dataPill(newIncident.record, "reference"),
        values: TemplateValue({
          assignment_group: wfa.dataPill(assignmentGroup.Record, "reference"),
          state: 2
        })
      }
    );

    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["notify_group"] },
      {
        ah_to: "software-support@company.com",
        ah_subject: `New Incident: ${wfa.dataPill(newIncident.record.number, "string")}`,
        ah_body: `Incident assigned to your group.`
      }
    );
  }
);
```

## Example 2: Escalation Chain

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["progressive_escalation"],
    name: "Progressive Escalation",
    description: "Escalates through multiple notification steps"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["high_priority"] },
    {
      table: "incident",
      condition: "priority=1^active=true",
      run_flow_in: "background"
    }
  ),

  _params => {
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_start"] },
      {
        log_level: "warn",
        log_message: `Starting escalation for: ${wfa.dataPill(_params.trigger.current.number, "string")}`
      }
    );

    const manager = wfa.action(
      action.core.lookUpRecord,
      { $id: Now.ID["find_manager"] },
      {
        table: "sys_user",
        conditions: "user_name=it.manager^active=true"
      }
    );

    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["escalate"] },
      {
        table_name: "incident",
        record: wfa.dataPill(_params.trigger.current, "reference"),
        values: TemplateValue({
          escalation: "1",
          assigned_to: wfa.dataPill(manager.Record, "reference"),
          urgency: "1"
        })
      }
    );

    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["notify_manager"] },
      {
        ah_to: wfa.dataPill(manager.Record.email, "string"),
        ah_subject: `CRITICAL: ${wfa.dataPill(_params.trigger.current.number, "string")}`,
        ah_body: `Critical incident escalated to you. Immediate action required.`
      }
    );

    wfa.action(
      action.core.sendSms,
      { $id: Now.ID["sms_manager"] },
      {
        recipients: `${wfa.dataPill(manager.Record.phone, "string")}`,
        message: `CRITICAL escalation: ${wfa.dataPill(_params.trigger.current.number, "string")}`
      }
    );

    wfa.action(
      action.core.createTask,
      { $id: Now.ID["create_followup"] },
      {
        task_table: "task",
        field_values: TemplateValue({
          short_description: `Follow up on ${wfa.dataPill(_params.trigger.current.number, "string")}`,
          assigned_to: wfa.dataPill(manager.Record, "reference"),
          due_date: wfa.dataPill(
            _params.trigger.current.due_date,
            "glide_date_time"
          ),
          parent: wfa.dataPill(_params.trigger.current, "reference")
        }),
        wait: false
      }
    );
  }
);
```

## Data Flow Between Actions

Actions can reference outputs from previous steps:

```typescript
const step1 = wfa.action(...);  // Returns { record: sys_id } or { Record: sys_id }
const step2 = wfa.action(..., {
  record: wfa.dataPill(step1.record, 'reference')  // Use step1 output
});
const step3 = wfa.action(..., {
  field: wfa.dataPill(step2.SomeField, 'string')  // Use step2 output
});
```

## Best Practices

1. **Plan the Sequence** - Map out dependencies before implementing
2. **Descriptive IDs** - Use clear Now.ID names (e.g., `find_manager`, `send_notification`)
3. **Store Outputs** - Save action results in variables when needed later in flow
4. **Check Dependencies** - Ensure each step has required data from previous steps
5. **Log Key Steps** - Add logging for debugging (only when explicitly requested)

## Common Use Cases

- End-to-end incident management
- Progressive escalation workflows (notify → escalate → create task)
- Complex record creation with lookups (create → lookup assignment → update → notify)
- Data synchronization across tables

## Next Steps

For Fluent API signatures, parameters, outputs, and complete coding examples, use `get_knowledge_source` tool to retrieve:

- **WFA_FLOW_TRIGGER_RECORD** - Record trigger API (created, updated, createdOrUpdated)
- **WFA_FLOW_ACTIONS_TABLE** - Table action APIs (lookUpRecord, updateRecord, createRecord)
- **WFA_FLOW_ACTIONS_COMMUNICATION** - Communication action APIs (sendEmail, sendNotification)

These knowledge sources contain authoritative API documentation with all parameters, data types, and working examples.
