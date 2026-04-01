# WFA_FLOW_ACTIONS_TABLE

The Table Actions API reference provides complete technical specifications for all table-related workflow automation actions. This document contains action signatures, input parameters, output fields, and syntax examples for creating, reading, updating, and deleting records in ServiceNow tables. For conceptual guidance, when-to-use recommendations, and best practices, refer to skill references.

## Action Usage Pattern

All table actions follow the same usage pattern:

```typescript
const result = wfa.action(
  action.core.<actionName>,
  { $id: Now.ID['action_id'], annotation?: 'Description' },
  {
    // action-specific input parameters
  }
);

// Access outputs: result.field_name
```

---

## action.core.createRecord

Creates a new record on any ServiceNow table with dynamically configured fields. Server-side validation rules are enforced (data policy, business rules, dictionary-defined mandatory fields), but UI policy does not apply.

### Input Parameters

| Parameter  | Type          | Default | Mandatory | Description                     |
| ---------- | ------------- | ------- | --------- | ------------------------------- |
| table_name | string        | -       | Yes       | Target table name               |
| values     | TemplateValue | -       | Yes       | Field values for the new record |

**Tools needed:** Use `get_table_schema` to get table field definitions before creating records.

### Output Fields

| Field      | Type      | Description                    |
| ---------- | --------- | ------------------------------ |
| record     | reference | Created record sys_id          |
| table_name | string    | Table where record was created |

**Access Pattern:**

```typescript
const newRecord = wfa.action(action.core.createRecord, { $id }, { ... });

// ✅ CORRECT - Use data pills directly in next action
wfa.action(action.core.updateRecord, { $id: Now.ID['update'] }, {
  record: wfa.dataPill(newRecord.record, 'reference'),  // Direct usage
  values: TemplateValue({ state: 2 })
});
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["create_incident_flow"],
    name: "Create Incident from Email"
  },

  wfa.trigger(
    trigger.application.inboundEmail,
    { $id: Now.ID["email_trigger"] },
    {
      email_conditions: "subjectLIKEurgent"
    }
  ),

  _params => {
    const incident = wfa.action(
      action.core.createRecord,
      {
        $id: Now.ID["create_incident"],
        annotation: "Create incident from email"
      },
      {
        table_name: "incident",
        values: TemplateValue({
          short_description: wfa.dataPill(_params.trigger.subject, "string"),
          description: wfa.dataPill(_params.trigger.body_text, "string"),
          priority: "1",
          urgency: "1",
          caller_id: wfa.dataPill(_params.trigger.target_record, "reference")
        })
      }
    );

    // Use the created record
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_created"] },
      {
        log_level: "info",
        log_message: `Created incident: ${wfa.dataPill(incident.record, "reference")}`
      }
    );
  }
);
```

---

## action.core.updateRecord

Updates an existing record on a ServiceNow table with dynamically configured fields. Server-side validation rules are enforced (data policy, business rules, dictionary-defined mandatory fields), but UI policy does not apply.

### Input Parameters

| Parameter  | Type          | Default | Mandatory | Description                      |
| ---------- | ------------- | ------- | --------- | -------------------------------- |
| table_name | string        | -       | Yes       | Target table name                |
| record     | reference     | -       | Yes       | Record sys_id to update          |
| values     | TemplateValue | -       | Yes       | Fields to update with new values |

**Tools needed:** Use `get_table_schema` to get table field definitions before updating records.

### Output Fields

| Field      | Type      | Description                    |
| ---------- | --------- | ------------------------------ |
| record     | reference | Updated record sys_id          |
| table_name | string    | Table where record was updated |

**Access Pattern:**

```typescript
const updated = wfa.action(action.core.updateRecord, { $id }, { ... });

// ✅ CORRECT - Use data pill directly in next action
wfa.action(action.core.sendNotification, { $id: Now.ID['notify'] }, {
  record: wfa.dataPill(updated.record, 'reference'),  // Direct usage
  users: ['admin'],
  message: 'Record updated successfully'
});
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["assign_incident_flow"],
    name: "Auto-Assign High Priority Incidents"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_created"] },
    {
      table: "incident",
      condition: "priority=1^assigned_toISEMPTY",
      run_flow_in: "background"
    }
  ),

  _params => {
    // Find assignment group
    const group = wfa.action(
      action.core.lookUpRecord,
      { $id: Now.ID["find_group"] },
      {
        table: "sys_user_group",
        conditions: "name=IT Support^active=true"
      }
    );

    // Update the incident
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["assign_incident"], annotation: "Assign to IT Support" },
      {
        table_name: "incident",
        record: wfa.dataPill(_params.trigger.target_record, "reference"),
        values: TemplateValue({
          assignment_group: wfa.dataPill(group.Record, "reference"),
          state: 2,
          work_notes: "Auto-assigned to IT Support team"
        })
      }
    );
  }
);
```

---

## action.core.deleteRecord

Deletes a specific record from a table. This is a permanent operation that cannot be undone.

### Input Parameters

| Parameter | Type      | Default | Mandatory | Description             |
| --------- | --------- | ------- | --------- | ----------------------- |
| record    | reference | -       | Yes       | Record sys_id to delete |

**Important:** Deletion requires delete ACLs on the target table and is permanent.

### Output Fields

None - Action completes silently if successful.

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["cleanup_flow"],
    name: "Delete Temporary Records"
  },

  wfa.trigger(
    trigger.scheduled.daily,
    { $id: Now.ID["daily_cleanup"] },
    {
      time: Time({ hours: 2, minutes: 0, seconds: 0 }, "UTC")
    }
  ),

  _params => {
    // Find expired temporary records
    const tempRecords = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["find_expired"] },
      {
        table: "u_temp_table",
        conditions: "sys_created_on<javascript:gs.daysAgoStart(7)",
        max_results: 100
      }
    );

    // Delete each expired record
    wfa.flowLogic.forEach(
      wfa.dataPill(tempRecords.Records, "array.object"),
      { $id: Now.ID["delete_loop"], annotation: "Delete each expired record" },
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

## action.core.lookUpRecord

Looks up a single record on any ServiceNow table with configurable conditions. If multiple records match, only the first record is returned based on sort order.

### Input Parameters

| Parameter                            | Type    | Default            | Mandatory | Description                                 |
| ------------------------------------ | ------- | ------------------ | --------- | ------------------------------------------- |
| table                                | string  | -                  | Yes       | Table to search                             |
| conditions                           | string  | -                  | Yes       | Encoded query conditions                    |
| sort_column                          | string  | -                  | No        | Field name to sort by                       |
| sort_type                            | choice  | 'sort_asc'         | No        | 'sort_asc' (a to z) or 'sort_desc' (z to a) |
| if_multiple_records_are_found_action | choice  | 'use_first_record' | No        | 'use_first_record' or 'error'               |
| \_\_snc_dont_fail_on_error           | boolean | -                  | No        | Don't fail flow on error                    |

**Tools needed:** Use `get_table_schema` to understand table fields for building conditions.

### Output Fields

| Field         | Type      | Description                                |
| ------------- | --------- | ------------------------------------------ |
| Record        | reference | Found record sys_id (uppercase field name) |
| status        | choice    | '0' (success) or '1' (error)               |
| Table         | string    | Table name                                 |
| error_message | string    | Error message if status is '1'             |

**Important:** The `Record` output field name is **uppercase** (unlike other actions which use lowercase `record`).

**Access Pattern:**

```typescript
const found = wfa.action(action.core.lookUpRecord, { $id }, { ... });

// ✅ CORRECT - Check if record was found and use directly
wfa.flowLogic.if(
  { $id: Now.ID['check_found'], condition: `${wfa.dataPill(found.status, 'string')}=0` },
  () => {
    // Record found - use found.Record (uppercase) directly in next action
    wfa.action(action.core.updateRecord, { $id: Now.ID['update'] }, {
      record: wfa.dataPill(found.Record, 'reference'),  // Direct usage, uppercase Record
      values: TemplateValue({ state: 2 })
    });
  }
);
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["lookup_user_flow"],
    name: "Lookup User and Assign"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_trigger"] },
    {
      table: "incident",
      condition: "priority=1",
      run_flow_in: "background"
    }
  ),

  _params => {
    // Find admin user
    const adminUser = wfa.action(
      action.core.lookUpRecord,
      { $id: Now.ID["find_admin"], annotation: "Find admin user" },
      {
        table: "sys_user",
        conditions: "user_name=admin^active=true",
        sort_column: "name",
        sort_type: "sort_asc"
      }
    );

    // Check if user was found
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_admin_found"],
        condition: `${wfa.dataPill(adminUser.status, "string")}=0`
      },
      () => {
        // Assign to admin user
        wfa.action(
          action.core.updateRecord,
          { $id: Now.ID["assign_to_admin"] },
          {
            table_name: "incident",
            record: wfa.dataPill(_params.trigger.target_record, "reference"),
            values: TemplateValue({
              assigned_to: wfa.dataPill(adminUser.Record, "reference")
            })
          }
        );
      }
    );

    wfa.flowLogic.else({ $id: Now.ID["handle_not_found"] }, () => {
      wfa.action(
        action.core.log,
        { $id: Now.ID["log_error"] },
        {
          log_level: "error",
          log_message: `Admin user not found: ${wfa.dataPill(adminUser.error_message, "string")}`
        }
      );
    });
  }
);
```

---

## action.core.lookUpRecords

Looks up multiple records on any ServiceNow table with configurable conditions. Returns an array of records and the count of records found. Maximum 10,000 records can be returned (configurable by property).

### Input Parameters

| Parameter   | Type    | Default    | Mandatory | Description                                 |
| ----------- | ------- | ---------- | --------- | ------------------------------------------- |
| table       | string  | -          | Yes       | Table to search                             |
| conditions  | string  | -          | Yes       | Encoded query conditions                    |
| max_results | integer | 1000       | No        | Maximum number of records to return         |
| sort_column | string  | -          | No        | Field name to sort by                       |
| sort_type   | choice  | 'sort_asc' | No        | 'sort_asc' (a to z) or 'sort_desc' (z to a) |

**Tools needed:** Use `get_table_schema` to understand table fields for building conditions.

### Output Fields

| Field   | Type    | Description                                          |
| ------- | ------- | ---------------------------------------------------- |
| Records | array   | Array of found record sys_ids (uppercase field name) |
| Count   | integer | Number of records found (uppercase field name)       |
| Table   | string  | Table name                                           |

**Important:** Output field names are **uppercase** (`Records`, `Count`, `Table`) unlike other actions.

**Access Pattern:**

```typescript
const results = wfa.action(action.core.lookUpRecords, { $id }, { ... });

// Check if records were found
wfa.flowLogic.if(
  { $id: Now.ID['check_count'], condition: `${wfa.dataPill(results.Count, 'integer')}>0` },
  () => {
    // Process records with forEach
    wfa.flowLogic.forEach(
      wfa.dataPill(results.Records, 'array.object'),
      { $id: Now.ID['process_loop'] },
      (record) => {
        // Process each record
      }
    );
  }
);
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["bulk_assign_flow"],
    name: "Bulk Assign Unassigned Incidents"
  },

  wfa.trigger(
    trigger.scheduled.daily,
    { $id: Now.ID["daily_trigger"] },
    {
      time: Time({ hours: 8, minutes: 0, seconds: 0 }, "UTC")
    }
  ),

  _params => {
    // Find default assignment group
    const group = wfa.action(
      action.core.lookUpRecord,
      { $id: Now.ID["find_group"] },
      {
        table: "sys_user_group",
        conditions: "name=Default Support Group"
      }
    );

    // Find all unassigned P1/P2 incidents
    const incidents = wfa.action(
      action.core.lookUpRecords,
      {
        $id: Now.ID["find_incidents"],
        annotation: "Find unassigned incidents"
      },
      {
        table: "incident",
        conditions: "active=true^priority<=2^assignment_groupISEMPTY",
        max_results: 50,
        sort_column: "sys_created_on",
        sort_type: "sort_asc"
      }
    );

    wfa.action(
      action.core.log,
      { $id: Now.ID["log_count"] },
      {
        log_level: "info",
        log_message: `Found ${wfa.dataPill(incidents.Count, "integer")} unassigned incidents`
      }
    );

    // Check if any incidents found
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_incidents"],
        condition: `${wfa.dataPill(incidents.Count, "integer")}>0`
      },
      () => {
        // Process each incident
        wfa.flowLogic.forEach(
          wfa.dataPill(incidents.Records, "array.object"),
          { $id: Now.ID["assign_loop"], annotation: "Assign each incident" },
          incident => {
            wfa.action(
              action.core.updateRecord,
              { $id: Now.ID["assign_incident"] },
              {
                table_name: "incident",
                record: wfa.dataPill(incident, "reference"),
                values: TemplateValue({
                  assignment_group: wfa.dataPill(group.Record, "reference"),
                  work_notes: "Auto-assigned by workflow"
                })
              }
            );
          }
        );
      }
    );
  }
);
```

---

## action.core.updateMultipleRecords

Updates multiple records on a ServiceNow table based on conditions. All matching records receive the same field values. Server-side validation rules are enforced (data policy, business rules), but UI policy does not apply.

### Input Parameters

| Parameter                  | Type          | Default    | Mandatory | Description                                 |
| -------------------------- | ------------- | ---------- | --------- | ------------------------------------------- |
| table_name                 | string        | -          | Yes       | Target table name                           |
| conditions                 | string        | -          | Yes       | Encoded query to filter records             |
| field_values               | TemplateValue | -          | Yes       | Fields to update with new values            |
| sort_column                | string        | -          | No        | Field name to sort by                       |
| sort_type                  | choice        | 'sort_asc' | No        | 'sort_asc' (a to z) or 'sort_desc' (z to a) |
| \_\_snc_dont_fail_on_error | boolean       | -          | No        | Don't fail flow on error                    |

**Tools needed:** Use `get_table_schema` to get table field definitions before updating records.

**Warning:** Business rules fire for each record updated, which can impact performance for large record sets.

### Output Fields

| Field   | Type    | Description                  |
| ------- | ------- | ---------------------------- |
| count   | integer | Number of records updated    |
| status  | choice  | '0' (success) or '1' (error) |
| message | string  | Status or error message      |

**Access Pattern:**

```typescript
const result = wfa.action(action.core.updateMultipleRecords, { $id }, { ... });

wfa.action(action.core.log, { $id }, {
  log_level: 'info',
  log_message: `Updated ${wfa.dataPill(result.count, 'integer')} records`
});
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["bulk_close_flow"],
    name: "Bulk Close Resolved Incidents"
  },

  wfa.trigger(
    trigger.scheduled.weekly,
    { $id: Now.ID["weekly_trigger"] },
    {
      day_of_week: 2, // Monday
      time: Time({ hours: 1, minutes: 0, seconds: 0 }, "UTC")
    }
  ),

  _params => {
    // Bulk close incidents resolved for 30+ days
    const result = wfa.action(
      action.core.updateMultipleRecords,
      { $id: Now.ID["bulk_close"], annotation: "Close old resolved incidents" },
      {
        table_name: "incident",
        conditions:
          "state=6^active=true^sys_updated_on<javascript:gs.daysAgoStart(30)",
        field_values: TemplateValue({
          state: 7,
          active: false,
          close_code: "Closed/Resolved by Caller",
          close_notes: "Auto-closed after 30 days in resolved state"
        }),
        sort_column: "sys_updated_on",
        sort_type: "sort_asc"
      }
    );

    // Log the result
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_result"] },
      {
        log_level: "info",
        log_message: `Closed ${wfa.dataPill(result.count, "integer")} incidents. Status: ${wfa.dataPill(result.status, "string")}`
      }
    );
  }
);
```

---

## action.core.createOrUpdateRecord

Creates or updates a record in a ServiceNow table by determining if it already exists. Identifies existing records using unique fields defined in the table dictionary. Adds records that don't exist and updates existing ones. Server-side validation rules are enforced.

### Input Parameters

| Parameter  | Type          | Default | Mandatory | Description                                           |
| ---------- | ------------- | ------- | --------- | ----------------------------------------------------- |
| table_name | string        | -       | Yes       | Target table name                                     |
| fields     | TemplateValue | -       | Yes       | Fields including unique identifiers and values to set |

**Tools needed:** Use `get_table_schema` to identify unique fields in the target table.

**Important:** The table must have unique fields defined for matching to work. Include the unique identifier field(s) in the fields object.

### Output Fields

| Field         | Type      | Description                        |
| ------------- | --------- | ---------------------------------- |
| record        | reference | Record sys_id (created or updated) |
| status        | choice    | 'created', 'updated', or 'error'   |
| table_name    | string    | Table name                         |
| error_message | string    | Error message if status is 'error' |

**Access Pattern:**

```typescript
const result = wfa.action(action.core.createOrUpdateRecord, { $id }, { ... });

wfa.flowLogic.if(
  { $id: Now.ID['check_created'], condition: `${wfa.dataPill(result.status, 'string')}=created` },
  () => {
    // Handle new record
  }
);

wfa.flowLogic.elseIf(
  { $id: Now.ID['check_updated'], condition: `${wfa.dataPill(result.status, 'string')}=updated` },
  () => {
    // Handle updated record
  }
);
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["sync_user_flow"],
    name: "Sync User from External System"
  },

  wfa.trigger(
    trigger.application.inboundEmail,
    { $id: Now.ID["email_trigger"] },
    {
      email_conditions: "subjectLIKEUser Sync"
    }
  ),

  _params => {
    // Upsert user by email (unique field)
    const user = wfa.action(
      action.core.createOrUpdateRecord,
      {
        $id: Now.ID["upsert_user"],
        annotation: "Create or update user by email"
      },
      {
        table_name: "sys_user",
        fields: TemplateValue({
          email: wfa.dataPill(_params.trigger.from_address, "string"),
          first_name: "John",
          last_name: "Doe",
          phone: "+1-555-0100",
          active: true
        })
      }
    );

    // Log based on operation performed
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_created"],
        condition: `${wfa.dataPill(user.status, "string")}=created`
      },
      () => {
        wfa.action(
          action.core.log,
          { $id: Now.ID["log_created"] },
          {
            log_level: "info",
            log_message: `Created new user: ${wfa.dataPill(user.record, "reference")}`
          }
        );
      }
    );

    wfa.flowLogic.elseIf(
      {
        $id: Now.ID["check_updated"],
        condition: `${wfa.dataPill(user.status, "string")}=updated`
      },
      () => {
        wfa.action(
          action.core.log,
          { $id: Now.ID["log_updated"] },
          {
            log_level: "info",
            log_message: `Updated existing user: ${wfa.dataPill(user.record, "reference")}`
          }
        );
      }
    );

    wfa.flowLogic.else({ $id: Now.ID["handle_error"] }, () => {
      wfa.action(
        action.core.log,
        { $id: Now.ID["log_error"] },
        {
          log_level: "error",
          log_message: `Error: ${wfa.dataPill(user.error_message, "string")}`
        }
      );
    });
  }
);
```

---

## Common Unique Fields by Table

When using `createOrUpdateRecord`, match is based on unique fields defined in the table dictionary:

| Table            | Common Unique Fields     |
| ---------------- | ------------------------ |
| sys_user         | email, user_name         |
| sys_user_group   | name                     |
| cmdb_ci          | serial_number, asset_tag |
| cmdb_ci_computer | serial_number, asset_tag |
| core_company     | name                     |
| sc_cat_item      | name                     |
| sys_properties   | name                     |
| u_custom_table   | (check table dictionary) |

**Tool Usage:** Use `get_table_schema({ table: 'table_name' })` to identify unique fields.

---
