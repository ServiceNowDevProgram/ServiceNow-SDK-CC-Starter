# Monitoring Pattern

Periodic checking pattern using scheduled triggers to monitor record conditions over time.

## Table of Contents

- [When to Use This Pattern](#when-to-use-this-pattern)
- [Temporal Keyword Mapping](#temporal-keyword-mapping)
- [Pattern Structure](#pattern-structure)
- [Example: Daily Incident Monitoring](#example-daily-incident-monitoring)
- [Common Mistakes](#common-mistakes)
- [When NOT to Use](#when-not-to-use)
- [Next Steps](#next-steps)

## When to Use This Pattern

Use when requirements include **temporal continuity** - ongoing checks, periodic actions, or "while" conditions:

- "While incidents are active, check X every day"
- "Monitor all open cases hourly"
- "Every morning, review high-priority tickets"
- "Periodically remind assignees of pending tasks"

**Key Distinction:**

- **"When X happens, do Y"** → Use **Record Trigger** (event-driven, fires once)
- **"While X condition, do Y periodically"** → Use **Scheduled Trigger** (time-driven, repeats)

## Temporal Keyword Mapping

| User Says                        | Trigger Type      | Pattern            |
| -------------------------------- | ----------------- | ------------------ |
| "when created", "when updated"   | Record Trigger    | Simple automation  |
| "while active", "as long as"     | Scheduled Trigger | Monitoring pattern |
| "every day/hour", "periodically" | Scheduled Trigger | Monitoring pattern |
| "monitor", "check regularly"     | Scheduled Trigger | Monitoring pattern |

## Pattern Structure

```
Scheduled Trigger → lookUpRecords (matching criteria) → forEach → Action
```

## Example: Daily Incident Monitoring

**Requirement:** "While incidents are active, log their status every day at 9 AM"

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["monitor_active_incidents"],
    name: "Monitor Active Incidents Daily"
  },

  wfa.trigger(
    trigger.scheduled.daily,
    { $id: Now.ID["daily_9am"] },
    { time: Time({ hours: 9, minutes: 0, seconds: 0 }, "UTC") }
  ),

  _params => {
    const activeIncidents = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["find_active"] },
      {
        table: "incident",
        conditions: "active=true^state<6",
        max_results: 100
      }
    );

    wfa.flowLogic.forEach(
      wfa.dataPill(activeIncidents.Records, "array.object"),
      { $id: Now.ID["process_each"] },
      incident => {
        wfa.action(
          action.core.log,
          { $id: Now.ID["log_status"] },
          {
            log_level: "info",
            log_message: `Incident ${wfa.dataPill(incident.number, "string")} - Priority: ${wfa.dataPill(incident.priority, "string")}`
          }
        );
      }
    );
  }
);
```

## Common Mistakes

❌ **Using Record Trigger for Monitoring** - Record trigger fires once on event. Won't re-check condition periodically.

❌ **Forgetting max_results** - lookUpRecords without limit can cause timeout on large datasets.

## When NOT to Use

- One-time event response (use Record Trigger instead)
- Real-time immediate reaction (use Record Trigger instead)

## Next Steps

For Fluent API signatures, parameters, outputs, and complete coding examples, use `get_knowledge_source` tool to retrieve:

- **WFA_FLOW_TRIGGER_SCHEDULED** - Scheduled trigger API (daily, weekly, monthly, repeat, runOnce) with Time and Duration specifications
- **WFA_FLOW_ACTIONS_TABLE** - Table action APIs (lookUpRecords, updateRecord, updateMultipleRecords)
- **WFA_FLOW_ACTIONS_COMMUNICATION** - Communication action APIs (sendEmail, sendNotification)
- **WFA_FLOW_ACTIONS_CONTROL** - Control action API (log)
- **WFA_FLOW_LOGICS** - Flow logic API (forEach, if/elseIf/else) with condition syntax

These knowledge sources contain authoritative API documentation with all parameters, data types, and working examples.
