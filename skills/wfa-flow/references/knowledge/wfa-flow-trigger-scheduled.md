# WFA_FLOW_TRIGGER_SCHEDULED

The Scheduled Triggers API reference provides complete technical specifications for time-based flow triggers. This document contains trigger signatures, configuration parameters, output fields, and syntax examples for Daily, Weekly, Monthly, Repeat, and Run Once triggers. For conceptual guidance, when-to-use recommendations, and implementation patterns, refer to skill references.

## Trigger Usage Pattern

All scheduled triggers follow the same usage pattern:

```typescript
wfa.trigger(
  trigger.scheduled.<triggerType>,
  { $id: Now.ID['trigger_id'] },
  {
    // time-based configuration parameters
  }
)
```

---

## trigger.scheduled.daily

Activates once per day at a specified time.

### Configuration Parameters

| Parameter | Type | Default | Mandatory | Description                                  |
| --------- | ---- | ------- | --------- | -------------------------------------------- |
| time      | Time | -       | Yes       | Time of day to run (hours, minutes, seconds) |

**Time Constructor:**

```typescript
Time(
  { hours: number, minutes: number, seconds: number },
  timezone: string  // e.g., 'UTC', 'America/Los_Angeles'
)
```

### Output Fields

| Field               | Type            | Description                    |
| ------------------- | --------------- | ------------------------------ |
| run_start_time      | glide_date_time | Flow execution start time      |
| run_start_date_time | glide_date_time | Flow execution start date/time |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.run_start_date_time, "glide_date_time");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["daily_cleanup"],
    name: "Daily Cleanup at 2 AM"
  },

  wfa.trigger(
    trigger.scheduled.daily,
    { $id: Now.ID["daily_trigger"] },
    {
      time: Time(
        {
          hours: 2,
          minutes: 0,
          seconds: 0
        },
        "UTC"
      )
    }
  ),

  _params => {
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_start"] },
      {
        log_level: "info",
        log_message: "Daily cleanup job started"
      }
    );

    // Look up old records to delete
    const oldRecords = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["lookup_old_records"] },
      {
        table: "temp_table",
        conditions: "sys_created_onRELATIVELE@dayofweek@ago@7",
        max_results: 1000
      }
    );

    // Delete each old record
    wfa.flowLogic.forEach(
      wfa.dataPill(oldRecords.Records, "array.object"),
      { $id: Now.ID["delete_loop"], annotation: "Delete old records" },
      record => {
        wfa.action(
          action.core.deleteRecord,
          { $id: Now.ID["delete_record"] },
          {
            record: `${wfa.dataPill(record, "reference")}`
          }
        );
      }
    );
  }
);
```

---

## trigger.scheduled.weekly

Activates once per week on a specified day and time.

### Configuration Parameters

| Parameter   | Type    | Default | Mandatory | Description                                                                                |
| ----------- | ------- | ------- | --------- | ------------------------------------------------------------------------------------------ |
| day_of_week | integer | -       | Yes       | Day of week (1=Sunday, 2=Monday, 3=Tuesday, 4=Wednesday, 5=Thursday, 6=Friday, 7=Saturday) |
| time        | Time    | -       | Yes       | Time of day to run (hours, minutes, seconds)                                               |

**day_of_week Values:**

- `1` - Sunday
- `2` - Monday
- `3` - Tuesday
- `4` - Wednesday
- `5` - Thursday
- `6` - Friday
- `7` - Saturday

### Output Fields

| Field               | Type            | Description                    |
| ------------------- | --------------- | ------------------------------ |
| run_start_time      | glide_date_time | Flow execution start time      |
| run_start_date_time | glide_date_time | Flow execution start date/time |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.run_start_date_time, "glide_date_time");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["weekly_report"],
    name: "Weekly Report Every Monday"
  },

  wfa.trigger(
    trigger.scheduled.weekly,
    { $id: Now.ID["weekly_trigger"] },
    {
      day_of_week: 2, // Monday
      time: Time(
        {
          hours: 9,
          minutes: 0,
          seconds: 0
        },
        "UTC"
      )
    }
  ),

  _params => {
    // ✅ CORRECT - Use data pill directly in template literal
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_report"] },
      {
        log_level: "info",
        log_message: `Weekly report generated at ${wfa.dataPill(_params.trigger.run_start_date_time, "glide_date_time")}`
      }
    );

    // Generate and send report
  }
);
```

---

## trigger.scheduled.monthly

Activates once per month on a specified day and time.

### Configuration Parameters

| Parameter    | Type    | Default | Mandatory | Description                                  |
| ------------ | ------- | ------- | --------- | -------------------------------------------- |
| day_of_month | integer | -       | Yes       | Day of month (1-31) to run                   |
| time         | Time    | -       | Yes       | Time of day to run (hours, minutes, seconds) |

**Note:** If `day_of_month` is greater than the number of days in the month (e.g., 31 for February), the flow will run on the last day of that month.

### Output Fields

| Field               | Type            | Description                    |
| ------------------- | --------------- | ------------------------------ |
| run_start_time      | glide_date_time | Flow execution start time      |
| run_start_date_time | glide_date_time | Flow execution start date/time |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.run_start_date_time, "glide_date_time");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["monthly_audit"],
    name: "Monthly Audit on 1st Day"
  },

  wfa.trigger(
    trigger.scheduled.monthly,
    { $id: Now.ID["monthly_trigger"] },
    {
      day_of_month: 1,
      time: Time(
        {
          hours: 0,
          minutes: 0,
          seconds: 0
        },
        "UTC"
      )
    }
  ),

  _params => {
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_audit_start"] },
      {
        log_level: "info",
        log_message: "Monthly audit started"
      }
    );

    // Perform monthly audit operations
    const records = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["find_records"] },
      {
        table: "incident",
        conditions:
          "sys_created_onONLast month@javascript:gs.beginningOfLastMonth()@javascript:gs.endOfLastMonth()",
        max_results: 1000
      }
    );

    wfa.action(
      action.core.log,
      { $id: Now.ID["log_count"] },
      {
        log_level: "info",
        log_message: `Found ${wfa.dataPill(records.Count, "integer")} incidents from last month`
      }
    );
  }
);
```

---

## trigger.scheduled.repeat

Activates repeatedly at a specified interval.

### Configuration Parameters

| Parameter | Type     | Default | Mandatory | Description                          |
| --------- | -------- | ------- | --------- | ------------------------------------ |
| repeat    | Duration | -       | Yes       | Interval duration between executions |

**Duration Constructor:**

```typescript
Duration({ days?: number, hours?: number, minutes?: number, seconds?: number })
```

**Common Patterns:**

- Every 15 minutes: `Duration({ minutes: 15 })`
- Every hour: `Duration({ hours: 1 })`
- Every 6 hours: `Duration({ hours: 6 })`
- Every day: `Duration({ days: 1 })`

### Output Fields

| Field               | Type            | Description                    |
| ------------------- | --------------- | ------------------------------ |
| run_start_time      | glide_date_time | Flow execution start time      |
| run_start_date_time | glide_date_time | Flow execution start date/time |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.run_start_date_time, "glide_date_time");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["health_check"],
    name: "Health Check Every 15 Minutes"
  },

  wfa.trigger(
    trigger.scheduled.repeat,
    { $id: Now.ID["repeat_trigger"] },
    {
      repeat: Duration({ minutes: 15 })
    }
  ),

  _params => {
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_health_check"] },
      {
        log_level: "info",
        log_message: "Health check running"
      }
    );

    // Check system health
    const unassignedCount = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["check_unassigned"] },
      {
        table: "incident",
        conditions: "assigned_toISEMPTY^priority=1^active=true",
        max_results: 100
      }
    );

    // ✅ CORRECT - Use data pill directly in condition
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_threshold"],
        condition: `${wfa.dataPill(unassignedCount.Count, "integer")}>10`
      },
      () => {
        wfa.action(
          action.core.sendEmail,
          { $id: Now.ID["alert_email"] },
          {
            ah_to: "admin@company.com",
            ah_subject: "Alert: High number of unassigned incidents",
            ah_body: "There are more than 10 unassigned P1 incidents."
          }
        );
      }
    );
  }
);
```

---

## trigger.scheduled.runOnce

Activates once at a specified date and time.

### Configuration Parameters

| Parameter | Type     | Default | Mandatory | Description                            |
| --------- | -------- | ------- | --------- | -------------------------------------- |
| run_in    | datetime | -       | Yes       | Specific date and time to run the flow |

**run_in Format:**

- Use ISO 8601 format: `'YYYY-MM-DD HH:MM:SS'`
- Example: `'2026-03-15 14:30:00'`

### Output Fields

| Field               | Type            | Description                    |
| ------------------- | --------------- | ------------------------------ |
| run_start_time      | glide_date_time | Flow execution start time      |
| run_start_date_time | glide_date_time | Flow execution start date/time |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.run_start_date_time, "glide_date_time");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["scheduled_maintenance"],
    name: "Scheduled Maintenance on March 15"
  },

  wfa.trigger(
    trigger.scheduled.runOnce,
    { $id: Now.ID["run_once_trigger"] },
    {
      run_in: "2026-03-15 02:00:00"
    }
  ),

  _params => {
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_maintenance_start"] },
      {
        log_level: "info",
        log_message: "Scheduled maintenance started"
      }
    );

    // Perform one-time maintenance operations
    wfa.action(
      action.core.updateMultipleRecords,
      { $id: Now.ID["update_records"] },
      {
        table_name: "cmdb_ci",
        conditions: "install_status=1",
        field_values: TemplateValue({
          u_maintenance_performed: "true",
          u_last_maintenance_date: wfa.dataPill(
            _params.trigger.run_start_date_time,
            "datetime"
          )
        })
      }
    );

    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["notify_complete"] },
      {
        ah_to: "admin@company.com",
        ah_subject: "Scheduled maintenance completed",
        ah_body: "The scheduled maintenance has been completed successfully."
      }
    );
  }
);
```

---

## Notes

- `Time` constructor is available globally from `@servicenow/sdk/global` - do NOT import from `@servicenow/sdk/core`
- `Duration` constructor is available globally from `@servicenow/sdk/global` - do NOT import from `@servicenow/sdk/core`
- For `repeat` trigger, minimum interval is typically 1 minute (check instance configuration)
- For `runOnce` trigger, the flow will execute once at the specified time and then become inactive
- For `monthly` trigger, if the specified day exceeds the month's days, it runs on the last day of that month
- Scheduled triggers do not have access to record data - use actions to query data within the flow
- All scheduled triggers output `run_start_time` and `run_start_date_time` for tracking execution timing
