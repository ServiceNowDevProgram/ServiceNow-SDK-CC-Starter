# List Control Examples

This document provides working implementation examples for creating ServiceNow List Controls using the Record API.

## Contents

- Example 1: Disable pagination count for large audit table
- Example 2: Enable cmdb list to show information about related list and disable the natural language query
- Example 3: Hide All Links and Filters breadcrumbs and drilldown link
- Example 4: Hide Related List if Empty
- Example 5: Role and Condition Combined (Separate Capabilities)
- Example 6: Filter and Link with Role Restrictions
- Example 7: List Edit Configuration - Row Edit Mode
- Example 8: List Edit with Reference Qualifier Tag
- Example 9: Disable List Editing
- Example 10: Enable hierarchical lists to compare related list of two record in a table
- Example 11: Omit related list count on custom relationship to improve performance.

## Example 1: Disable pagination count for large audit table

```typescript
import { Record } from "@servicenow/sdk/core";

export const performanceControl = Record({
  $id: Now.ID["performance-list-control"],
  table: "sys_ui_list_control",
  data: {
    name: "sys_audit",
    omit_count: true,
    omit_related_list_count: true
  }
});
```

**Use case**: Tables with >10,000 records where counting impacts performance.

## Example 2: Enable cmdb list to show information about related list and disable the natural language query

```typescript
import { Record } from "@servicenow/sdk/core";

export const hierarchicalControl = Record({
  $id: Now.ID["hierarchical-control"],
  table: "sys_ui_list_control",
  data: {
    name: "cmdb_ci",
    hierarchical_lists: true,
    disable_nlq: true
  }
});
```

**Use case**: Configuration Item (CI) tables with parent-child relationships.

## Example 3: Hide All Links and Filters breadcrumbs and drilldown link

Remove all links and filter options from a list, it will remove filter breadcrumbs and no drilldown link:

```typescript
import { Record } from "@servicenow/sdk/core";

export const restrictedListControl = Record({
  $id: Now.ID["restricted-list-control"],
  table: "sys_ui_list_control",
  data: {
    name: "sensitive_data_table",
    omit_links: true,
    omit_filters: true,
    omit_drilldown_link: true
  }
});
```

**Result**:

- No reference links to other tables
- No filters or breadcrumbs
- First column is not clickable (no drilldown)
- Users can still use reference icon if needed

## Example 4: Hide Related List if Empty

Hide a related list completely when it has no records:

```typescript
import { Record } from "@servicenow/sdk/core";

export const omitIfEmptyControl = Record({
  $id: Now.ID["omit-if-empty-control"],
  table: "sys_ui_list_control",
  data: {
    name: "incident",
    related_list: "incident.parent_incident",
    omit_if_empty: true
  }
});
```

**Result**: Related list header and section are completely hidden when there are no child incidents.

## Example 5: Role and Condition Combined (Separate Capabilities)

Combine roles for New button and condition for Edit button:

```typescript
import { Record } from "@servicenow/sdk/core";

export const roleAndConditionControl = Record({
  $id: Now.ID["role-and-condition-control"],
  table: "sys_ui_list_control",
  data: {
    name: "incident",
    related_list: "task_sla.task",
    new_roles: ["admin", "sla_admin"], // Only these roles see New
    edit_condition: Now.include("../scripts/hidEditForCompletedSLA.js")
  }
});
```

**Important**: Don't use `omit_new_button: true` and `new_roles` together for the same capability. The omit flag will override role permissions.

**Result**:

- Only admin and sla_admin roles see New button
- Edit button is hidden when SLA is completed

## Example 6: Filter and Link with Role Restrictions

Restrict who can see filters and links:

```typescript
import { Record } from "@servicenow/sdk/core";

export const filterRoleControl = Record({
  $id: Now.ID["filter-role-control"],
  table: "sys_ui_list_control",
  data: {
    name: "incident",
    filter_roles: ["admin", "report_viewer"],
    link_roles: ["admin", "itil", "user"]
  }
});
```

**Result**:

- Only admin and report_viewer can access filters
- Admin, itil, and user roles can click links

## Example 7: List Edit Configuration - Row Edit Mode

Enable row-based editing for incident priority updates where users might change multiple fields:

```typescript
import { Record } from "@servicenow/sdk/core";

export const incidentRowEdit = Record({
  $id: Now.ID["incident-row-edit"],
  table: "sys_ui_list_control",
  data: {
    name: "x_snc_incident",
    list_edit_type: "save_by_row" // Save when user navigates away or clicks Save
  }
});
```

**Use case**: When users need to update multiple fields in a row before committing changes.

**Behavior**:

- User clicks a cell
- Edits multiple cells in the same row
- Changes saved when user navigates away from row OR clicks Save icon
- Allows bulk field updates before save

## Example 8: List Edit with Reference Qualifier Tag

Use reference qualifier tag to filter assignment group based on category during list editing:

```typescript
import { Record } from "@servicenow/sdk/core";

export const incidentListEditWithRefQual = Record({
  $id: Now.ID["incident-ref-qual-tag"],
  table: "sys_ui_list_control",
  data: {
    name: "x_snc_incident",
    list_edit_ref_qual_tag: "list_edit_incident" // Tag passed to reference qualifier
  }
});
```

**Reference Qualifier Script** (on assignment_group field):

```javascript
// In the reference qualifier for assignment_group field
if (
  typeof listEditRefQualTag != "undefined" &&
  listEditRefQualTag == "list_edit_incident"
) {
  // When editing in list view, filter groups based on category
  if (current.category) {
    return "type=" + current.category;
  }
}
// Normal form-based qualifier
return "active=true";
```

**Use case**: Different filtering logic for reference fields when editing in list view vs form view.

## Example 9: Disable List Editing

Disable inline editing for sensitive data table:

```typescript
import { Record } from "@servicenow/sdk/core";

export const disableListEdit = Record({
  $id: Now.ID["disable-list-edit"],
  table: "sys_ui_list_control",
  data: {
    name: "x_snc_financial_records",
    list_edit_type: "disabled" // Prevent inline editing
  }
});
```

_Use case_: Sensitive tables where changes should only happen through forms with proper validation and business rules.

## Example 10: Enable hierarchical lists to compare related list of two record in a table

```typescript
import { Record } from "@servicenow/sdk/core";

export const hierarchicalList = Record({
  $id: Now.ID["hierarchical-list"],
  table: "sys_ui_list_control",
  data: {
    name: "x_snc_incident",
    hierarchical_lists: true // Enable hierarchical lists
  }
});
```

_Use case_: Enable hierarchical lists to compare related list of two record in a table.

## Example 11: List control on custom relationship to omit related list count.

**NOTE**: When no simple reference field exists (e.g., `table.field`), the relationship `sys_id` is required. For creating custom relationships, use `get_knowledge_source` tool to get the **RELATIONSHIP** knowledge source.

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";
import { activeHighPriorityRelationship } from "../relationships/game_allotment_relationships.now";

// List control for sn_sportshub_sports table to omit related list count
// This improves performance by avoiding database queries to calculate related record counts
export const sportsTableListControl = Record({
  $id: Now.ID["sports_table_list_control"],
  table: "sys_ui_list_control",
  data: {
    name: "sn_sportshub_sports",
    related_list: `REL:${activeHighPriorityRelationship.$id}`, // Reference to relationship variable
    omit_related_list_count: "true"
  }
});
```
