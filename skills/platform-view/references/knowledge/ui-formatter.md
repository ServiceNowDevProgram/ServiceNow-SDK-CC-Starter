# UI_FORMATTER

<!-- Related skill: platform-view -->

## sys_ui_formatter table

Define UI formatters `[sys_ui_formatter]` that can be added to form sections to display contextual or computed UI elements (for example, activity stream, process flow, variables, etc.).

### Properties (columns)

| Field Name          | Type      | Mandatory | Default     | Comments                                                                            |
| ------------------- | --------- | --------- | ----------- | ----------------------------------------------------------------------------------- |
| `name`              | string    | true      |             | The display name of the formatter as shown in form.                                 |
| `formatter`         | string    | true      |             | The underlying UI macro/formatter id used to render this formatter.                 |
| `type`              | string    | false     | `formatter` | Type identifier used by the platform to classify the record.                        |
| `active`            | boolean   | false     | `true`      | Whether this formatter is available to use on forms.                                |
| `angular_module`    | string    | false     |             | Optional Angular module used by the formatter macro (if applicable).                |
| `seismic_component` | reference | false     |             | Optional Seismic component identifier if the formatter renders a Seismic component. |
| `sys_scope`         | reference | false     |             | Application scope.                                                                  |

## Built-in UI Formatters

| Formatter Name                      | Macro/XML                                          | Angular Module           | When to Use                                                                                                                                                                                                                                                                     | Prerequisites                                                                                                                                                                                | Position                                                      |
| ----------------------------------- | -------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| Activity Formatter                  | `activity.xml`                                     | -                        | Show a chronological stream of activities related to a record, such as journal entries like comments and work notes                                                                                                                                                             | Journal fields present and activity stream enabled, available by default for every table                                                                                                     | Should be added as last element in the section                |
| Process Flow Formatter              | `process_flow`                                     | -                        | Shows a graphical summary of a record's lifecycle stages at the top of the form                                                                                                                                                                                                 | Stage Model configured for the table                                                                                                                                                         | Should be added as first element in the section               |
| CI Relations Formatter              | `ui_ng_relation_formatter.xml`                     | `sn.ciRelationFormatter` | Show CI relationship map in CMDB contexts or visualizes the relationships between the current Configuration Item (CI) and other related CIs in a map view                                                                                                                       | CMDB installed; target records are CIs with relationship records in `cmdb_rel_ci` and appropriate relationship types; target table is extended from `cmdb_ci`                                | Should be added as first element in the section               |
| Parent Breadcrumb Formatter         | `parent_crumbs`                                    | -                        | Show parent hierarchy breadcrumb for records with a self-referential parent and number field defining hierarchy or show a navigational trail of parent records for the current task                                                                                             | Table must have a reference field `parent` field defining hierarchy; records have parent values to render a breadcrumb trail                                                                 | Should be added as first element in the section               |
| Contextual Search Results Formatter | `cxs_table_search.xml`                             | -                        | Automatically searches defined sources (like Knowledge Base, Catalog, Communities, etc.) using fields from the current record (e.g., short description, description) as search terms. It displays suggested results directly in the form, helping agents resolve issues faster. | Contextual Search enabled and configured; knowledge bases indexed; search sources/mappings defined for the target table so related results can be returned                                   | Should be added below the search context field in the section |
| Variables Editor Formatter          | `com_glideapp_questionset_default_question_editor` | -                        | Displays the values of questions for records generated from a record producer for task-extended tables                                                                                                                                                                          | Table extends a task table; questions are defined for the table; Record is created from a record producer                                                                                    | -                                                             |
| Checklist Formatter                 | `inline_checklist_macro`                           | `inlineChecklist`        | Display and manage checklist items on task records to track sub-tasks or steps                                                                                                                                                                                                  | Checklist feature/plugin enabled; target table supports checklists (typically task-derived); checklist definitions/items exist or can be added; user has rights to view/edit checklist items | Should be added as last element in the section                |
| Attached Knowledge Formatter        | `attached_knowledge`                               | -                        | Used to display a list of knowledge articles that have been linked to a specific task record, such as an Incident, Problem, or Case                                                                                                                                             | Knowledge Management installed; At least one knowledge article is attached to the record, target table is 'task' or extended from 'task' table                                               | Should be added as last element in the section                |

## Custom Formatter

A custom formatter is a classic form UI component you define when built-in formatters don’t satisfy your UX or data-display needs. It is a `sys_ui_formatter` record that references a UI macro (and optionally an Angular module) to render specialized, non-field content on a form.

**⚠️ CRITICAL: DO NOT CREATE CUSTOM FORMATTERS - NOT SUPPORTED IN FLUENT**
Custom formatters are NOT supported in Fluent and should NOT be created. Always use the built-in formatters listed in this documentation instead.

## Steps to be followed sequentially to create and add a formatter to a form

Follow these exact steps for a target table/view and formatter macro/label.

1. Check formatter exists (`sys_ui_formatter`)

   **⚠️ IMPORTANT: Activity Formatter and Attached Knowledge Formatter already exist globally - NEVER create new records for these formatters**
   - For **Activity Formatter** (`activity.xml`) and **Attached Knowledge Formatter** (`attached_knowledge`): Skip to step 4 (Check section exists) as these formatters already exist globally in the `sys_ui_formatter` table

   - For all other formatters, query sys_ui_formatter for target table with formatter name using `runQuery` tool:

   `table`: 'sys_ui_formatter',
   `encodedQuery`: 'table=<table_name>^formatter=<formatter_name>'
   - if formatter is not found, Query sys_ui_formatter for global table using `runQuery` tool

   `table`: 'sys_ui_formatter',
   `encodedQuery`: 'table=global^formatter=<formatter_name>'
   - If formatter still not found, and table is extended from another table query with table extended from using `runQuery` tool:

   `table`: 'sys_ui_formatter',
   `encodedQuery`: 'table=<table_extended_from>^formatter=<formatter_name>'
   - If formatter is found, use the sys_id as `sys_ui_formatter` and formatter as `element` in the sys_ui_element record

   - If no formatter is found, create a `sys_ui_formatter` record with:

   - `name` = display label (for example, 'Task Process Flow Formatter')
   - `formatter` = macro name (for example, `process_flow`) or macro scoped_name in case of custom formatter (for example, `my_app_my_macro`)
   - `table` = <table_name>
   - `active` = true, `type` = 'formatter'

2. If using the Process Flow Formatter (process_flow.xml), ensure stages are configured:
   - Verify existing Stage Model entries in `sys_process_flow` table using `runQuery` tool
     `table`: sys_process_flow
     `encodedQuery`: table=<table_name>

   - If none exist, add records in `sys_process_flow` (create the appropriate stage definitions for the target table).
     - `name` = <table_name>
     - `table` = <table_name>
     - `active` = true, `type` = 'process_flow'
     - `order` = 10
     - `condition` = <define_condition>(for example, `status=open^EQ` where status is a choice field on the target table)
     - `label` = <label>(for example, 'Open')
     - `name` = <name>(for example, 'open')

3. If using Contextual Search Results Formatter (cxs_table_search.xml), ensure search sources are configured in the table:
   - Verify existing search source entries in `cxs_table_config` table

   `table`: cxs_table_config,
   `encodedQuery`: table=<table_name>
   - If none exist, [create a `cxs_table_config` record](#create-a-cxs_table_config-record)

4. Check section exists (`sys_ui_section`)

   **Query the sys_ui_section Table using `runQuery` tool to check if section exists for the target table and view (use Default view if view is not specified)**

   `table`: sys_ui_section,
   `encodedQuery`: name=<table_name>^view='<view_name>'
   - If only one section is found, use the sys_id as section in the sys_ui_element record.

   - if multiple sections are found, you will have to ask the user to select the section from the list and use the selected section sys_id as name in the sys_ui_element record

   - if no sections are found, create one `sys_ui_section` record with:
     - `name` = <table_name>
     - `caption` = '' (caption should be empty for first section)
     - `view` = <view_name> or 'Default view'
     - `active` = true, `type` = 'section'
     - `type` = 'section'

   - if multiple sections are created, create a form (sys_ui_form) and form-section(sys_ui_form_section) record
     - create sys_ui_form record with:
       - `name` = <table_name>
       - `view` = <view_id> or 'Default view'

     - create sys_ui_form_section record with:
       - `sys_ui_section` = <section_sys_id>, import section record if created in seperate file
       - `position` = <position>
       - `sys_ui_form` = <form_sys_id>, import form record if created in seperate file

5. Add formatter element (`sys_ui_element`) record with:
   - Query the sys_ui_element Table using `runQuery` tool to check if element exists for the target section:
     `table`: sys_ui_element,
     `encodedQuery`: sys_ui_section=<section_sys_id>^element=<formatter_name_with_xml>

   - If element is found, inform user that element already exists and ask for confirmation to update the element.

   - If element is not found, create a `sys_ui_element` record with:
     - `sys_ui_section` = <section_sys_id> ❌ do not use Now.ID[<section>]
     - `element` = formatter from sys_ui_formatter (for example, `activity.xml`, `process_flow.xml`, etc.)
     - `type` = 'formatter'
     - `sys_ui_formatter` = reference to the formatter record
     - `position` = numbers for placement.

6. Ensure at least one other non-formatter element exists in the section so the formatter renders.
7. Adjust position of other elements if required.

## Critical Requirements for Parent Breadcrumb Formatter

**⚠️ CRITICAL: EXACT FIELD NAME REQUIREMENT**

For the Parent Breadcrumb Formatter to work correctly, the table MUST have a reference field named exactly `parent` that references the same table the record is on, or a table it extends from (like the base Task table). This is a strict requirement:

- ✅ Field name: `parent`
- ❌ Field name: `parent_ticket` (not acceptable)
- ❌ Field name: `parent_task` (not acceptable)
- ❌ Field name: `parent_record` (not acceptable)
- ❌ Any other variation (not acceptable)

The field must:

1. Be named exactly `parent` (case-sensitive)
2. Be a reference field type
3. Allow records to form a hierarchical parent-child relationship
4. Reference the same table the record is on, or a table it extends from (like the base Task table)

If the table does not have a field named exactly `parent`, you must create one before implementing the Parent Breadcrumb Formatter.

## Create a cxs_table_config record

- Get search context by quering cxs_context_config table using `runQuery` tool

  `table`: cxs_context_config,

- Ask user to select the search context from the list (using interview tool) and use the selected search context sys_id as name in the cxs_context_config record .

- Create a `cxs_table_config` record with:
  - `table` = <table_name>
  - `name` = <table_name>
  - `active` = true
  - `cxs_context_config` = <cxs_context_config_sys_id>
  - `ui_type` = 'platform'
  - `results_header_text` = <title>(for example, 'Related Search Results')
  - `limit` = <limit>(for example, 10)
  - `results_per_page` = <results_per_page>(for example, 10)
  - `related_search_active` = true
  - `enable_source_selector` = true
  - `match_condition` = <match_condition>(See "CRITICAL ALERT" below - must verify field exists in table first)

### 🚨 CRITICAL ALERT: Field Validation for match_condition field

❌ NEVER use fields that don't exist in the table! for match_condition field**
✅ ALWAYS query sys_dictionary for the table FIRST** to verify field existence before creating match conditions
✅ Only use fields that actually exist** in the selected table
Common examples: `active=true^EQ` **ONLY** if table has 'active' field, or `state!=closed^EQ` if table has 'state' field, `u_status=active^EQ` **ONLY** if table has 'u_status' field
✅ use `sys_created_on!=NULL^EQ` as a safe fallback for match_condition field**

- Query sys_glide_object table to get the String id using `runQuery` tool

`table`: sys_glide_object
encodedQuery: name=string

- Query the sys_dictionary to get all the String fields of the table using `runQuery` tool

`table`: sys_dictionary
encodedQuery: table=<table_name>^internal_type=<string_id>

- If short_description or description or any descriptive field is included in the table, use one of them as the search field.

- if short_description or description field is not available, Ask user to select the String field from the list (using interview tool) and use the selected String fields as name in the cxs_table_field_config record.

- Prefered search field is short_description or description.
  - Create only one `cxs_table_field_config` record with the search field:
    - `active` = true, `type` = 'cxs_table_field_config'
    - `cxs_table_config` = <cxs_table_config_sys_id> // use the sys_id of the cxs_table_config record created in previous step
    - `field` = <field_name>(for example, 'short_description')
    - `default_config` = true
    - `sys_name` = <field_name>(for example, 'Short Description[short_description]')
    - `name` = <field_name>(for example, 'Short Description[short_description]')

## Examples

### Create formatter

#### Create a Activity formatter record

**Activity Formatter already exists globally in ServiceNow - do not create new records for it.**

Instead, directly add the formatter element to your form section using the existing global Activity Formatter:

```typescript
// Add Activity Formatter element to section (no need to create formatter record)
Record({
  $id: Now.ID["activity_formatter_element"],
  table: "sys_ui_element",
  data: {
    sys_ui_section: section.$id, // reference to your section
    element: "activity.xml",
    type: "formatter",
    position: 99 // should be last element in section
  }
});
```

#### Create a Process Flow formatter record

```typescript
//src/fluent/ui/process_flow_formatter.now.ts
import { Record } from "@servicenow/sdk/core";

export const process_flow_formatter = Record({
  $id: Now.ID["process_flow_formatter"],
  table: "sys_ui_formatter",
  data: {
    name: "Process Flow Formatter",
    type: "formatter",
    formatter: "process_flow.xml",
    table: "table_name",
    active: true
  }
});
```

#### create parent breadcrumb formatter

```typescript
//src/fluent/ui/parent_breadcrumb_formatter.now.ts

import { Record } from "@servicenow/sdk/core";

export const parent_breadcrumb_formatter = Record({
  $id: Now.ID["parent_breadcrumb_formatter"],
  table: "sys_ui_formatter",
  data: {
    name: "parent_breadcrumb",
    type: "formatter",
    formatter: "parent_crumbs.xml", // confirm display name in your instance
    table: "table_name",
    active: true
  }
});
```

### Create Checklist formatter record

```typescript
//src/fluent/ui/checklist_formatter.now.ts
import { Record } from "@servicenow/sdk/core";

export const checklist_formatter = Record({
  $id: Now.ID["checklist_formatter"],
  table: "sys_ui_formatter",
  data: {
    name: "Checklist Formatter",
    type: "formatter",
    formatter: "inline_checklist_macro.xml",
    angular_module: "inlineChecklist",
    table: "table_name",
    active: true
  }
});
```

### Create a CI Relations formatter record

```typescript
//src/fluent/ui/ci_relations_formatter.now.ts
import { Record } from "@servicenow/sdk/core";

export const ci_relations_formatter = Record({
  $id: Now.ID["ci_relations_formatter"],
  table: "sys_ui_formatter",
  data: {
    name: "CI Relations Formatter",
    type: "formatter",
    formatter: "ui_ng_relation_formatter.xml",
    angular_module: "sn.ciRelationFormatter",
    table: "table_name",
    active: true
  }
});
```

### Create custom formatter record

```typescript
import { Record } from "@servicenow/sdk/core";

export const ci_relations_formatter = Record({
  $id: Now.ID["custom_formatter"],
  table: "sys_ui_formatter",
  data: {
    name: "custom Formatter",
    type: "formatter",
    formatter: "my_macro_custom_macro.xml", // macro scoped_name in case of custom formatter
    table: "table_name",
    active: true
  }
});
```

### Example to Add formatter to the form

#### Create a section record if not available already

```typescript
import { Record } from "@servicenow/sdk/core";

export const section = Record({
  $id: Now.ID["section"],
  table: "sys_ui_section",
  data: {
    name: "table_name",
    caption: "", // caption should be empty for first section
    view: "Default view",
    active: true,
    type: "section"
  }
});
```

#### Add formatter element to the section

```typescript
import { Record } from "@servicenow/sdk/core";
import { section } from "./section.now.ts"; // import section record created in previous step if its in different file
// add formatter element to the section
Record({
  $id: Now.ID["process_flow_section"],
  table: "sys_ui_element",
  data: {
    sys_ui_section: section.$id, // id for the referenced section record, if section record is in different file import it
    element: "process_flow.xml",
    type: "formatter",
    formatter: process_flow_formatter.$id, // id for the referenced section record
    position: 1
  }
});

// add non-formatter element to the section
Record({
  $id: Now.ID["non_formatter_element"],
  table: "sys_ui_element",
  data: {
    sys_ui_section: section.$id, // id for the referenced section record
    element: "name",
    position: 1,
    type: "element"
  }
});
```

### Add stages to sys_process_flow table if not available already for process flow formatter

```typescript
import { Record } from "@servicenow/sdk/core";

// all the stages are added in sys_process_flow_stages table
Record({
  $id: Now.ID["49451a99ff30b2107fafffffffffffe4"],
  table: "sys_process_flow",
  data: {
    active: true,
    condition: "state=new^EQ", // or stage=1^EQ as per table fields
    label: "New",
    name: "Task Flow - New State",
    order: "100",
    table: "table_name"
  }
});

Record({
  $id: Now.ID["49451a99ff30b2107fafffffffffffe4"],
  table: "sys_process_flow",
  data: {
    active: true,
    condition: "state=Assess^EQ", // or stage=1^EQ as per table fields
    label: "Assess",
    name: "Task Flow - Assess State",
    order: "200",
    table: "table_name"
  }
});

Record({
  $id: Now.ID["49451a99ff30b2107fafffffffffffe6"],
  table: "sys_process_flow",
  data: {
    active: true,
    condition: "state=In progress^EQ", // or stage=1^EQ as per table fields
    label: "In Progress",
    name: "Task Flow - In Progress",
    order: "300",
    table: "table_name"
  }
});
```

### Create contextual search record in cxs_table_config table

```typescript
import { Record } from "@servicenow/sdk/core";

// create contextual search record
export const case_task_cxs_table_config = Record({
  $id: Now.ID["case_task_cxs_table_config"],
  table: "cxs_table_config",
  data: {
    table: "sn_formatter_demo_case_task",
    active: true,
    cxs_context_config: "03ddb541c31121005655107698ba8f7f", // Knowledge Base Search context
    results_header_text: "Related Search Results",
    limit: 10,
    name: "Case Task[sn_formatter_demo_case_task]", // name can be same as table name
    ui_type: "platform",
    results_per_page: 10,
    match_condition: "sys_created_on!=NULL^EQ" // fallback option
  }
});

// add a field related to the contextual search record
export const case_task_cxs_field_config = Record({
  $id: Now.ID["case_task_cxs_field_config"],
  table: "cxs_table_field_config",
  data: {
    active: true,
    cxs_table_config: case_task_cxs_table_config.$id,
    field: "short_description",
    name: "short_description[short_description]",
    default_config: true,
    sys_name: "short_description[short_description]"
  }
});
```

### Create form and form section record for multiple sections

```typescript
import { Record } from "@servicenow/sdk/core";

// create form record
export const form = Record({
  $id: Now.ID["form"],
  table: "sys_ui_form",
  data: {
    table: "table_name",
    view: "Default view"
  }
});

// create form section record
export const form_section = Record({
  $id: Now.ID["form_section"],
  table: "sys_ui_form_section",
  data: {
    sys_ui_form: form.$id, // import form record if created in seperate file
    sys_ui_section: section.$id, // import section record if created in seperate file
    position: 1
  }
});

// create form section record
export const form_section_2 = Record({
  $id: Now.ID["form_section_2"],
  table: "sys_ui_form_section",
  data: {
    sys_ui_form: form.$id, // import form record if created in seperate file
    sys_ui_section: section_2.$id, // import section record if created in seperate file
    position: 2
  }
});
```
