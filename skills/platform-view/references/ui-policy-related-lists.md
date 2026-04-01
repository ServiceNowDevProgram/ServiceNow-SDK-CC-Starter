# UI Policy Related List Actions

This document contains detailed information about UI Policy related list actions.

## Contents

- Overview
- Related List Action Properties
- Related List Identifiers
- Visibility Control
- Finding Related List IDs
- Best Practices
- Examples
- Important Notes
- Next Steps

## Overview

Related list actions control the visibility of related lists on forms based on policy conditions. Unlike field actions that control individual fields, related list actions show or hide entire related lists (child records, reference relationships, etc.) based on the form's current state.

## Related List Action Properties

Each related list action requires the following properties:

| Property | Type                | Mandatory | Default  | Description                                          |
| -------- | ------------------- | --------- | -------- | ---------------------------------------------------- |
| $id      | `Now.ID[string]`    | true      |          | Unique identifier for the related list action        |
| list     | string              | true      |          | Related list identifier (GUID or table.field format) |
| visible  | boolean \| 'ignore' | false     | 'ignore' | Controls related list visibility                     |

## Related List Identifiers

The `list` property identifies which related list to control. There are different formats depending on the type of relationship:

### System-Defined Relationships (GUID Format)

For system-defined related lists, custom queries, attachments, and other system relationships, use the plain GUID:

```javascript
list: "b9edf0ca0a0a0b010035de2d6b579a03";
```

**Note:** The plugin automatically adds the `REL:` prefix when writing to ServiceNow. Do not include `REL:` in your Fluent code.

### Reference Field Relationships (Table.Field Format)

For relationships based on reference fields, use the `table.field` format:

```javascript
list: "incident.caller_id";
```

This format identifies related lists where records in the `incident` table reference the current record through the `caller_id` field.

### Parent-Child Relationships (Table.Table Format)

For parent-child table relationships, use the `parent_table.child_table` format:

```javascript
list: "change_request.change_task";
```

## Visibility Control

The `visible` property controls whether a related list is shown or hidden on the form.

### Options

- `visible: true` - Show the related list
- `visible: false` - Hide the related list
- `visible: 'ignore'` (default) - No change to visibility

### Behavior

- When a related list is hidden, users cannot see or interact with the related records
- The `reverseIfFalse` policy setting inverts the visibility when conditions are false
- Hidden related lists do not affect the underlying records; they only control visibility

## Finding Related List IDs

To find the correct related list identifier, you can use ServiceNow's script execution or query tools.

### Finding System Relationship GUIDs

Use this script in ServiceNow to find related list GUIDs:

```javascript
// Find related lists by name
var gr = new GlideRecord("sys_ui_related_list");
gr.addQuery("name", "CONTAINS", "your_table_name");
gr.query();
while (gr.next()) {
  gs.print(gr.name + " -> " + gr.sys_id);
}
```

### Finding Custom Related List GUIDs

```javascript
// Find custom queries and relationships
var gr = new GlideRecord("sys_ui_related_list");
gr.addQuery("related_list_id", "!=", "");
gr.addQuery("view", "Default view");
gr.query();
while (gr.next()) {
  gs.print(gr.related_list_id + " -> " + gr.sys_id);
}
```

## Best Practices

1. **Use Reference Format When Possible:** For reference field relationships, use `table.field` format for better readability
2. **Document GUID Relationships:** When using GUIDs, add comments explaining what related list they represent
3. **Test Visibility Changes:** Verify that hiding related lists doesn't break business processes
4. **Consider User Experience:** Don't hide critical related lists that users need for their workflows
5. **Group Related Actions:** Define related list actions alongside field actions that affect similar conditions
6. **Use Descriptive IDs:** Use meaningful names in `Now.ID[]` to identify related list actions
7. **Avoid Overuse:** Only control related list visibility when there's a clear business need

## Examples

### Basic Related List Control

```javascript
export const incidentRelatedListPolicy = UiPolicy({
  $id: Now.ID["incident_related_list_control"],
  table: "incident",
  shortDescription: "Control related lists for high priority incidents",
  onLoad: true,
  conditions: "priority=1",
  relatedListActions: [
    {
      $id: Now.ID["show_tasks"],
      list: "incident.parent",
      visible: true
    },
    {
      $id: Now.ID["hide_approvals"],
      list: "sysapproval_approver.sysapproval",
      visible: false
    }
  ]
});
```

### Hide Sensitive Related Lists

```javascript
export const confidentialIncidentPolicy = UiPolicy({
  $id: Now.ID["confidential_incident_policy"],
  table: "incident",
  shortDescription: "Hide sensitive related lists for confidential incidents",
  onLoad: true,
  conditions: "u_confidential=true",
  relatedListActions: [
    {
      $id: Now.ID["hide_public_comments"],
      list: "incident.comments",
      visible: false
    },
    {
      $id: Now.ID["hide_activities"],
      list: "incident.activity",
      visible: false
    }
  ]
});
```

### Progressive Related List Display

```javascript
export const changeProgressionPolicy = UiPolicy({
  $id: Now.ID["change_progression_policy"],
  table: "change_request",
  shortDescription: "Show related lists as change progresses",
  onLoad: true,
  conditions: "state=2",
  relatedListActions: [
    {
      $id: Now.ID["show_risk_assessment"],
      list: "change_request.risk_assessment",
      visible: true
    },
    {
      $id: Now.ID["show_ci_relationships"],
      list: "change_request.affected_cis",
      visible: true
    },
    {
      $id: Now.ID["hide_implementation_plan"],
      list: "change_request.implementation_plan",
      visible: false
    }
  ]
});
```

## Important Notes

- **GUID Format:** Use plain GUIDs without the `REL:` prefix in your Fluent code
- **Reference Fields:** Use `table.field` format for reference field relationships
- **Required IDs:** Each related list action must have a unique `$id`
- **Visibility Only:** Related list actions only control visibility, not data or behavior
- **Reverse Behavior:** The `reverseIfFalse` policy setting affects related list visibility
- **Multiple Policies:** When multiple policies affect the same related list, the most restrictive setting applies
- **Performance:** Hiding related lists can improve form load performance by reducing data queries

## Next Steps

- For script handling details, load `references/ui-policy-scripts.md` via `load_skill_resource` with skill name "ui-layout-control"
- For field action details, refer to `ui-policy-guide.md` loaded via `load_skill_resource`
- For fluent API and more examples, use `get_knowledge_source` tool to get the UI Policy knowledge source
