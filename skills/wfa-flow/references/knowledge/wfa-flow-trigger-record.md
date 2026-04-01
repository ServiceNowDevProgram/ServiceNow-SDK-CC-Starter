# WFA_FLOW_TRIGGER_RECORD

The Record Triggers API reference provides complete technical specifications for record-based flow triggers. This document contains trigger signatures, configuration parameters, output fields, and syntax examples for Created, Updated, and Created or Updated triggers. For conceptual guidance, when-to-use recommendations, and implementation patterns, refer to skill references.

## Trigger Usage Pattern

All record triggers follow the same usage pattern:

```typescript
wfa.trigger(
  trigger.record.<triggerType>,
  { $id: Now.ID['trigger_id'] },
  {
    table: 'table_name',
    condition: 'encoded_query',
    // ... other configuration parameters
  }
)
```

---

## trigger.record.created

Activates when a new record is created in the specified table.

### Configuration Parameters

| Parameter             | Type                                         | Default | Mandatory | Description                                                  |
| --------------------- | -------------------------------------------- | ------- | --------- | ------------------------------------------------------------ |
| table                 | string                                       | -       | Yes       | Table to monitor for new records                             |
| condition             | string                                       | -       | No        | Encoded query to filter which records trigger the flow       |
| run_flow_in           | 'any' \| 'background' \| 'foreground'        | 'any'   | No        | Execution context for the flow                               |
| run_on_extended       | 'true' \| 'false'                            | 'false' | No        | Whether to run on child tables                               |
| run_when_setting      | 'both' \| 'interactive' \| 'non_interactive' | 'both'  | No        | Session type filter                                          |
| run_when_user_setting | 'any' \| 'one_of' \| 'not_one_of'            | 'any'   | No        | User-based filter mode                                       |
| run_when_user_list    | string[]                                     | []      | No        | List of user sys_ids for filtering (requires run_query tool) |

**Tools needed:** Use `get_table_schema` to get table field definitions, `run_query` to get user sys_ids for run_when_user_list.

### Output Fields

| Field               | Type            | Description                    |
| ------------------- | --------------- | ------------------------------ |
| current             | reference       | The created record             |
| table_name          | string          | Table where record was created |
| run_start_time      | glide_date_time | Flow execution start time      |
| run_start_date_time | glide_date_time | Flow execution start date/time |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.current, "reference");
  wfa.dataPill(_params.trigger.current.field_name, "string");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["auto_assign_new_incidents"],
    name: "Auto-Assign New High Priority Incidents"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_created_trigger"] },
    {
      table: "incident",
      condition: "priority=1^active=true",
      run_flow_in: "background",
      run_on_extended: "false",
      run_when_setting: "both",
      run_when_user_setting: "any"
    }
  ),

  _params => {
    // ✅ Log creation - use data pills directly in template literal
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_creation"] },
      {
        log_level: "info",
        log_message: `New incident created: ${wfa.dataPill(_params.trigger.current.number, "string")} with priority ${wfa.dataPill(_params.trigger.current.priority, "string")}`
      }
    );
  }
);
```

### Advanced Reference Access Patterns

**Multi-Level Dot-Walking:**

Record triggers support accessing nested reference fields up to multiple levels deep. This allows you to traverse relationships between tables.

```typescript
// Access nested reference fields (3 levels deep)
Flow(
  { $id: Now.ID["nested_reference_example"], name: "Nested Reference Access" },
  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["trigger"] },
    { table: "incident" }
  ),
  _params => {
    // ✅ Access nested reference fields directly in template literal
    // Level 1: _params.trigger.current.assignment_group (reference)
    // Level 2: _params.trigger.current.assignment_group.name (string from referenced record)

    wfa.action(
      action.core.log,
      { $id: Now.ID["log"] },
      {
        log_level: "info",
        log_message: `Incident assigned to group: ${wfa.dataPill(_params.trigger.current.assignment_group.name, "string")} - ${wfa.dataPill(_params.trigger.current.assignment_group.description, "string")}`
      }
    );
  }
);
```

**Multiple Data Pills in Template Strings:**

Combine multiple data pills in a single message for comprehensive logging or notifications.

```typescript
Flow(
  { $id: Now.ID["comprehensive_log"], name: "Comprehensive Logging" },
  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["trigger"] },
    { table: "incident" }
  ),
  _params => {
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_details"] },
      {
        log_level: "info",
        log_message: `Incident Created - Number: ${wfa.dataPill(_params.trigger.current.number, "string")}, Priority: ${wfa.dataPill(_params.trigger.current.priority, "string")}, Table: ${wfa.dataPill(_params.trigger.table_name, "string")}, Time: ${wfa.dataPill(_params.trigger.run_start_date_time, "glide_date_time")}, Group: ${wfa.dataPill(_params.trigger.current.assignment_group.name, "string")}`
      }
    );
  }
);
```

---

## trigger.record.updated

Activates when an existing record is updated in the specified table.

### Configuration Parameters

| Parameter             | Type                                              | Default | Mandatory | Description                                                  |
| --------------------- | ------------------------------------------------- | ------- | --------- | ------------------------------------------------------------ |
| table                 | string                                            | -       | Yes       | Table to monitor for updated records                         |
| condition             | string                                            | -       | No        | Encoded query to filter which records trigger the flow       |
| run_flow_in           | 'any' \| 'background' \| 'foreground'             | 'any'   | No        | Execution context for the flow                               |
| run_on_extended       | 'true' \| 'false'                                 | 'false' | No        | Whether to run on child tables                               |
| run_when_setting      | 'both' \| 'interactive' \| 'non_interactive'      | 'both'  | No        | Session type filter                                          |
| run_when_user_setting | 'any' \| 'one_of' \| 'not_one_of'                 | 'any'   | No        | User-based filter mode                                       |
| run_when_user_list    | string[]                                          | []      | No        | List of user sys_ids for filtering (requires run_query tool) |
| trigger_strategy      | 'once' \| 'unique_changes' \| 'every' \| 'always' | 'once'  | No        | Controls when trigger fires for updates                      |

**trigger_strategy Values:**

- `'once'` - Fires only the first time condition matches
- `'unique_changes'` - Fires for each unique change to monitored fields
- `'every'` - Fires on every update regardless of field changes
- `'always'` - Fires only if flow is not currently running

**Tools needed:** Use `get_table_schema` to get table field definitions, `run_query` to get user sys_ids for run_when_user_list.

### Output Fields

| Field               | Type            | Description                                                     |
| ------------------- | --------------- | --------------------------------------------------------------- |
| current             | reference       | The updated record (current state)                              |
| changed_fields      | array           | Array of changed field details with previous and current values |
| table_name          | string          | Table where record was updated                                  |
| run_start_time      | glide_date_time | Flow execution start time                                       |
| run_start_date_time | glide_date_time | Flow execution start date/time                                  |

**changed_fields Structure:**

```typescript
{
  field_name: string,
  previous_value: string,
  previous_display_value: string,
  current_value: string,
  current_display_value: string
}
```

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.current, "reference");
  wfa.dataPill(_params.trigger.changed_fields, "array.object");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["escalate_priority_change"],
    name: "Escalate When Priority Changes"
  },

  wfa.trigger(
    trigger.record.updated,
    { $id: Now.ID["incident_updated_trigger"] },
    {
      table: "incident",
      condition: "priority=1",
      run_flow_in: "background",
      run_on_extended: "false",
      run_when_setting: "both",
      run_when_user_setting: "any",
      trigger_strategy: "unique_changes"
    }
  ),

  _params => {
    // ✅ Access changed fields directly - use data pills directly
    wfa.flowLogic.forEach(
      wfa.dataPill(_params.trigger.changed_fields, "array.object"),
      { $id: Now.ID["loop_changes"] },
      change => {
        wfa.action(
          action.core.log,
          { $id: Now.ID["log_change"] },
          {
            log_level: "info",
            log_message: `Field ${wfa.dataPill(change.field_name, "string")} changed from ${wfa.dataPill(change.previous_value, "string")} to ${wfa.dataPill(change.current_value, "string")}`
          }
        );
      }
    );
  }
);
```

---

## trigger.record.createdOrUpdated

Activates when a record is either created or updated in the specified table.

### Configuration Parameters

| Parameter             | Type                                              | Default | Mandatory | Description                                                  |
| --------------------- | ------------------------------------------------- | ------- | --------- | ------------------------------------------------------------ |
| table                 | string                                            | -       | Yes       | Table to monitor for created or updated records              |
| condition             | string                                            | -       | No        | Encoded query to filter which records trigger the flow       |
| run_flow_in           | 'any' \| 'background' \| 'foreground'             | 'any'   | No        | Execution context for the flow                               |
| run_on_extended       | 'true' \| 'false'                                 | 'false' | No        | Whether to run on child tables                               |
| run_when_setting      | 'both' \| 'interactive' \| 'non_interactive'      | 'both'  | No        | Session type filter                                          |
| run_when_user_setting | 'any' \| 'one_of' \| 'not_one_of'                 | 'any'   | No        | User-based filter mode                                       |
| run_when_user_list    | string[]                                          | []      | No        | List of user sys_ids for filtering (requires run_query tool) |
| trigger_strategy      | 'once' \| 'unique_changes' \| 'every' \| 'always' | 'once'  | No        | Controls when trigger fires for updates                      |

**trigger_strategy Values:**

- `'once'` - Fires only the first time condition matches
- `'unique_changes'` - Fires for each unique change to monitored fields
- `'every'` - Fires on every update regardless of field changes
- `'always'` - Fires only if flow is not currently running

**Tools needed:** Use `get_table_schema` to get table field definitions, `run_query` to get user sys_ids for run_when_user_list.

### Output Fields

| Field               | Type            | Description                                               |
| ------------------- | --------------- | --------------------------------------------------------- |
| current             | reference       | The created or updated record (current state)             |
| changed_fields      | array           | Array of changed field details (only present for updates) |
| table_name          | string          | Table where record was created or updated                 |
| run_start_time      | glide_date_time | Flow execution start time                                 |
| run_start_date_time | glide_date_time | Flow execution start date/time                            |

**Note:** The `changed_fields` output is only populated when the trigger fires for an update. On record creation, `changed_fields` will be empty or undefined.

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.current, "reference");

  // Check if changed_fields exists (update) or not (create)
  wfa.flowLogic.if({ $id: Now.ID["check_update"], condition: "..." }, () => {
    wfa.dataPill(_params.trigger.changed_fields, "array.object");
  });
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["audit_incident_changes"],
    name: "Audit All Incident Changes"
  },

  wfa.trigger(
    trigger.record.createdOrUpdated,
    { $id: Now.ID["incident_created_or_updated_trigger"] },
    {
      table: "incident",
      condition: "priority=1^ORpriority=2",
      run_flow_in: "background",
      run_on_extended: "false",
      run_when_setting: "both",
      run_when_user_setting: "any",
      trigger_strategy: "once"
    }
  ),

  _params => {
    // ✅ Check if this is a create or update - use data pills directly
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_if_update"],
        condition: `${wfa.dataPill(_params.trigger.changed_fields, "string")}ISNOTEMPTY`
      },
      () => {
        // This is an update
        wfa.action(
          action.core.log,
          { $id: Now.ID["log_update"] },
          {
            log_level: "info",
            log_message: `Incident ${wfa.dataPill(_params.trigger.current.number, "string")} updated`
          }
        );
      }
    );

    wfa.flowLogic.else({ $id: Now.ID["handle_create"] }, () => {
      // This is a create
      wfa.action(
        action.core.log,
        { $id: Now.ID["log_create"] },
        {
          log_level: "info",
          log_message: `Incident ${wfa.dataPill(_params.trigger.current.number, "string")} created`
        }
      );
    });
  }
);
```

---

## Notes

- Use `get_table_schema` tool to get table field definitions and data types before using trigger conditions
- Use `run_query` tool to get sys_ids for users when using `run_when_user_list` parameter
- Condition syntax follows ServiceNow encoded query format (e.g., `priority=1^active=true`)
- The `trigger_strategy` parameter is only available for `updated` and `createdOrUpdated` triggers
- For `createdOrUpdated` trigger, check if `changed_fields` exists to determine if operation was create or update
- Background execution (`run_flow_in: 'background'`) is recommended for most automation workflows
