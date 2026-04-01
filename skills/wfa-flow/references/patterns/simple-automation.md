# Simple Automation Pattern

Single trigger + single action - The foundational flow pattern.

## Table of Contents

- [When to Use](#when-to-use)
- [Pattern Structure](#pattern-structure)
- [Example 1: Auto-Assign High-Priority Incidents](#example-1-auto-assign-high-priority-incidents)
- [Example 2: Send Notification on Assignment](#example-2-send-notification-on-assignment)
- [Example 3: Create Task on Incident](#example-3-create-task-on-incident)
- [Example 4: Daily Scheduled Report](#example-4-daily-scheduled-report)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Tool Integration](#tool-integration)

## When to Use

- Simple, linear automation
- Direct response to trigger
- Single operation per trigger
- No branching or complex logic

## Pattern Structure

```
Trigger (When) → Action (What)
```

## Example 1: Auto-Assign High-Priority Incidents

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["auto_assign_high_priority"],
    name: "Auto-Assign High Priority",
    description: "Assigns critical incidents to on-call engineer"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_created"] },
    {
      table: "incident",
      condition: "priority<=2^active=true",
      run_flow_in: "background"
    }
  ),

  _params => {
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["assign_oncall"] },
      {
        table_name: "incident",
        record: wfa.dataPill(_params.trigger.current, "reference"),
        values: TemplateValue({
          assigned_to: "<oncall_engineer_sys_id>",
          assignment_group: "<it_support_group_sys_id>",
          state: 2
        })
      }
    );
  }
);
```

## Example 2: Send Notification on Assignment

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["notify_on_assignment"],
    name: "Notify on Assignment",
    description: "Sends notification when incident assigned"
  },

  wfa.trigger(
    trigger.record.updated,
    { $id: Now.ID["incident_assigned"] },
    {
      table: "incident",
      condition: "assigned_toISNOTEMPTY",
      trigger_strategy: "unique_changes",
      run_flow_in: "background"
    }
  ),

  _params => {
    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["notify_assignee"] },
      {
        ah_to: wfa.dataPill(
          _params.trigger.current.assigned_to.email,
          "string"
        ),
        ah_subject: `Incident ${wfa.dataPill(_params.trigger.current.number, "string")} assigned`,
        ah_body:
          "You have been assigned a new incident. Please check your assignments." // Static string (data pills not supported in ah_body)
      }
    );
  }
);
```

## Example 3: Create Task on Incident

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["create_followup_task_flow"],
    name: "Create Follow-up Task",
    description: "Creates task for every new incident"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_created"] },
    {
      table: "incident",
      condition: "priority<=3",
      run_flow_in: "background"
    }
  ),

  _params => {
    wfa.action(
      action.core.createTask,
      { $id: Now.ID["create_followup"] },
      {
        task_table: "task",
        field_values: TemplateValue({
          short_description: `Follow-up: ${wfa.dataPill(_params.trigger.current.number, "string")}`,
          description: "Review and validate incident details",
          assigned_to: wfa.dataPill(
            _params.trigger.current.assigned_to,
            "reference"
          ),
          parent: wfa.dataPill(_params.trigger.current, "reference")
        })
      }
    );
  }
);
```

## Example 4: Daily Scheduled Report

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["daily_incident_summary"],
    name: "Daily Incident Summary",
    description: "Sends daily email with open incident count"
  },

  wfa.trigger(
    trigger.scheduled.daily,
    { $id: Now.ID["daily_9am"] },
    {
      time: Time({ hours: 9, minutes: 0, seconds: 0 }, "America/Los_Angeles")
    }
  ),

  _params => {
    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["send_summary"] },
      {
        ah_to: "it-management@company.com",
        ah_subject: "Daily Incident Summary",
        ah_body: `Daily summary of open incidents. Use run_query to get counts.`
      }
    );
  }
);
```

## Best Practices

1. **Keep It Simple** - If you need multiple actions, use sequential-steps pattern
2. **Use Specific Conditions** - Filter triggers to only fire when truly needed (e.g., `priority<=2^active=true`)
3. **Background Execution** - Use `run_flow_in: 'background'` to avoid blocking users
4. **Test Conditions** - Verify trigger conditions match expected records

## Common Use Cases

- Auto-assignment based on criteria
- Immediate notifications
- Simple record updates
- Scheduled single-action tasks (daily reports, batch operations)

## Next Steps

For Fluent API signatures, parameters, outputs, and complete coding examples, use `get_knowledge_source` tool to retrieve:

- **WFA_FLOW_TRIGGER_RECORD** - Record trigger API (created, updated, createdOrUpdated)
- **WFA_FLOW_ACTIONS_TABLE** - Table action APIs (createRecord, updateRecord, lookUpRecord)
- **WFA_FLOW_ACTIONS_COMMUNICATION** - Communication action APIs (sendEmail, sendNotification)

These knowledge sources contain authoritative API documentation with all parameters, data types, and working examples.
