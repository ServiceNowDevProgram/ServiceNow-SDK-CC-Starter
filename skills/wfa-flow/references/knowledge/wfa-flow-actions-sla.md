# WFA_FLOW_ACTIONS_SLA

# Workflow Automation Flow Actions - SLA

The SLA Actions API reference provides complete technical specifications for SLA-based workflow timing actions. This document contains action signatures, parameter definitions, types, and syntax examples. For conceptual guidance, when-to-use recommendations, and best practices, refer to skill references.

---

## action.core.slaPercentageTimer

Pauses flow execution until a specified percentage of SLA time has elapsed.

### Input Parameters

| Parameter       | Type      | Default | Mandatory | Description                                                                                                                                               |
| --------------- | --------- | ------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| percentage      | integer   | -       | Yes       | Percentage of SLA duration to wait (0-100)                                                                                                                |
| task_sla_record | reference | -       | No        | Task SLA record reference from task_sla table                                                                                                             |
| sla_flow_inputs | object    | -       | No        | SLA configuration details with fields: duration (string), relative_duration_works_on (string), is_repair (boolean), duration_type (string), name (string) |

### Output Fields

| Field                   | Type     | Description                                                                                 |
| ----------------------- | -------- | ------------------------------------------------------------------------------------------- |
| status                  | choice   | Status of the SLA timer: completed, paused, repair, skipped, cancelled (default: completed) |
| scheduled_end_date_time | datetime | Scheduled end date/time for the SLA                                                         |
| total_duration          | duration | Total duration of the SLA                                                                   |

### SLA Status Values

- `completed` - SLA percentage timer completed successfully
- `paused` - SLA timer is paused
- `repair` - SLA is in repair mode
- `skipped` - SLA timer was skipped
- `cancelled` - SLA timer was cancelled

### Usage Examples

**Basic SLA Percentage Timer:**

```typescript
// Wait until 75% of SLA time has elapsed
wfa.action(
  action.core.slaPercentageTimer,
  { $id: Now.ID["wait_75_percent"] },
  {
    percentage: 75
  }
);
```

**Progressive Escalation at Multiple SLA Milestones:**

```typescript
// 50% milestone
wfa.action(
  action.core.slaPercentageTimer,
  { $id: Now.ID["wait_50_percent"] },
  { percentage: 50 }
);

wfa.action(
  action.core.sendEmail,
  { $id: Now.ID["notify_50"] },
  {
    ah_to: wfa.dataPill(_params.trigger.current.assigned_to.email, "string"),
    ah_subject: "SLA Reminder - 50% Time Elapsed",
    ah_body: `Incident at 50% of SLA time.`,
    record: wfa.dataPill(_params.trigger.current, "reference"),
    table_name: "incident"
  }
);

// 75% milestone
wfa.action(
  action.core.slaPercentageTimer,
  { $id: Now.ID["wait_75_percent"] },
  { percentage: 75 }
);

wfa.action(
  action.core.sendEmail,
  { $id: Now.ID["notify_75"] },
  {
    ah_to: wfa.dataPill(
      _params.trigger.current.assigned_to.manager.email,
      "string"
    ),
    ah_subject: "SLA Escalation - 75% Time Elapsed",
    ah_body: `Incident at 75% of SLA time.`,
    record: wfa.dataPill(_params.trigger.current, "reference"),
    table_name: "incident"
  }
);
```

**SLA Timer with Task SLA Record Reference:**

```typescript
const slaTimer = wfa.action(
  action.core.slaPercentageTimer,
  { $id: Now.ID["sla_timer_specific"] },
  {
    percentage: 80
  }
);

wfa.flowLogic.if(
  {
    $id: Now.ID["check_sla_status"],
    condition: `${wfa.dataPill(slaTimer.status, "string")}=completed`
  },
  () => {
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_sla_completed"] },
      {
        log_level: "info",
        log_message: `SLA timer completed at 80%`
      }
    );
  }
);
```

**SLA-Based Priority Escalation:**

```typescript
wfa.action(
  action.core.slaPercentageTimer,
  { $id: Now.ID["wait_70_percent"] },
  { percentage: 70 }
);

// ✅ CORRECT - Use data pill directly in condition
wfa.flowLogic.if(
  {
    $id: Now.ID["check_priority"],
    condition: `${wfa.dataPill(_params.trigger.current.priority, "integer")}>1`
  },
  () => {
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["escalate_priority"] },
      {
        table_name: "incident",
        record: wfa.dataPill(_params.trigger.current, "reference"),
        values: TemplateValue({
          priority: 1,
          comments: "Priority escalated due to 70% SLA time elapsed"
        })
      }
    );
  }
);
```

---
