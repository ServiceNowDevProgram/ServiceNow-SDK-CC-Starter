# Record Iteration Pattern

forEach loops to process multiple records with consistent logic.

## Table of Contents

- [When to Use](#when-to-use)
- [Pattern Structure](#pattern-structure)
- [Example 1: Notify Multiple Users](#example-1-notify-multiple-users)
- [Example 2: Bulk Update with Loop Control](#example-2-bulk-update-with-loop-control)
- [Loop Control](#loop-control)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Alternative: updateMultipleRecords](#alternative-updatemultiplerecords)

## When to Use

- Process multiple records with same logic
- Bulk updates or notifications
- Iterating through lookup results
- Multi-record operations

## Pattern Structure

```
Trigger → Lookup Records → forEach(record) { Actions }
```

## Example 1: Notify Multiple Users

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["notify_users"],
    name: "Notify Multiple Users",
    description: "Sends email to active users"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_created"] },
    {
      table: "incident",
      condition: "priority=1^active=true",
      run_flow_in: "background"
    }
  ),

  _params => {
    const activeUsers = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["find_users"] },
      {
        table: "sys_user",
        conditions: "active=true^roles!=",
        max_results: 50
      }
    );

    wfa.flowLogic.forEach(
      wfa.dataPill(activeUsers.Records, "records"),
      { $id: Now.ID["loop_users"] },
      user => {
        wfa.action(
          action.core.sendEmail,
          { $id: Now.ID["email_user"] },
          {
            ah_to: wfa.dataPill(user.email, "string"),
            ah_subject: "Incident notification",
            ah_body:
              "An incident requires your attention. Please check ServiceNow for details.",
            record: wfa.dataPill(_params.trigger.current, "reference"),
            table_name: "incident"
          }
        );
      }
    );
  }
);
```

## Example 2: Bulk Update with Loop Control

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["bulk_update_stale_incidents"],
    name: "Bulk Update Stale Incidents",
    description: "Updates stale incidents with processing limit"
  },

  wfa.trigger(
    trigger.scheduled.daily,
    { $id: Now.ID["daily_check"] },
    {
      time: Time({ hours: 8, minutes: 0, seconds: 0 }, "America/New_York")
    }
  ),

  _params => {
    wfa.flowLogic.setFlowVariables(
      { $id: Now.ID["init_counter"] },
      {
        processed_count: 0
      }
    );

    const staleIncidents = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["find_stale"] },
      {
        table: "incident",
        conditions: "state!=6^state!=7^sys_updated_on<javascript:gs.daysAgo(7)",
        max_results: 100
      }
    );

    wfa.flowLogic.forEach(
      wfa.dataPill(staleIncidents.Records, "array.object"),
      { $id: Now.ID["loop_incidents"] },
      incident => {
        wfa.flowLogic.if(
          {
            $id: Now.ID["check_limit"],
            condition: `${wfa.dataPill(_params.flowVariables.processed_count, "string")}>=50`
          },
          () => {
            wfa.flowLogic.exitLoop({
              $id: Now.ID["exit_at_limit"]
            });
          }
        );

        wfa.flowLogic.if(
          {
            $id: Now.ID["check_assigned"],
            condition: `${wfa.dataPill(incident.assigned_to, "string")}ISNOTEMPTY`
          },
          () => {
            wfa.flowLogic.skipIteration({
              $id: Now.ID["skip_assigned"]
            });
          }
        );

        wfa.action(
          action.core.updateRecord,
          { $id: Now.ID["escalate"] },
          {
            table_name: "incident",
            record: wfa.dataPill(incident.sys_id, "reference"),
            values: TemplateValue({
              escalation: "1",
              comments: "Escalated due to inactivity > 7 days"
            })
          }
        );

        wfa.flowLogic.setFlowVariables(
          { $id: Now.ID["increment"] },
          {
            processed_count:
              wfa.dataPill(_params.flowVariables.processed_count, "integer") + 1
          }
        );
      }
    );
  }
);
```

## Loop Control

**exitLoop** - Stops loop entirely and continues after loop:

```typescript
wfa.flowLogic.exitLoop({ $id: Now.ID["stop_loop"] });
```

**skipIteration** - Skips current iteration, continues with next:

```typescript
wfa.flowLogic.skipIteration({ $id: Now.ID["skip_current"] });
```

## Best Practices

1. **Limit Record Count** - Use `max_results` (recommend 50-100 max) to avoid timeouts
2. **Use Loop Control** - exitLoop for processing limits, skipIteration for conditional filtering
3. **Handle Empty Arrays** - Check `Count>0` before forEach to avoid empty loop execution
4. **Avoid Nested Loops** - They multiply execution time exponentially

## Common Use Cases

- Notify all group members
- Bulk update matching records with per-record logic
- Send reminders to multiple users
- Create multiple related records

## Alternative: updateMultipleRecords

For simple bulk field updates without per-record logic, use updateMultipleRecords instead (faster, no loop overhead):

```typescript
wfa.action(
  action.core.updateMultipleRecords,
  { $id: Now.ID["bulk_update"] },
  {
    table_name: "incident",
    conditions: "state=1^priority=1",
    values: TemplateValue({ state: 2, urgency: 1 })
  }
);
```

## Next Steps

For Fluent API signatures, parameters, outputs, and complete coding examples, use `get_knowledge_source` tool to retrieve:

- **WFA_FLOW_TRIGGER_RECORD** - Record trigger API (created, updated, createdOrUpdated)
- **WFA_FLOW_TRIGGER_SCHEDULED** - Scheduled trigger API (daily, weekly, monthly, repeat)
- **WFA_FLOW_ACTIONS_TABLE** - Table action APIs (lookUpRecords, updateRecord, updateMultipleRecords)
- **WFA_FLOW_LOGICS** - Flow logic API (forEach, if/elseIf/else, skipIteration, exitLoop)

These knowledge sources contain authoritative API documentation with all parameters, data types, and working examples.
