# UI_POLICY

<!-- Related skill: platform-view -->

## UI Policy Object Properties

| Property           | Type           | Required | Description                                                                                                                                                  |
| ------------------ | -------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| $id                | Now.ID[string] | Yes      | Unique identifier for the UI Policy using Now.ID format.                                                                                                     |
| table              | String         | Yes      | The table to which the UI Policy applies.                                                                                                                    |
| shortDescription   | String         | Yes      | A brief description of what the UI Policy does.                                                                                                              |
| active             | Boolean        | No       | Whether the UI Policy is active. Default: true                                                                                                               |
| global             | Boolean        | No       | Whether the UI Policy applies globally across all applications. Default: true                                                                                |
| onLoad             | Boolean        | No       | Whether the UI Policy runs when the form loads. Default: false                                                                                               |
| reverseIfFalse     | Boolean        | No       | Whether to reverse the actions when the condition is false. Default: true                                                                                    |
| inherit            | Boolean        | No       | Whether the UI Policy is inherited by tables that extend this table. Default: false                                                                          |
| isolateScript      | Boolean        | No       | Whether to run the script in an isolated scope. Default: false                                                                                               |
| order              | Number         | No       | Execution order (lower numbers execute first). Default: 100                                                                                                  |
| conditions         | String         | No       | A condition script or query that determines when the UI Policy applies.                                                                                      |
| runScripts         | Boolean        | No       | Whether to execute the scripts defined in scriptTrue/scriptFalse. Default: false                                                                             |
| scriptTrue         | String         | No       | JavaScript code that runs when the condition evaluates to true. Must be wrapped in `function onCondition() {}`. Module scripts not supported (client-side).  |
| scriptFalse        | String         | No       | JavaScript code that runs when the condition evaluates to false. Must be wrapped in `function onCondition() {}`. Module scripts not supported (client-side). |
| uiType             | String         | No       | UI type where the policy applies. Options: 'desktop', 'mobile_or_service_portal', 'all'. Default: 'desktop'. Only available when runScripts is true.         |
| actions            | Array          | No       | List of field actions to perform when the UI Policy condition is met.                                                                                        |
| relatedListActions | Array          | No       | List of related list visibility controls.                                                                                                                    |

### UI Policy Action Properties

Each action in the `actions` array has the following properties:

| Property         | Type                | Required | Description                                                                                                                                                              |
| ---------------- | ------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| field            | String              | Yes      | The field to which the action applies. Uses coalesce key `['ui_policy', 'field']` for identity.                                                                          |
| visible          | boolean \| 'ignore' | No       | Whether the field should be visible: `true` (visible), `false` (hidden), `'ignore'` (no change). Default: 'ignore'                                                       |
| readOnly         | boolean \| 'ignore' | No       | Whether the field should be read-only: `true` (disabled), `false` (editable), `'ignore'` (no change). Direct mapping to ServiceNow's 'disabled' field. Default: 'ignore' |
| mandatory        | boolean \| 'ignore' | No       | Whether the field should be mandatory: `true` (required), `false` (optional), `'ignore'` (no change). Default: 'ignore'                                                  |
| cleared          | boolean             | No       | Whether the field should be cleared when the condition is met. Boolean only, no 'ignore' option. Default: false                                                          |
| table            | String              | No       | Table reference for the action. Defaults to the policy table if not specified.                                                                                           |
| value            | String              | No       | Value to set for the field when the policy condition is met.                                                                                                             |
| fieldMessage     | String              | No       | Message to display for the field.                                                                                                                                        |
| fieldMessageType | String              | No       | Type of field message: 'error', 'info', 'warning', 'none'. Default: 'none'                                                                                               |
| valueAction      | String              | No       | Action to perform on the field value. Default: 'ignore'                                                                                                                  |

### UI Policy Related List Action Properties

Each action in the `relatedListActions` array has the following properties:

| Property | Type                | Required | Description                                                                                                                                                                                                                                                            |
| -------- | ------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| list     | String              | No       | Related list identifier as plain GUID (e.g., 'b9edf0ca0a0a0b010035de2d6b579a03') or table.field format (e.g., 'x_snc_app_table.reference_field'). Plugin automatically adds/removes 'REL:' prefix during transformation. Empty/undefined applies to all related lists. |
| visible  | boolean \| 'ignore' | No       | Whether the related list should be visible: `true` (visible), `false` (hidden), `'ignore'` (no change). Default: 'ignore'                                                                                                                                              |

**Note:** Related list actions use coalesce keys `['ui_policy', 'list']` for identity management.

## UI Policy Condition Syntax

UI Policy conditions use ServiceNow's encoded query syntax. Combine multiple conditions using `^` (AND) operator.

| Operator    | Format                    | Example                    | Description                     |
| ----------- | ------------------------- | -------------------------- | ------------------------------- |
| =           | `field`=`value`           | `priority=1`               | Field equals value              |
| !=          | `field`!=`value`          | `state!=6`                 | Field does not equal value      |
| \>          | `field`\>`value`          | `priority>2`               | Field greater than value        |
| \>=         | `field`\>=`value`         | `priority>=2`              | Field greater than or equal     |
| \<          | `field`\<`value`          | `priority<3`               | Field less than value           |
| \<=         | `field`\<=`value`         | `priority<=3`              | Field less than or equal        |
| IN          | `field`IN`v1,v2,v3`       | `stateIN1,2,3`             | Field value in list             |
| BETWEEN     | `field`BETWEEN`min`@`max` | `priorityBETWEEN1@3`       | Field between min and max       |
| LIKE        | `field`LIKE`value`        | `categoryLIKEhardware`     | Field contains value            |
| NOT LIKE    | `field`NOT LIKE`value`    | `categoryNOT LIKEsoftware` | Field does not contain value    |
| STARTSWITH  | `field`STARTSWITH`value`  | `titleSTARTSWITHUrgent`    | Field starts with value         |
| ANYTHING    | `field`ANYTHING           | `descriptionANYTHING`      | Field has any value (not empty) |
| EMPTYSTRING | `field`EMPTYSTRING        | `descriptionEMPTYSTRING`   | Field is empty or null          |
| SAMEAS      | `field1`SAMEAS`field2`    | `prioritySAMEASurgency`    | Field1 equals field2            |
| NSAMEAS     | `field1`NSAMEAS`field2`   | `priorityNSAMEASurgency`   | Field1 does not equal field2    |

**Important for Choice Fields:**

Choice (dropdown) fields store values that may differ from display labels. Always use the **stored value** in conditions, not the label. For example, if a dropdown shows "Travel" (label) but stores "travel" (value), use `category=travel` in your condition.

**To find stored values:** Right-click the field → Show Choice List → check the "Value" column (not "Label").

**Combining Conditions:**

```typescript
// AND operator: all conditions must be true
conditions: "priority=1^state!=6^categoryLIKEhardware";

// Complex condition with multiple operators
conditions: "priorityIN1,2^stateNOT LIKE6,7^assigned_toANYTHING";
```

### Example 1: Basic Field Visibility and Mandatory Control

```typescript
import { UiPolicy } from "@servicenow/sdk/core";

export const highPriorityPolicy = UiPolicy({
  $id: Now.ID["high_priority_policy"],
  table: "incident",
  shortDescription:
    "Make impact and urgency mandatory for high priority incidents",
  onLoad: true,
  conditions: "priority=1",
  actions: [
    { field: "impact", mandatory: true },
    { field: "urgency", mandatory: true }
  ]
});
```

### Example 2: UI Policy with Client-Side Scripts

```typescript
import { UiPolicy } from "@servicenow/sdk/core";

export const categoryPolicy = UiPolicy({
  $id: Now.ID["category_policy"],
  table: "incident",
  shortDescription:
    "Show asset field for hardware incidents with custom messaging",
  onLoad: true,
  conditions: "category=hardware",
  runScripts: true,
  uiType: "all",
  scriptTrue: `function onCondition() {
    g_form.setVisible('asset', true);
    g_form.setMandatory('asset', true);
    g_form.addInfoMessage('Please specify the asset for this hardware incident.');
  }`,
  scriptFalse: `function onCondition() {
    g_form.setVisible('asset', false);
    g_form.setMandatory('asset', false);
    g_form.clearMessages();
  }`,
  actions: [{ field: "asset", visible: true, mandatory: true }]
});
```

### Example 3: Field Clearing and Combined Actions

```typescript
import { UiPolicy } from "@servicenow/sdk/core";

export const incidentResolutionPolicy = UiPolicy({
  $id: Now.ID["incident_resolution_policy"],
  table: "incident",
  shortDescription: "Clear resolution fields when incident is reopened",
  onLoad: true,
  conditions: "state!=6",
  actions: [
    { field: "close_code", cleared: true },
    { field: "close_notes", cleared: true },
    { field: "resolved_at", cleared: true },
    { field: "resolution_notes", visible: false, cleared: true }
  ]
});
```

### Example 4: Field Values, Messages, and Read-Only Control

```typescript
import { UiPolicy } from "@servicenow/sdk/core";

export const criticalIncidentPolicy = UiPolicy({
  $id: Now.ID["critical_incident_policy"],
  table: "incident",
  shortDescription: "Set defaults and lock fields for critical incidents",
  onLoad: true,
  conditions: "priority=1",
  actions: [
    { field: "assignment_group", mandatory: true },
    { field: "urgency", value: "1", readOnly: true },
    { field: "impact", value: "1", readOnly: true },
    {
      field: "short_description",
      fieldMessage: "Critical incident - ensure detailed description",
      fieldMessageType: "warning"
    }
  ]
});
```

### Example 5: Related List Control with Complex Conditions

```typescript
import { UiPolicy } from "@servicenow/sdk/core";

// Control related list visibility based on multiple conditions
export const securityIncidentPolicy = UiPolicy({
  $id: Now.ID["security_incident_policy"],
  table: "incident",
  shortDescription:
    "Lock fields and control related lists for high-priority security incidents",
  onLoad: true,
  conditions: "category=security^priorityIN1,2",
  actions: [
    { field: "caller_id", readOnly: true },
    { field: "assignment_group", readOnly: true },
    { field: "work_notes", mandatory: true }
  ],
  relatedListActions: [
    {
      list: "b9edf0ca0a0a0b010035de2d6b579a03", // Attachments (system relationship GUID)
      visible: false
    },
    {
      list: "incident_task.parent", // Incident Tasks (reference field format)
      visible: true
    }
  ]
});
```

**Note:** For related list identifiers:

- System relationships: Use plain GUID (e.g., `b9edf0ca0a0a0b010035de2d6b579a03`). Plugin adds/removes `REL:` prefix automatically.
- Reference fields: Use `table.field` format (e.g., `incident_task.parent`).

## Transformation Notes

The plugin provides bidirectional transformation between Fluent TypeScript and ServiceNow XML:

**XML → Fluent:**

- Default values are omitted from generated code (e.g., `active: true`, `global: true`, `reverseIfFalse: true`)
- Action properties with 'ignore' values are omitted
- Empty arrays and default scripts are excluded

**Fluent → XML:**

- Boolean action properties (`visible`, `readOnly`, `mandatory`) automatically convert to ServiceNow string format ('true'/'false'/'ignore')
- Scripts are wrapped in CDATA tags
- Related list `REL:` prefix is automatically added/removed
- System fields (sys_id, sys_created_by, etc.) are automatically generated

## Identity and Coalesce Keys

**UI Policy Actions:** Use coalesce keys `['ui_policy', 'field']` - no $id field required. Identity determined by parent policy and field name combination.

**Related List Actions:** Use coalesce keys `['ui_policy', 'list']` - $id field required with `Now.Internal.WithID` pattern. Identity determined by parent policy and list identifier combination.

## Property Mapping

The plugin automatically converts between camelCase (Fluent) and snake_case (ServiceNow):

| Fluent (camelCase) | ServiceNow (snake_case) |
| ------------------ | ----------------------- |
| shortDescription   | short_description       |
| onLoad             | on_load                 |
| reverseIfFalse     | reverse_if_false        |
| isolateScript      | isolate_script          |
| runScripts         | run_scripts             |
| scriptTrue         | script_true             |
| scriptFalse        | script_false            |
| readOnly           | disabled                |
| fieldMessage       | field_message           |
| fieldMessageType   | field_message_type      |
| valueAction        | value_action            |
