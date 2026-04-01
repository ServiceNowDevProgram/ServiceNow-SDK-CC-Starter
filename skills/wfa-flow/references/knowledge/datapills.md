# DATAPILLS

The DataPills API reference provides complete technical specifications for the `wfa.dataPill()` function and `FlowDataType` enum. This document contains API signatures, parameter definitions, types, and syntax examples for referencing and passing data between flow components.

---

## ⚠️ CRITICAL: Data Pills Usage Pattern

**WFA flows are declarative, not imperative JavaScript. Data pills MUST be used directly in action parameters and flow conditions.**

### ❌ WRONG - Do NOT assign data pills to const/let/var

```typescript
// ❌ WRONG - Assigning data pills to variables
const recordId = wfa.dataPill(result.record, "reference");
const status = wfa.dataPill(result.status, "string");
const description = wfa.dataPill(
  _params.trigger.current.short_description,
  "string"
);

// Then using the variables - INCORRECT PATTERN
wfa.action(
  action.core.updateRecord,
  { $id },
  {
    record: recordId, // WRONG!
    values: TemplateValue({
      state: status // WRONG!
    })
  }
);
```

**Why this doesn't work:** WFA flows are declarative. Variable assignments are imperative JavaScript code that doesn't execute in the flow runtime.

### ✅ CORRECT - Use data pills directly in parameters

```typescript
// ✅ CORRECT - Use wfa.dataPill() directly in action parameters
const result = wfa.action(action.core.createRecord, { $id }, { ... });

wfa.action(action.core.updateRecord, { $id: Now.ID['update'] }, {
  record: wfa.dataPill(result.record, 'reference'),  // Direct usage
  values: TemplateValue({
    short_description: wfa.dataPill(_params.trigger.current.short_description, 'string'),
    priority: wfa.dataPill(_params.trigger.current.priority, 'integer'),
    state: 2
  })
});

// ✅ CORRECT - Use in flow conditions
wfa.flowLogic.if(
  {
    $id: Now.ID['check'],
    condition: `${wfa.dataPill(result.status, 'string')}=0`
  },
  () => {
    // Use data pills directly in actions
    wfa.action(action.core.log, { $id: Now.ID['log'] }, {
      log_level: 'info',
      log_message: `Record ${wfa.dataPill(result.record, 'reference')} created`
    });
  }
);
```

**Key Principle:** Data pills are declarative expressions that get resolved at flow runtime. Always use `wfa.dataPill()` directly where you need the value - in action parameters, TemplateValue fields, or template literals within conditions.

---

## wfa.dataPill()

Wrapper function for datapill expressions that associates data with its type for Flow Designer XML serialization.

### Signature

```typescript
wfa.dataPill<T>(_expression: T, _type: FlowDataType): DataPillReturn<T>
```

### Parameters

**1. \_expression** - The datapill expression to wrap (e.g., `_params.trigger.current.active`, `actionResult.fieldName`)

**2. \_type** - The FlowDataType specifying the data type for serialization

### Return Value

Returns the expression unchanged (passthrough at runtime). Type information is extracted at compile time by build plugins.

### Usage Contexts

- **Trigger data**: `_params.trigger.current.*` - Record fields from trigger
- **Action outputs**: `actionResult.fieldName` - Action output fields
- **Subflow outputs**: `subflowResult.fieldName` - Subflow output fields
- **Table name**: `_params.trigger.table_name` - Trigger table name

**Example:**

```typescript
// Trigger record fields
wfa.dataPill(_params.trigger.current.short_description, "string");
wfa.dataPill(_params.trigger.current.priority, "integer");
wfa.dataPill(_params.trigger.current.active, "boolean");
wfa.dataPill(_params.trigger.current, "reference");

// Reference field dot-walking
wfa.dataPill(_params.trigger.current.assigned_to.email, "string");
wfa.dataPill(_params.trigger.current.assigned_to.manager.name, "string");

// Action outputs
wfa.dataPill(createRecord.Record, "reference");
wfa.dataPill(lookUpRecords.Records, "records")
// In template literals
`Incident ${wfa.dataPill(_params.trigger.current.number, "integer")} created`;
```

---

## FlowDataType Enum

Type union for all supported ServiceNow Flow data types.

### Basic Types

- `'string'` - Standard text field
- `'string_full_utf8'` - Full UTF-8 text (email subjects, body content)
- `'string_boolean'` - String representation of boolean
- `'integer'` - Whole number
- `'int'` - Alias for integer
- `'long'` - Long integer
- `'longint'` - Alias for long
- `'boolean'` - True/false value
- `'char'` - Character field

**Example:**

```typescript
wfa.dataPill(_params.trigger.current.short_description, "string");
wfa.dataPill(_params.trigger.subject, "string_full_utf8");
wfa.dataPill(_params.trigger.current.priority, "integer");
wfa.dataPill(_params.trigger.current.active, "boolean");
```

---

### Numeric Types

- `'decimal'` - Decimal number
- `'float'` - Floating point number
- `'currency'` - Currency value
- `'currency2'` - Secondary currency
- `'price'` - Price field
- `'percent_complete'` - Percentage value

**Example:**

```typescript
wfa.dataPill(_params.trigger.current.cost, "currency");
wfa.dataPill(_params.trigger.current.completion, "percent_complete");
wfa.dataPill(_params.trigger.current.discount, "decimal");
```

---

### Date/Time Types

- `'datetime'` - Standard datetime
- `'date'` - Date only
- `'time'` - Time only
- `'glide_date_time'` - ServiceNow GlideDateTime
- `'glide_date'` - ServiceNow GlideDate
- `'glide_time'` - ServiceNow GlideTime
- `'glide_utc_time'` - UTC time
- `'glide_precise_time'` - Precise time
- `'glide_duration'` - Duration value
- `'due_date'` - Due date field
- `'calendar_date_time'` - Calendar datetime
- `'schedule_date_time'` - Schedule datetime
- `'integer_date'` - Integer-based date
- `'integer_time'` - Integer-based time
- `'insert_timestamp'` - Insert timestamp

**Example:**

```typescript
wfa.dataPill(_params.trigger.current.sys_created_on, "datetime");
wfa.dataPill(_params.trigger.current.due_date, "glide_date_time");
wfa.dataPill(_params.trigger.current.work_start, "date");
wfa.dataPill(_params.trigger.current.duration, "glide_duration");
```

---

### Reference Types

- `'reference'` - Record reference (supports dot-walking)
- `'reference_name'` - Display value of reference
- `'document_id'` - Document/record sys_id
- `'table_name'` - Table name
- `'short_table_name'` - Short table name
- `'field_name'` - Field name
- `'short_field_name'` - Short field name
- `'field_list'` - Field list
- `'domain_id'` - Domain identifier
- `'domain_path'` - Domain path
- `'source_id'` - Source identifier
- `'source_name'` - Source name
- `'source_table'` - Source table

**Example:**

```typescript
wfa.dataPill(_params.trigger.current, "reference");
wfa.dataPill(_params.trigger.current.assigned_to.sys_id, "reference");
wfa.dataPill(_params.trigger.table_name, "table_name");
wfa.dataPill(_params.trigger.current.caller_id, "reference");
```

---

### Array Types

- `'array.string'` - Array of strings
- `'array.integer'` - Array of integers
- `'array.boolean'` - Array of booleans
- `'array.datetime'` - Array of datetime values
- `'array.object'` - Array of objects
- `'data_array'` - Generic data array
- `'records'` - Array of record references (lookUpRecords output)
- `'glide_list'` - ServiceNow list field

**Example:**

```typescript
const records = wfa.action(
  action.core.lookUpRecords,
  { $id },
  { table: "cmdb_ci" }
);
wfa.flowLogic.forEach(
  wfa.dataPill(records.Records, "records"),
  { $id: Now.ID["loop_cis"] },
  ci => {
    wfa.dataPill(ci.name, "string");
  }
);
```

---

### Complex Types

- `'object'` - Complex object
- `'data_object'` - Data structure object
- `'data_structure'` - Structured data
- `'collection'` - Collection type
- `'json'` - JSON content
- `'xml'` - XML content
- `'template_value'` - TemplateValue object
- `'snapshot_template_value'` - Snapshot template value
- `'variable_template_value'` - Variable template value

**Example:**

```typescript
wfa.dataPill(_params.trigger.current.u_config, "json");
wfa.dataPill(_params.trigger.current.variables, "object");
```

---

### HTML/Script Types

- `'html'` - HTML content
- `'html_script'` - HTML script
- `'html_template'` - HTML template
- `'translated_html'` - Translated HTML
- `'script'` - Script content
- `'script_client'` - Client script
- `'script_server'` - Server script
- `'script_plain'` - Plain script
- `'email_script'` - Email script
- `'css'` - CSS content

**Example:**

```typescript
wfa.action(
  action.core.sendEmail,
  { $id },
  {
    ah_to: wfa.dataPill(_params.trigger.current.caller_id.email, "string"),
    ah_subject: "Update",
    ah_body: wfa.dataPill(_params.trigger.body_text, "html"),
    record: wfa.dataPill(_params.trigger.current, "reference"),
    table_name: "incident"
  }
);
```

---

### Communication Types

- `'email'` - Email address
- `'phone_number'` - Phone number
- `'phone_number_e164'` - E.164 phone number
- `'ph_number'` - Phone number alias
- `'ip_addr'` - IP address
- `'ip_address'` - IP address alias
- `'url'` - URL/web address

**Example:**

```typescript
wfa.dataPill(_params.trigger.current.caller_id.email, "email");
wfa.dataPill(_params.trigger.current.caller_id.phone, "phone_number");
wfa.dataPill(_params.trigger.current.u_server_ip, "ip_address");
```

---

### ServiceNow-Specific Types

- `'journal'` - Journal field
- `'journal_input'` - Journal input field
- `'journal_list'` - Journal list
- `'sys_class_name'` - System class name
- `'sys_class_path'` - System class path
- `'workflow'` - Workflow reference
- `'workflow_conditions'` - Workflow conditions
- `'approval_rules'` - Approval rules configuration
- `'conditions'` - Query conditions
- `'condition_string'` - Condition string
- `'expression'` - Expression field
- `'formula'` - Formula field

**Example:**

```typescript
wfa.dataPill(_params.trigger.current.work_notes, "journal_input");
wfa.dataPill(_params.trigger.current.sys_class_name, "sys_class_name");
```

---

### Choice and UI Types

- `'choice'` - Choice list value
- `'radio'` - Radio button value
- `'color'` - Color value
- `'color_display'` - Color display
- `'bootstrap_color'` - Bootstrap color
- `'icon'` - Icon field
- `'nds_icon'` - NDS icon
- `'glyphicon'` - Glyphicon
- `'image'` - Image field
- `'user_image'` - User image
- `'audio'` - Audio field
- `'video'` - Video field
- `'catalog_preview'` - Catalog preview

**Example:**

```typescript
wfa.dataPill(_params.trigger.current.state, "choice");
wfa.dataPill(_params.trigger.current.priority, "choice");
wfa.dataPill(_params.trigger.current.impact, "choice");
```

---

### Text Content Types

- `'journal'` - Journal field
- `'journal_input'` - Journal input
- `'journal_list'` - Journal list
- `'wiki_text'` - Wiki text
- `'wide_text'` - Wide text
- `'multi_line_text'` - Multi-line text
- `'multi_small'` - Multi small text
- `'multi_two_lines'` - Two-line text
- `'translated'` - Translated field
- `'translated_field'` - Translated field variant
- `'translated_text'` - Translated text

**Example:**

```typescript
wfa.dataPill(_params.trigger.current.description, "multi_line_text");
wfa.dataPill(_params.trigger.current.work_notes, "journal_input");
wfa.dataPill(_params.trigger.current.comments, "journal");
```

---

### Other ServiceNow Types

- `'password'` - Password field
- `'password2'` - Secondary password
- `'user_roles'` - User roles list
- `'user_input'` - User input
- `'variables'` - Variables field
- `'variable_conditions'` - Variable conditions
- `'properties'` - Properties field
- `'name_values'` - Name-value pairs
- `'name_value_pairs'` - Name-value pairs variant
- `'simple_name_values'` - Simple name-value pairs
- `'external_names'` - External names
- `'slush_bucket'` - Slush bucket
- `'slushbucket'` - Slush bucket alias
- `'guid'` - GUID field
- `'auto_increment'` - Auto-increment field
- `'auto_number'` - Auto-number field
- `'counter'` - Counter field
- `'order_index'` - Order index
- `'version'` - Version field
- `'compressed'` - Compressed field
- `'composite_field'` - Composite field
- `'composite_name'` - Composite name
- `'mask_code'` - Mask code
- `'tree_code'` - Tree code
- `'tree_path'` - Tree path
- `'record_hierarchy_path'` - Record hierarchy path
- `'documentation_field'` - Documentation field
- `'file_attachment'` - File attachment
- `'breakdown_element'` - Breakdown element
- `'days_of_week'` - Days of week
- `'day_of_week'` - Day of week
- `'week_of_month'` - Week of month
- `'month_of_year'` - Month of year
- `'repeat_count'` - Repeat count
- `'repeat_type'` - Repeat type
- `'timer'` - Timer field
- `'geo_point'` - Geographic point
- `'mid_config'` - MID config
- `'wms_job'` - WMS job
- `'dynamic_attribute_store'` - Dynamic attribute store
- `'replication_payload'` - Replication payload
- `'reminder_field_name'` - Reminder field name
- `'schedule_interval_count'` - Schedule interval count
- `'metric_absolute'` - Metric absolute
- `'metric_counter'` - Metric counter
- `'metric_derive'` - Metric derive
- `'metric_gauge'` - Metric gauge
- `'nl_task_int1'` - NL task int
- `'sysevent_name'` - System event name
- `'sysrule_field_name'` - System rule field name
- `'glide_var'` - Glide variable
- `'glide_action_list'` - Glide action list
- `'glide_variables'` - Glide variables
- `'index_name'` - Index name
- `'internal_type'` - Internal type
- `'language'` - Language field
- `'related_tags'` - Related tags
- `'graphql_schema'` - GraphQL schema

---

## Complete Examples

### Trigger DataPills

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["incident_escalation_flow"],
    name: "Incident Escalation with DataPills"
  },
  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_created"] },
    { table: "incident", condition: "priority=1" }
  ),
  _params => {
    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["send_email"] },
      {
        ah_to: wfa.dataPill(
          _params.trigger.current.assigned_to.manager.email,
          "string"
        ),
        ah_subject: `Critical: ${wfa.dataPill(_params.trigger.current.number, "integer")}`,
        ah_body: `Incident ${wfa.dataPill(_params.trigger.current.number, "integer")} requires attention.
          Description: ${wfa.dataPill(_params.trigger.current.short_description, "string")}
          Caller: ${wfa.dataPill(_params.trigger.current.caller_id.name, "string")}`,
        record: wfa.dataPill(_params.trigger.current, "reference"),
        table_name: "incident"
      }
    );
  }
);
```

### Email Content DataPills (string_full_utf8)

```typescript
Flow(
  {
    $id: Now.ID["inbound_email_processor"],
    name: "Process Inbound Email"
  },
  wfa.trigger(
    trigger.application.inboundEmail,
    { $id: Now.ID["email_trigger"] },
    { email_conditions: "^EQ", target_table: "incident" }
  ),
  _params => {
    wfa.action(
      action.core.createRecord,
      { $id: Now.ID["create_from_email"] },
      {
        table_name: "incident",
        values: TemplateValue({
          short_description: wfa.dataPill(
            _params.trigger.subject,
            "string_full_utf8"
          ),
          description: wfa.dataPill(
            _params.trigger.body_text,
            "string_full_utf8"
          ),
          caller_id: wfa.dataPill(_params.trigger.user.sys_id, "reference")
        })
      }
    );
  }
);
```

### Reference Field Dot-Walking

```typescript
Flow(
  {
    $id: Now.ID["manager_notification_flow"],
    name: "Multi-Level Manager Notification"
  },
  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["trigger"] },
    { table: "incident" }
  ),
  _params => {
    // Single-level dot-walk
    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["notify_manager"] },
      {
        ah_to: wfa.dataPill(
          _params.trigger.current.assigned_to.manager.email,
          "string"
        ),
        ah_subject: "Team Assignment",
        ah_body: `Team member ${wfa.dataPill(_params.trigger.current.assigned_to.name, "string")} assigned`,
        record: wfa.dataPill(_params.trigger.current, "reference"),
        table_name: "incident"
      }
    );

    // Multi-level dot-walk (4 levels)
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_skip_manager"] },
      {
        log_level: "info",
        log_message: `Skip manager: ${wfa.dataPill(_params.trigger.current.caller_id.manager.manager.manager.name, "string")}`
      }
    );
  }
);
```

### Action Output DataPills

```typescript
Flow(
  {
    $id: Now.ID["task_creation_flow"],
    name: "Task Creation with Output DataPills"
  },
  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["trigger"] },
    { table: "incident" }
  ),
  _params => {
    const newTask = wfa.action(
      action.core.createRecord,
      { $id: Now.ID["create_task"] },
      {
        table_name: "task",
        values: TemplateValue({
          short_description: "Follow-up task",
          parent: wfa.dataPill(_params.trigger.current, "reference")
        })
      }
    );

    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["assign_task"] },
      {
        table_name: "task",
        record: wfa.dataPill(newTask.Record, "reference"),
        values: TemplateValue({
          assigned_to: wfa.dataPill(
            _params.trigger.current.assigned_to.sys_id,
            "reference"
          )
        })
      }
    );
  }
);
```

### Array DataPills with forEach

```typescript
Flow(
  {
    $id: Now.ID["ci_update_flow"],
    name: "Update Multiple CIs"
  },
  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["trigger"] },
    { table: "change_request" }
  ),
  _params => {
    const cis = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["get_cis"] },
      { table: "cmdb_ci", conditions: "install_status=1" }
    );

    wfa.flowLogic.forEach(
      wfa.dataPill(cis.Records, "records"),
      { $id: Now.ID["process_ci"] },
      ci => {
        wfa.action(
          action.core.createTask,
          { $id: Now.ID["create_task"] },
          {
            task_table: "task",
            field_values: TemplateValue({
              short_description: `Verify: ${wfa.dataPill(ci.name, "string")}`,
              cmdb_ci: wfa.dataPill(ci.sys_id, "reference")
            })
          }
        );
      }
    );
  }
);
```

## Discovering Available DataPill Fields

Use the `get_table_schema` tool to discover available fields and their correct FlowDataType for any ServiceNow table. **This tool is essential for determining the exact type to use in `wfa.dataPill()`'s second parameter.**

### Getting Table Schema

```typescript
// Tool call to get table schema
get_table_schema({
  table: "incident"
});
```

### Schema Response

The tool returns field definitions including:

- **Field name** - Use in datapill expressions (`_params.trigger.current.short_description`)
- **Field type** - The exact FlowDataType to use in `wfa.dataPill(expression, type)` second parameter
- **Reference table** - For reference fields, shows which table they reference for dot-walking

**The field type returned by `get_table_schema` directly maps to the FlowDataType parameter:**

```typescript
// Schema returns: { priority: { type: 'integer' } }
// Use directly in datapill:
wfa.dataPill(_params.trigger.current.priority, "integer");

// Schema returns: { assigned_to: { type: 'reference', reference: 'sys_user' } }
// Use directly in datapill:
wfa.dataPill(_params.trigger.current.assigned_to, "reference");

// Schema returns: { short_description: { type: 'string' } }
// Use directly in datapill:
wfa.dataPill(_params.trigger.current.short_description, "string");
```

### Example Workflow

```
1. Call get_table_schema tool to get field definitions
   → Returns: { short_description: { type: 'string' }, priority: { type: 'integer' }, assigned_to: { type: 'reference', reference: 'sys_user' } }

2. Use field information in datapills
   → wfa.dataPill(_params.trigger.current.short_description, 'string')
   → wfa.dataPill(_params.trigger.current.priority, 'integer')
   → wfa.dataPill(_params.trigger.current.assigned_to, 'reference')

3. For reference fields, use dot-walking to access nested fields
   → wfa.dataPill(_params.trigger.current.assigned_to.email, 'string')
   → wfa.dataPill(_params.trigger.current.assigned_to.manager.name, 'string')
```

**Example:**

```typescript
// Step 1: Get table schema using tool
get_table_schema({ table: "incident" });

// Step 2: Use discovered fields in flow
Flow(
  { $id: Now.ID["incident_flow"], name: "Incident Processing" },
  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["trigger"] },
    { table: "incident" }
  ),
  _params => {
    // Use fields discovered from schema
    wfa.action(
      action.core.log,
      { $id: Now.ID["log"] },
      {
        log_level: "info",
        log_message: `Incident: ${wfa.dataPill(_params.trigger.current.number, "integer")},
        Priority: ${wfa.dataPill(_params.trigger.current.priority, "choice")},
        Assigned: ${wfa.dataPill(_params.trigger.current.assigned_to.name, "string")}`
      }
    );
  }
);
```

---
