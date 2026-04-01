# WFA_FLOW_ACTIONS_SERVICE_CATALOG

# Workflow Automation Flow Actions - Service Catalog

The Service Catalog Actions API reference provides complete technical specifications for programmatic interaction with ServiceNow Service Catalog. These actions enable automated catalog request submission, variable population from templates, and fulfillment task creation. This document contains action signatures, parameter definitions, types, and syntax examples.

Use these actions to build automated provisioning workflows, bulk ordering systems, template-based request standardization, and integrated fulfillment processes.

---

## action.core.submitCatalogItemRequest

Submit a catalog item request programmatically with configurable blocking/non-blocking execution and comprehensive error handling.

Creates a requested item [sc_req_item] on a Service Catalog Request [sc_req]. The request can be configured to be a blocking or non-blocking task. Pass request variables to provide additional information or allow the fulfiller to modify the variables.

### Input Parameters

| Parameter                | Type      | Default               | Mandatory | Description                                                     |
| ------------------------ | --------- | --------------------- | --------- | --------------------------------------------------------------- |
| catalog_item             | reference | -                     | Yes       | sys_id of catalog item (sc_cat_item) to order                   |
| catalog_item_inputs      | string    | -                     | No        | Variable assignments in ^-delimited format (max 32000000 chars) |
| sysparm_quantity         | integer   | 1                     | No        | Number of items to order                                        |
| wait_for_completion      | boolean   | -                     | No        | Wait for request fulfillment before continuing (blocking)       |
| timeout_flag             | boolean   | -                     | No        | Enable timeout for blocking requests                            |
| sysparm_requested_for    | reference | -                     | No        | User to request for (sys_user). Defaults to current user.       |
| delivery_address         | string    | -                     | No        | Delivery address for physical items (max 4000 chars)            |
| special_instructions     | string    | -                     | No        | Special fulfillment instructions (max 4000 chars)               |
| schedule                 | reference | -                     | No        | Schedule for work duration (cmn_schedule)                       |
| duration                 | duration  | '1970-01-01 00:00:00' | No        | Duration for scheduled work                                     |
| \_snc_dont_fail_on_error | boolean   | -                     | No        | Continue flow execution on error (suppresses failures)          |

### Output Fields

| Field          | Type      | Description                                       |
| -------------- | --------- | ------------------------------------------------- |
| requested_item | reference | created request item (sc_req_item)                |
| status         | choice    | Status code: 0=success, 1=error, 2=timeout        |
| error_message  | string    | Error description if status ≠ 0 (max 10000 chars) |

### Usage Examples

**Basic Catalog Request:**

```typescript
import { laptopCatalogItem } from "../catalogs/laptop-catalog";

// In flow:
const request = wfa.action(
  action.core.submitCatalogItemRequest,
  { $id: Now.ID["submit_laptop_request"], annotation: "Submit laptop order" },
  {
    catalog_item: `${laptopCatalogItem}`, // Reference CatalogItem with template literal
    sysparm_quantity: 1,
    catalog_item_inputs: "memory=16GB^storage=512GB^os=Windows 11",
    sysparm_requested_for: wfa.dataPill(
      _params.trigger.current.caller_id,
      "reference"
    )
  }
);
```

**With Status Checking:**

```typescript
import { laptopCatalogItem } from "../catalogs/laptop-catalog";

// In flow:
const request = wfa.action(
  action.core.submitCatalogItemRequest,
  { $id: Now.ID["submit_request"] },
  {
    catalog_item: `${laptopCatalogItem}`,
    catalog_item_inputs: "configuration=standard^urgency=high"
  }
);

wfa.flowLogic.if(
  {
    $id: Now.ID["check_success"],
    condition: `${wfa.dataPill(request.status, "string")}=0`
  },
  () => {
    // Success path - use request.requested_item
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["update_parent"] },
      {
        table_name: "incident",
        record: wfa.dataPill(_params.trigger.current, "reference"),
        values: TemplateValue({
          work_notes: `Request submitted: ${wfa.dataPill(request.requested_item, "reference")}`
        })
      }
    );
  }
);
wfa.flowLogic.elseIf(
  {
    $id: Now.ID["check_error"],
    condition: `${wfa.dataPill(request.status, "string")}=1`
  },
  () => {
    // Error path - log error message
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_error"] },
      {
        log_level: "error",
        log_message: wfa.dataPill(request.error_message, "string")
      }
    );
  }
);
wfa.flowLogic.elseIf(
  {
    $id: Now.ID["check_timeout"],
    condition: `${wfa.dataPill(request.status, "string")}=2`
  },
  () => {
    // Timeout path
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_timeout"] },
      {
        log_level: "warn",
        log_message: "Request submission timed out - may still be processing"
      }
    );
  }
);
```

**With Error Suppression:**

```typescript
import { optionalCatalogItem } from "../catalogs/optional-catalog";

// In flow:
const request = wfa.action(
  action.core.submitCatalogItemRequest,
  { $id: Now.ID["optional_request"] },
  {
    catalog_item: `${optionalCatalogItem}`,
    _snc_dont_fail_on_error: true // Flow continues even if submission fails
  }
);

// Flow continues regardless of success/failure
// Check status to determine if request was created
```

**Blocking Request (Wait for Completion):**

```typescript
import { urgentCatalogItem } from "../catalogs/urgent-catalog";

// In flow:
const request = wfa.action(
  action.core.submitCatalogItemRequest,
  { $id: Now.ID["blocking_request"] },
  {
    catalog_item: `${urgentCatalogItem}`,
    wait_for_completion: true, // Flow pauses until request is fulfilled
    timeout_flag: true // Enable timeout to prevent indefinite wait
  }
);

// Flow resumes here only after request fulfillment completes
```

### Important Notes

- **CatalogItem Must Be Imported** - If CatalogItem is not in the same file, search the codebase to find its definition file, import it, and reference with template literal: `` `${catalogItem}` ``
- **Always Check Status** - Verify `status` output (0/1/2) before using `requested_item`
- **Variable Format** - `catalog_item_inputs` uses encoded query format: `variable1=value1^variable2=value2`
- **Blocking Performance** - `wait_for_completion: true` can significantly slow flows for complex items with long fulfillment times
- **Status Meanings** - 0=success (requested_item contains valid sys_id), 1=error (check error_message), 2=timeout (request may still be processing)
- **Default User** - If `sysparm_requested_for` is not specified, request is created for the current user executing the flow
- **Error Handling** - Use conditional logic to handle all three status codes appropriately if needed by the business logic
- **Quantity** - sysparm_quantity defaults to 1 if not provided

---

## action.core.getCatalogVariables

Retrieve catalog variables from a template catalog item and populate them on a request item.

Surfaces Catalog Variables on the Flow and/or on a given record. Obtains the catalog variables in a Flow or from a record in a table. Uses an existing catalog item as a template to populate the available variables.

### Input Parameters

| Parameter             | Type        | Default | Mandatory | Description                                                              |
| --------------------- | ----------- | ------- | --------- | ------------------------------------------------------------------------ |
| requested_item        | reference   | -       | Yes       | sys_id of request item to populate (sc_req_item)                         |
| template_catalog_item | reference   | -       | Yes       | sys_id of catalog item template (st_sys_catalog_items_and_variable_sets) |
| catalog_variables     | slushbucket | -       | No        | Specific variables to copy from template (max 12000 chars)               |

### Output Fields

No output fields. This is an informational action with side effects (populates variables on the request item).

### Usage Examples

**IMPORTANT:** CatalogItem definitions are typically in separate files. Search the codebase to find the CatalogItem definition file and import it into the flow.

**Catalog Item Definition Example (catalogs/laptop-catalog.ts or any folder):**

```typescript
import {
  CatalogItem,
  SingleLineTextVariable,
  ReferenceVariable
} from "@servicenow/sdk/core";

export const laptopCatalogItem = CatalogItem({
  $id: Now.ID["laptop_catalog_item"],
  name: "Standard Laptop",
  shortDescription: "Request standard laptop",
  variables: {
    memory: SingleLineTextVariable({ question: "Memory Size", order: 100 }),
    storage: SingleLineTextVariable({ question: "Storage Size", order: 200 }),
    processor: SingleLineTextVariable({
      question: "Processor Type",
      order: 300
    })
  }
});
```

**Flow File - Basic Variable Population:**

```typescript
import { laptopCatalogItem } from "../catalogs/laptop-catalog";

// In flow:
wfa.action(
  action.core.getCatalogVariables,
  {
    $id: Now.ID["get_template_vars"],
    annotation: "Populate from standard laptop template"
  },
  {
    requested_item: wfa.dataPill(_params.trigger.request_item, "reference"),
    template_catalog_item: `${laptopCatalogItem}` // Reference CatalogItem with template literal
  }
);

// Variables from template are now populated on the request item
```

**Selective Variable Copy:**

```typescript
import { laptopCatalogItem } from "../catalogs/laptop-catalog";

// In flow:
wfa.action(
  action.core.getCatalogVariables,
  { $id: Now.ID["copy_specific_vars"] },
  {
    requested_item: wfa.dataPill(requestItem.record, "reference"),
    template_catalog_item: `${laptopCatalogItem}`,
    catalog_variables: [
      laptopCatalogItem.variables.memory, // Reference variable properties
      laptopCatalogItem.variables.storage,
      laptopCatalogItem.variables.processor
    ]
  }
);
```

### Important Notes

- **CatalogItem Import** - See submitCatalogItemRequest Important Notes above for CatalogItem import requirements
- **catalog_variables Format** - Use variable property references: `[catalogItem.variables.varName]` not string array `["varName"]`
- **No Return Value** - This action populates variables on the request item but doesn't return any outputs
- **Both Parameters Required** - Both `requested_item` and `template_catalog_item` are mandatory
- **Template Reference Table** - `template_catalog_item` references `st_sys_catalog_items_and_variable_sets` (supports both catalog items AND variable sets)
- **Variable Inheritance** - Variables from template are copied to the request item, maintaining their configuration
- **Selective Copy** - Use `catalog_variables` parameter to copy only specific variables instead of all
- **Existing Values** - Existing variable values on the request item may be overwritten by template values

---

## action.core.createCatalogTask

Create a Catalog Task on a Service Catalog Request Item [sc_req_item] with optional blocking behavior and variable population.

The task can be configured to be a blocking or non-blocking task. Pass Catalog Variables to the task to provide additional information or to allow the fulfiller to modify the variables. To pass variables from the Catalog Request Item, configure a template catalog item to populate the available catalog variables and select which ones will be applied to the task. Any changes to variables during task fulfillment will be updated on the Catalog Request Item.

### Input Parameters

| Parameter             | Type          | Default   | Mandatory | Description                                                       |
| --------------------- | ------------- | --------- | --------- | ----------------------------------------------------------------- |
| ah_requested_item     | reference     | -         | Yes       | sys_id of request item (sc_req_item)                              |
| ah_short_description  | string        | -         | No        | Task description (max 12000 chars)                                |
| ah_fields             | templatevalue | -         | No        | Additional field values in TemplateValue format (max 65000 chars) |
| ah_wait               | boolean       | true      | No        | Wait for task completion (default: true = blocking)               |
| ah_table_name         | string        | 'sc_task' | No        | Table name (readOnly, always 'sc_task')                           |
| template_catalog_item | reference     | -         | No        | Template catalog item (sc_cat_item) for variable population       |
| catalog_variables     | slushbucket   | -         | No        | Catalog variables to apply to task (max 12000 chars)              |

### Output Fields

| Field          | Type      | Description                              |
| -------------- | --------- | ---------------------------------------- |
| "Catalog Task" | reference | sys_id of created catalog task (sc_task) |

**Note:** Output field name contains a space. Access using bracket notation: `result["Catalog Task"]`

### Usage Examples

**Basic Catalog Task Creation:**

```typescript
const task = wfa.action(
  action.core.createCatalogTask,
  {
    $id: Now.ID["create_fulfillment_task"],
    annotation: "Create laptop fulfillment task"
  },
  {
    ah_requested_item: wfa.dataPill(
      _params.trigger.current.sys_id,
      "reference"
    ),
    ah_short_description: "Fulfill laptop request",
    ah_fields: TemplateValue({
      assignment_group: "IT Support Group sys_id",
      priority: "2"
    }),
    ah_wait: false // Non-blocking
  }
);

// Access output with bracket notation (note the space in field name)
// Use task["Catalog Task"] directly in subsequent actions:
// wfa.dataPill(task["Catalog Task"], "reference")
```

**Blocking Task (Wait for Completion):**

```typescript
const task = wfa.action(
  action.core.createCatalogTask,
  { $id: Now.ID["create_and_wait"] },
  {
    ah_requested_item: wfa.dataPill(request.requested_item, "reference"),
    ah_short_description: "Critical provisioning task",
    ah_wait: true // Flow pauses until task completes (default behavior)
  }
);

// Flow resumes here only after task is closed
```

**With Template Variable Population:**

```typescript
import { serverCatalogItem } from "../catalogs/server-catalog";

// In flow:
const task = wfa.action(
  action.core.createCatalogTask,
  { $id: Now.ID["create_with_vars"] },
  {
    ah_requested_item: wfa.dataPill(_params.trigger.request_item, "reference"),
    ah_short_description: "Configure server from template",
    template_catalog_item: `${serverCatalogItem}`, // Reference CatalogItem
    catalog_variables: [
      serverCatalogItem.variables.cpu_count, // Reference variable properties
      serverCatalogItem.variables.memory_size,
      serverCatalogItem.variables.disk_config
    ],
    ah_fields: TemplateValue({
      assignment_group: "Infrastructure Team sys_id",
      priority: "2"
    }),
    ah_wait: false
  }
);
```

**Using Task Output in Subsequent Actions:**

```typescript
const task = wfa.action(
  action.core.createCatalogTask,
  { $id: Now.ID["create_task"] },
  {
    ah_requested_item: wfa.dataPill(reqItem.sys_id, "reference"),
    ah_short_description: "Provision new hire equipment",
    ah_wait: false
  }
);

// Update request item with task reference
wfa.action(
  action.core.updateRecord,
  { $id: Now.ID["link_task"] },
  {
    table_name: "sc_req_item",
    record: wfa.dataPill(reqItem.sys_id, "reference"),
    values: TemplateValue({
      work_notes: `Fulfillment task created: ${wfa.dataPill(task["Catalog Task"], "reference")}`
    })
  }
);
```

### Important Notes

- **CatalogItem Import** - See submitCatalogItemRequest Important Notes above for CatalogItem import requirements
- **catalog_variables Format** - See getCatalogVariables Important Notes for variable reference format
- **⚠️ CRITICAL - Parameter Prefix:** ALL parameters use `ah_` prefix (not standard WFA pattern). This is unique to createCatalogTask.
- **⚠️ CRITICAL - Output Field Name:** Output field is `"Catalog Task"` with space - MUST use bracket notation: `task["Catalog Task"]`
- **Differs from createTask** - This action is catalog-specific (sc_task table only). Use `action.core.createTask` for other task types.
- **Blocking by Default** - `ah_wait` defaults to `true` (flow pauses until task completes). Set to `false` for non-blocking.
- **Table Name ReadOnly** - `ah_table_name` is always 'sc_task' and cannot be changed
- **TemplateValue Required** - `ah_fields` uses TemplateValue wrapper: `TemplateValue({ field: value })`, NOT plain string format
- **Variable Updates** - Changes to catalog variables during task fulfillment are reflected back on the request item
- **Template Variables** - Use `template_catalog_item` and `catalog_variables` together to populate task with specific variables from a template

---

## Integration Patterns

### Catalog Order Flow

**Catalog Definition File (catalogs/laptop-catalog.ts):**

```typescript
import { CatalogItem, SingleLineTextVariable } from "@servicenow/sdk/core";

export const laptopCatalogItem = CatalogItem({
  $id: Now.ID["laptop_catalog_item"],
  name: "Standard Laptop",
  shortDescription: "Request standard laptop",
  variables: {
    memory: SingleLineTextVariable({ question: "Memory Size", order: 100 }),
    storage: SingleLineTextVariable({ question: "Storage Size", order: 200 })
  }
});
```

**Flow File - Complete workflow from trigger to request submission:**

```typescript
import { laptopCatalogItem } from "../catalogs/laptop-catalog";

Flow(
  { $id: Now.ID["auto_provision"], name: "Auto Provision Hardware" },

  wfa.trigger(
    trigger.application.serviceCatalog,
    { $id: Now.ID["catalog_trigger"] },
    { run_flow_in: "background" }
  ),

  _params => {
    const request = wfa.action(
      action.core.submitCatalogItemRequest,
      { $id: Now.ID["submit"] },
      {
        catalog_item: `${laptopCatalogItem}`,
        catalog_item_inputs: "memory=16GB^storage=512GB",
        sysparm_requested_for: wfa.dataPill(
          _params.trigger.request_item.requested_for,
          "reference"
        )
      }
    );

    wfa.flowLogic.if(
      {
        $id: Now.ID["success"],
        condition: `${wfa.dataPill(request.status, "string")}=0`
      },
      () => {
        wfa.action(
          action.core.updateRecord,
          { $id: Now.ID["update"] },
          {
            table_name: "sc_req_item",
            record: wfa.dataPill(_params.trigger.request_item, "reference"),
            values: TemplateValue({ state: 2 })
          }
        );
      }
    );
  }
);
```

### Approval and Fulfillment Flow

**Flow File - Multi-step workflow with approval gate and task creation:**

```typescript
import { laptopCatalogItem } from "../catalogs/laptop-catalog";

Flow(
  {
    $id: Now.ID["catalog_approval"],
    name: "Catalog Request Approval and Fulfillment"
  },

  wfa.trigger(
    trigger.application.serviceCatalog,
    { $id: Now.ID["catalog_trigger"] },
    { run_flow_in: "background" }
  ),

  _params => {
    const approval = wfa.action(
      action.core.askForApproval,
      { $id: Now.ID["request_approval"] },
      {
        record: wfa.dataPill(_params.trigger.request_item, "reference"),
        table_name: "sc_req_item",
        approval_reason: "Manager approval required"
      }
    );

    wfa.flowLogic.if(
      {
        $id: Now.ID["approved"],
        condition: `${wfa.dataPill(approval.approval_state, "string")}=approved`
      },
      () => {
        const task = wfa.action(
          action.core.createCatalogTask,
          { $id: Now.ID["create_task"] },
          {
            ah_requested_item: wfa.dataPill(
              _params.trigger.request_item,
              "reference"
            ),
            ah_short_description: "Fulfill approved request",
            template_catalog_item: `${laptopCatalogItem}`,
            catalog_variables: [
              laptopCatalogItem.variables.memory,
              laptopCatalogItem.variables.storage
            ],
            ah_fields: TemplateValue({
              assignment_group: "Procurement Team sys_id"
            }),
            ah_wait: false
          }
        );

        // Note: Use bracket notation for output field
        // Access as: task["Catalog Task"]
      }
    );
  }
);
```

---

## Common Gotchas

### 1. Status Code Checking

**❌ WRONG - Not checking status:**

```typescript
const request = wfa.action(action.core.submitCatalogItemRequest, ...);
// Directly using requested_item without checking status
wfa.dataPill(request.requested_item, "reference");  // May be empty if status = 1 or 2!
```

**✅ CORRECT - Always check status first:**

```typescript
const request = wfa.action(action.core.submitCatalogItemRequest, ...);
wfa.flowLogic.if(
  { condition: `${wfa.dataPill(request.status, "string")}=0` },
  () => {
    // Safe to use requested_item here
    wfa.dataPill(request.requested_item, "reference");
  }
);
```

### 2. createCatalogTask Output Access

**❌ WRONG - Using dot notation:**

```typescript
const task = wfa.action(action.core.createCatalogTask, ...);
wfa.dataPill(task.CatalogTask, "reference");  // FAILS - field name has space!
```

**✅ CORRECT - Using bracket notation:**

```typescript
const task = wfa.action(action.core.createCatalogTask, ...);
wfa.dataPill(task["Catalog Task"], "reference");  // CORRECT - brackets handle space
```

### 3. Variable Format

**❌ WRONG - Incorrect delimiter:**

```typescript
catalog_item_inputs: "memory=16GB,storage=512GB"; // Wrong delimiter!
```

**✅ CORRECT - Using ^ delimiter:**

```typescript
catalog_item_inputs: "memory=16GB^storage=512GB"; // Correct encoded query format
```

### 4. ah_fields Format

**❌ WRONG - Plain string format:**

```typescript
ah_fields: "assignment_group=IT Support^priority=2"; // Wrong format!
```

**✅ CORRECT - TemplateValue format:**

```typescript
ah_fields: TemplateValue({
  assignment_group: "IT Support Group sys_id",
  priority: "2"
}); // Correct TemplateValue wrapper
```
