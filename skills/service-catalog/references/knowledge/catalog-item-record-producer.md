# CATALOG_ITEM_RECORD_PRODUCER

# Catalog Item Record Producer - Templates and Examples

This knowledge source provides working code templates for ServiceNow Record Producers. For guidelines, requirements, and patterns, activate the `service-catalog` skill first.

## Comprehensive Record Producer with Field Mapping

```typescript
import { CatalogItemRecordProducer } from "@servicenow/sdk/core";

const serviceCatalog = "e0d08b13c3330100c8b837659bba8fb4";
const itServicesCategory = "d258b953c611227a0146101fb1be7c31";

export const comprehensiveIncidentProducer = CatalogItemRecordProducer({
  $id: Now.ID["comprehensive_incident_producer"],
  name: "Report Incident with Full Configuration",
  shortDescription: "Complete incident producer with variables and scripts",
  table: "incident",

  catalogs: [serviceCatalog],
  categories: [itServicesCategory],

  variables: {
    short_description: SingleLineTextVariable({
      question: "Brief Summary",
      mandatory: true,
      mapToField: true,
      field: "short_description",
      order: 100
    }),
    urgency: SelectBoxVariable({
      question: "Urgency",
      mandatory: true,
      mapToField: true,
      field: "urgency",
      choices: {
        "1": { label: "High", sequence: 1 },
        "2": { label: "Medium", sequence: 2 },
        "3": { label: "Low", sequence: 3 }
      },
      order: 200
    }),
    assignment_group: ReferenceVariable({
      question: "Assignment Group",
      mapToField: true,
      field: "assignment_group",
      referenceTable: "sys_user_group",
      order: 300
    })
  },

  script: Now.include("../../scripts/rp-pre-insert.js"),
  postInsertScript: Now.include("../../scripts/rp-post-insert.js"),

  redirectUrl: "generatedRecord",
  view: "ess",
  allowEdit: true
});
```

**rp-pre-insert.js:**

```javascript
(function executeProducerScript(current, producer) {
  // Set defaults and auto-populate fields
  current.impact = 3;
  current.contact_type = "self-service";
  current.caller_id = gs.getUserID();

  // Complex conditional logic
  if (producer.urgency === "1") {
    current.priority = 1;
    current.assignment_group = "Hardware Team";
  }
  // Do NOT use current.update() or current.insert() here
})(current, producer);
```

**rp-post-insert.js:**

```javascript
(function executeProducerScript(current, producer) {
  // Post-creation updates are safe here
  current.work_notes = "Created via Service Catalog at " + gs.nowDateTime();
  current.update();

  // Create related records if needed
  var task = new GlideRecord("sc_task");
  task.initialize();
  task.request = current.sys_id;
  task.short_description =
    "Follow up on incident: " + current.short_description;
  task.insert();
})(current, producer);
```

---

## Different Table Examples

```typescript
// Problem Record Producer
export const problemProducer = CatalogItemRecordProducer({
  $id: Now.ID["problem_producer"],
  name: "Report a Problem",
  shortDescription: "Create a problem record",
  table: "problem",
  catalogs: [serviceCatalog],
  categories: [itServicesCategory],
  script: Now.include("../../scripts/rp-problem-pre-insert.js"),
  redirectUrl: "generatedRecord"
});

// Change Request Producer
export const changeRequestProducer = CatalogItemRecordProducer({
  $id: Now.ID["change_request_producer"],
  name: "Submit Change Request",
  shortDescription: "Create a change request",
  table: "change_request",
  catalogs: [serviceCatalog],
  categories: [itServicesCategory],
  script: Now.include("../../scripts/rp-change-pre-insert.js"),
  redirectUrl: "generatedRecord"
});
```

**rp-problem-pre-insert.js:**

```javascript
(function executeProducerScript(current, producer) {
  current.short_description = producer.problem_summary;
  current.description = producer.problem_details;
  current.impact = producer.impact_level;
  current.priority = producer.priority_level;
  current.caller_id = gs.getUserID();
})(current, producer);
```

**rp-change-pre-insert.js:**

```javascript
(function executeProducerScript(current, producer) {
  current.short_description = producer.change_summary;
  current.description = producer.change_details;
  current.type = producer.change_type;
  current.requested_by = gs.getUserID();
  current.risk = producer.risk_level;
})(current, producer);
```

---

## Properties

| Name             | Type      | Description                                                          |
| ---------------- | --------- | -------------------------------------------------------------------- |
| $id              | string    | **Required.** Unique identifier.                                     |
| table            | TableName | **Required.** Target table (e.g., `'incident'`, `'change_request'`). |
| name             | string    | **Required.** Name to appear in the catalog.                         |
| script           | string    | Server-side script before record creation.                           |
| postInsertScript | string    | Script after record creation. Safe to call `current.update()`.       |
| saveScript       | string    | Script on step save in Catalog Builder.                              |
| redirectUrl      | string    | `'generatedRecord'` (default) or `'catalogHomePage'`.                |
| allowEdit        | boolean   | Allow editing after creation. Default: `false`.                      |
| canCancel        | boolean   | Allow user to cancel. Default: `false`.                              |
| variables        | object    | Variable definitions for the form.                                   |
| variableSets     | array     | Variable set references.                                             |

All catalog item properties (catalogs, categories, accessType, etc.) also apply.

## Field Mapping Methods

| Scenario                       | Recommended Method |
| ------------------------------ | ------------------ |
| Simple text/choice mapping     | `mapToField: true` |
| System values (gs.getUserID()) | Script             |
| Conditional logic              | Script             |
| Calculated values              | Script             |
| Variables in Variable Sets     | Script             |

## Script Types

| Script             | Timing        | Can call update()? |
| ------------------ | ------------- | ------------------ |
| `script`           | Before insert | **No**             |
| `postInsertScript` | After insert  | **Yes**            |
| `saveScript`       | On step save  | No                 |

### Available Script Objects

| Object              | Description                                        |
| ------------------- | -------------------------------------------------- |
| `current`           | GlideRecord of the record being created            |
| `producer.var_name` | Form variable values                               |
| `cat_item`          | Record Producer definition (postInsertScript only) |
| `gs`                | GlideSystem                                        |

### Script Rules

- **Never** call `current.update()` or `current.insert()` in pre-insert script
- **Never** call `current.setAbortAction()`
- **Never** set `current.sys_class_name`
- Use `postInsertScript` for post-creation updates, related records, notifications

## Unsupported Tables

Do **not** create record producers for:

- `sc_request`, `sc_req_item`, `sc_task` — Use Catalog Items instead.

---

## Common User Requests Mapping

| User Request                      | Agent Implementation                           |
| --------------------------------- | ---------------------------------------------- |
| "Report an incident from catalog" | Record Producer targeting incident table       |
| "Submit a change request"         | Record Producer targeting change_request table |
| "Report a problem"                | Record Producer targeting problem table        |
| "Create record with auto-fill"    | Record Producer with pre-insert script         |

---

## Quick Reference

### ALWAYS

- Use `mapToField: true` for simple field mappings
- Use `Now.include(...)` for external script files
- Use `postInsertScript` for post-creation updates and related records
- Set `redirectUrl: "generatedRecord"` to redirect to the created record

### NEVER

- Call `current.update()` or `current.insert()` in pre-insert script
- Call `current.setAbortAction()` in Record Producer scripts
- Set `current.sys_class_name` in scripts
- Create Record Producers for `sc_request`, `sc_req_item`, `sc_task`
