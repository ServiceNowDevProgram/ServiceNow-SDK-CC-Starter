# WFA_FLOW_TRIGGER_APPLICATION

The Application Triggers API reference provides complete technical specifications for application-specific flow triggers. This document contains trigger signatures, configuration parameters, output fields, and syntax examples for Inbound Email, SLA Task, Service Catalog, Knowledge Management, and Remote Table Query triggers. For conceptual guidance, when-to-use recommendations, and implementation patterns, refer to skill references.

## Trigger Usage Pattern

All application triggers follow the same usage pattern:

```typescript
wfa.trigger(
  trigger.application.<triggerType>,
  { $id: Now.ID['trigger_id'] },
  {
    // application-specific configuration parameters
  }
)
```

---

## trigger.application.inboundEmail

Activates when an inbound email matches specified conditions.

### Configuration Parameters

| Parameter                 | Type    | Default | Mandatory | Description                                                      |
| ------------------------- | ------- | ------- | --------- | ---------------------------------------------------------------- |
| email_conditions          | string  | -       | No        | Encoded query conditions to filter emails                        |
| order                     | integer | -       | No        | Priority order for this trigger (lower values = higher priority) |
| stop_condition_evaluation | boolean | true    | No        | When true, stops processing after first matching trigger         |
| target_table              | string  | -       | No        | Table associated with reply record                               |

**Tools needed:** Email conditions follow ServiceNow encoded query format. Use `get_table_schema` with table `sys_email` to get available email fields.

### Output Fields

| Field             | Type             | Description                                  |
| ----------------- | ---------------- | -------------------------------------------- |
| inbound_email     | reference        | Reference to sys_email record                |
| target_table_name | string           | Table name of the target record              |
| body_text         | string_full_utf8 | Email body content                           |
| subject           | string_full_utf8 | Email subject line                           |
| user              | reference        | User who sent the email (sys_user reference) |
| target_record     | reference        | Related target record if applicable          |
| from_address      | string_full_utf8 | Sender email address                         |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.inbound_email, "reference");
  wfa.dataPill(_params.trigger.subject, "string_full_utf8");
  wfa.dataPill(_params.trigger.body_text, "string_full_utf8");
  wfa.dataPill(_params.trigger.from_address, "string_full_utf8");
  wfa.dataPill(_params.trigger.user, "reference");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["process_support_emails"],
    name: "Process Support Emails"
  },

  wfa.trigger(
    trigger.application.inboundEmail,
    { $id: Now.ID["email_trigger"] },
    {
      email_conditions: "subjectCONTAINSsupport^ORsubjectCONTAINShelp",
      order: 100,
      stop_condition_evaluation: true,
      target_table: "incident"
    }
  ),

  _params => {
    // ✅ Create incident from email - use data pills directly
    const newIncident = wfa.action(
      action.core.createRecord,
      { $id: Now.ID["create_incident"] },
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
          caller_id: wfa.dataPill(_params.trigger.user, "reference"),
          priority: 3,
          state: 1
        })
      }
    );

    wfa.action(
      action.core.log,
      { $id: Now.ID["log_creation"] },
      {
        log_level: "info",
        log_message: `Created incident from email: ${wfa.dataPill(_params.trigger.from_address, "string")}`
      }
    );

    // Send confirmation email
    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["send_confirmation"] },
      {
        ah_to: `${wfa.dataPill(_params.trigger.from_address, "string")}`,
        ah_subject: `Re: ${wfa.dataPill(_params.trigger.subject, "string")}`,
        ah_body:
          "Thank you for contacting support. Your incident has been created.",
        record: wfa.dataPill(newIncident.record, "reference"),
        table_name: "incident"
      }
    );
  }
);
```

---

## trigger.application.slaTask

Activates when an SLA task event occurs (e.g., SLA breach, SLA warning).

### Configuration Parameters

This trigger has no input configuration parameters. It automatically activates for SLA task events.

### Output Fields

| Field           | Type      | Description                  |
| --------------- | --------- | ---------------------------- |
| task_sla_record | reference | Reference to task_sla record |
| sla_flow_inputs | object    | SLA-specific input data      |

**sla_flow_inputs Structure:**

```typescript
{
  duration: string,                    // SLA duration
  relative_duration_works_on: string,  // Schedule for duration calculation
  is_repair: boolean,                  // Whether this is a repair SLA
  duration_type: string,               // Type of duration calculation
  name: string                         // SLA definition name
}
```

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.task_sla_record, "reference");
  wfa.dataPill(_params.trigger.sla_flow_inputs.name, "string");
  wfa.dataPill(_params.trigger.sla_flow_inputs.duration, "string");
  wfa.dataPill(_params.trigger.sla_flow_inputs.is_repair, "boolean");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["sla_breach_handler"],
    name: "Handle SLA Breach Events"
  },

  wfa.trigger(trigger.application.slaTask, { $id: Now.ID["sla_trigger"] }, {}),

  _params => {
    // ✅ Log SLA event - use data pills directly in template literal
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_sla_event"] },
      {
        log_level: "warn",
        log_message: `SLA event triggered for ${wfa.dataPill(_params.trigger.sla_flow_inputs.name, "string")} with duration ${wfa.dataPill(_params.trigger.sla_flow_inputs.duration, "string")}`
      }
    );

    // Lookup the SLA record details - use data pill directly
    const slaDetails = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["get_sla_details"] },
      {
        table: "task_sla",
        conditions: `sys_id=${wfa.dataPill(_params.trigger.task_sla_record, "reference")}`,
        max_results: 1
      }
    );

    // Send notification - use data pill directly
    wfa.action(
      action.core.sendNotification,
      { $id: Now.ID["notify_sla"] },
      {
        notification_name: "sla_breach_notification",
        record: wfa.dataPill(_params.trigger.task_sla_record, "reference"),
        table: "task_sla"
      }
    );
  }
);
```

### Complete SLA Monitoring Pattern

This example demonstrates integrating the SLA Task trigger with the `action.core.slaPercentageTimer` action for progressive monitoring at multiple SLA milestones.

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["sla_progress_monitor"],
    name: "SLA Progress Monitoring with Escalation"
  },

  wfa.trigger(trigger.application.slaTask, { $id: Now.ID["sla_trigger"] }, {}),

  _params => {
    // ✅ Log SLA start - use data pills directly in template literal
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_sla_start"] },
      {
        log_level: "info",
        log_message: `SLA monitoring started: ${wfa.dataPill(_params.trigger.sla_flow_inputs.name, "string")}, Duration: ${wfa.dataPill(_params.trigger.sla_flow_inputs.duration, "string")} ${wfa.dataPill(_params.trigger.sla_flow_inputs.duration_type, "string")}`
      }
    );

    // Monitor at 25% elapsed
    wfa.action(
      action.core.slaPercentageTimer,
      { $id: Now.ID["wait_25"] },
      { percentage: 25 }
    );
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_25"] },
      {
        log_level: "info",
        log_message: `${wfa.dataPill(_params.trigger.sla_flow_inputs.name, "string")}: 25% of SLA time elapsed`
      }
    );

    // Monitor at 50% elapsed - Send early warning
    wfa.action(
      action.core.slaPercentageTimer,
      { $id: Now.ID["wait_50"] },
      { percentage: 50 }
    );
    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["notify_50"] },
      {
        ah_to: "team@example.com",
        ah_subject: `SLA Warning: ${wfa.dataPill(_params.trigger.sla_flow_inputs.name, "string")} - 50% Elapsed`,
        ah_body: `SLA ${slaName} is halfway elapsed. Please review and take action if needed.`
      }
    );

    // Monitor at 75% elapsed - Escalate to management
    wfa.action(
      action.core.slaPercentageTimer,
      { $id: Now.ID["wait_75"] },
      { percentage: 75 }
    );
    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["notify_75"] },
      {
        ah_to: "management@example.com",
        ah_subject: `SLA CRITICAL: ${slaName} - 75% Elapsed`,
        ah_body: `SLA ${slaName} is at 75%. Immediate attention required.`
      }
    );

    // Monitor at 100% - SLA breached
    wfa.action(
      action.core.slaPercentageTimer,
      { $id: Now.ID["wait_100"] },
      { percentage: 100 }
    );
    wfa.action(
      action.core.sendNotification,
      { $id: Now.ID["breach_notification"] },
      {
        notification: "sla_breach_notification",
        record: wfa.dataPill(_params.trigger.task_sla_record, "reference"),
        table_name: "task_sla"
      }
    );
  }
);
```

---

## trigger.application.serviceCatalog

Activates when a Service Catalog request item workflow needs to be processed. Used for automating fulfillment, approval, and notification workflows for catalog requests.

### Configuration Parameters

| Parameter   | Type   | Default      | Mandatory | Description                                                                             |
| ----------- | ------ | ------------ | --------- | --------------------------------------------------------------------------------------- |
| run_flow_in | string | "background" | No        | Execution context: "any" (system chooses), "background" (async), or "foreground" (sync) |

**Execution Context Guidance:**

- `"background"` (recommended): Asynchronous execution, does not block user request submission
- `"foreground"`: Synchronous execution, blocks user until flow completes (use sparingly)
- `"any"`: System chooses context (default behavior)

### Output Fields

| Field               | Type      | Description                                            |
| ------------------- | --------- | ------------------------------------------------------ |
| request_item        | reference | Reference to sc_req_item record (catalog request item) |
| table_name          | string    | Table name, always "sc_req_item"                       |
| run_start_time      | datetime  | When the trigger started execution                     |
| run_start_date_time | datetime  | Alternative field for run start time                   |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.request_item, "reference");
  wfa.dataPill(_params.trigger.table_name, "string");
  wfa.dataPill(_params.trigger.run_start_date_time, "datetime");

  // Dot-walk to access request item fields
  wfa.dataPill(_params.trigger.request_item.short_description, "string");
  wfa.dataPill(_params.trigger.request_item.requested_for, "reference");
  wfa.dataPill(_params.trigger.request_item.cat_item, "reference");
};
```

### Example: Basic Catalog Task Creation

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["laptop_fulfillment"],
    name: "Laptop Fulfillment Workflow"
  },

  wfa.trigger(
    trigger.application.serviceCatalog,
    { $id: Now.ID["catalog_trigger"] },
    {
      run_flow_in: "background"
    }
  ),

  _params => {
    // Log the catalog request
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_request"] },
      {
        log_level: "info",
        log_message: `Processing catalog request: ${wfa.dataPill(_params.trigger.request_item, "string")}`
      }
    );

    // Create fulfillment task
    wfa.action(
      action.core.createCatalogTask,
      { $id: Now.ID["create_task"] },
      {
        ah_requested_item: wfa.dataPill(
          _params.trigger.request_item,
          "reference"
        ),
        ah_short_description: "Fulfill laptop request",
        ah_fields: TemplateValue({
          assignment_group: "Hardware Fulfillment Team",
          priority: "3"
        })
      }
    );
  }
);
```

### Complete Pattern: Service Catalog with getCatalogVariables

This example demonstrates using the Service Catalog trigger with `getCatalogVariables` to retrieve user selections and drive conditional workflow logic.

```typescript
import "@servicenow/sdk/global";
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";
import { CatalogItem, MultipleChoiceVariable } from "@servicenow/sdk/core";

// Define catalog item with variables
const laptopCatalogItem = CatalogItem({
  $id: Now.ID["laptop_catalog_item"],
  name: "Corporate Laptop Request",
  shortDescription: "Request a laptop for corporate use",
  description:
    "Use this form to request a laptop with your preferred specifications.",
  variables: {
    memory: MultipleChoiceVariable({
      question: "Memory (RAM)",
      order: 10,
      choices: {
        "8GB": { label: "8GB", sequence: 0 },
        "16GB": { label: "16GB", sequence: 1 },
        "32GB": { label: "32GB", sequence: 2 }
      }
    }),
    storage: MultipleChoiceVariable({
      question: "Storage",
      order: 20,
      choices: {
        "256GB": { label: "256GB SSD", sequence: 0 },
        "512GB": { label: "512GB SSD", sequence: 1 },
        "1TB": { label: "1TB SSD", sequence: 2 }
      }
    }),
    operating_system: MultipleChoiceVariable({
      question: "Operating System",
      order: 30,
      choices: {
        Windows: { label: "Windows 11 Pro", sequence: 0 },
        macOS: { label: "macOS", sequence: 1 }
      }
    })
  }
});

Flow(
  {
    $id: Now.ID["laptop_config_workflow"],
    name: "Laptop Configuration Workflow"
  },

  wfa.trigger(
    trigger.application.serviceCatalog,
    { $id: Now.ID["catalog_trigger"] },
    {
      run_flow_in: "background"
    }
  ),

  _params => {
    // Step 1: Get catalog variables from request item
    const catalogVars = wfa.action(
      action.core.getCatalogVariables,
      {
        $id: Now.ID["get_variables"],
        annotation: "Retrieve laptop configuration variables"
      },
      {
        requested_item: wfa.dataPill(_params.trigger.request_item, "reference"),
        template_catalog_item: `${laptopCatalogItem}`,
        catalog_variables: [
          laptopCatalogItem.variables.memory,
          laptopCatalogItem.variables.storage,
          laptopCatalogItem.variables.operating_system
        ]
      }
    );

    // Step 2: Log retrieved variable values
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_variables"] },
      {
        log_level: "info",
        log_message: `Laptop config - Memory: ${wfa.dataPill(catalogVars.memory, "string")}, Storage: ${wfa.dataPill(catalogVars.storage, "string")}, OS: ${wfa.dataPill(catalogVars.operating_system, "string")}`
      }
    );

    // Step 3: Route based on catalog variables
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_premium_config"],
        condition: `${wfa.dataPill(catalogVars.memory, "string")}=32GB`,
        annotation: "Check if premium configuration requested"
      },
      () => {
        // Premium config - route to specialized team
        wfa.action(
          action.core.createCatalogTask,
          { $id: Now.ID["create_premium_task"] },
          {
            ah_requested_item: wfa.dataPill(
              _params.trigger.request_item,
              "reference"
            ),
            ah_short_description: `Fulfill premium laptop - ${wfa.dataPill(catalogVars.memory, "string")} RAM, ${wfa.dataPill(catalogVars.storage, "string")} storage`,
            ah_fields: TemplateValue({
              assignment_group: "Premium Hardware Team",
              priority: "2",
              work_notes: `High-spec configuration: ${wfa.dataPill(catalogVars.operating_system, "string")}`
            })
          }
        );
      }
    );
    wfa.flowLogic.else(
      {
        $id: Now.ID["other_config"]
      },
      () => {
        // Standard config - route to standard team
        wfa.action(
          action.core.createCatalogTask,
          { $id: Now.ID["create_standard_task"] },
          {
            ah_requested_item: wfa.dataPill(
              _params.trigger.request_item,
              "reference"
            ),
            ah_short_description: "Fulfill standard laptop request",
            ah_fields: TemplateValue({
              assignment_group: "Standard Hardware Team",
              priority: "3"
            })
          }
        );
      }
    );
  }
);
```

**Key Integration Points:**

- **getCatalogVariables Action**: Retrieves variable values from request item
- **CatalogItem Definition**: Provides type-safe variable references
- **Variable-Based Routing**: Use catalog variables in conditions to route workflows
- **Template String Reference**: Use `${catalogItem}` to reference catalog item
- **Variable Array**: Pass `catalogItem.variables.variableName` in `catalog_variables` array

---

## trigger.application.knowledgeManagement

Activates when knowledge management events occur (e.g., knowledge article published, retired).

### Configuration Parameters

This trigger has no input configuration parameters. It automatically activates for knowledge management events.

### Output Fields

| Field               | Type            | Description                          |
| ------------------- | --------------- | ------------------------------------ |
| knowledge           | reference       | Reference to kb_knowledge record     |
| table_name          | string          | Table name (default: 'kb_knowledge') |
| run_start_time      | glide_date_time | Flow execution start time            |
| run_start_date_time | glide_date_time | Flow execution start date/time       |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.knowledge, "reference");
  wfa.dataPill(_params.trigger.table_name, "string");
  wfa.dataPill(_params.trigger.run_start_date_time, "glide_date_time");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["knowledge_article_published"],
    name: "Notify on Knowledge Article Publication"
  },

  wfa.trigger(
    trigger.application.knowledgeManagement,
    { $id: Now.ID["knowledge_trigger"] },
    {}
  ),

  _params => {
    // ✅ Lookup knowledge article details - use data pill directly
    const article = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["get_article"] },
      {
        table: "kb_knowledge",
        conditions: `sys_id=${wfa.dataPill(_params.trigger.knowledge, "reference")}`,
        max_results: 1
      }
    );

    // Log publication - use data pill directly in template literal
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_publication"] },
      {
        log_level: "info",
        log_message: `Knowledge article published: ${wfa.dataPill(article.Record.short_description, "string")}`
      }
    );

    // Notify knowledge managers - use data pills directly
    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["notify_managers"] },
      {
        ah_to: "knowledge-managers@company.com",
        ah_subject: `New Knowledge Article: ${wfa.dataPill(article.Record.short_description, "string")}`,
        ah_body: "A new knowledge article has been published.",
        record: wfa.dataPill(_params.trigger.knowledge, "reference"),
        table_name: "kb_knowledge"
      }
    );
  }
);
```

---

## trigger.application.remoteTableQuery

Activates when a remote table query is executed.

### Configuration Parameters

| Parameter | Type   | Default | Mandatory | Description                                |
| --------- | ------ | ------- | --------- | ------------------------------------------ |
| u_table   | string | -       | No        | Remote table name (from sys_script_vtable) |

**Tools needed:** Use `run_query` on `sys_script_vtable` table to find available remote tables.

### Output Fields

| Field            | Type   | Description                          |
| ---------------- | ------ | ------------------------------------ |
| table_name       | string | Remote table name                    |
| query_parameters | object | Name-value pairs of query parameters |
| query_id         | string | Unique identifier for this query     |

**Access Pattern:**

```typescript
_params => {
  wfa.dataPill(_params.trigger.table_name, "string");
  wfa.dataPill(_params.trigger.query_id, "string");
};
```

### Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["process_remote_query"],
    name: "Process Remote Table Queries"
  },

  wfa.trigger(
    trigger.application.remoteTableQuery,
    { $id: Now.ID["remote_query_trigger"] },
    {
      u_table: "x_custom_remote_table"
    }
  ),

  _params => {
    // ✅ Log remote query - use data pills directly in template literal
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_query"] },
      {
        log_level: "info",
        log_message: `Remote table query executed: ${wfa.dataPill(_params.trigger.table_name, "string")}, Query ID: ${wfa.dataPill(_params.trigger.query_id, "string")}`
      }
    );

    // Perform any post-processing based on query - use data pills directly
    wfa.action(
      action.core.createRecord,
      { $id: Now.ID["log_query_execution"] },
      {
        table_name: "u_query_log",
        values: TemplateValue({
          u_table_name: wfa.dataPill(_params.trigger.table_name, "string"),
          u_query_id: wfa.dataPill(_params.trigger.query_id, "string"),
          u_executed_at: wfa.dataPill(
            _params.trigger.run_start_date_time,
            "datetime"
          )
        })
      }
    );
  }
);
```

---

## Notes

- Use `get_table_schema` tool with table `sys_email` to get available email fields for `inboundEmail` trigger conditions
- Use `run_query` tool on `sys_script_vtable` table to find available remote tables for `remoteTableQuery` trigger
- The `inboundEmail` trigger's `order` parameter determines processing priority when multiple email triggers exist
- The `slaTask` trigger provides SLA-specific metadata in `sla_flow_inputs` object
- The `knowledgeManagement` trigger fires for knowledge article lifecycle events (publish, retire, etc.)
- The `remoteTableQuery` trigger activates when external systems query ServiceNow via remote tables
- For `inboundEmail` trigger, set `stop_condition_evaluation: true` to prevent multiple flows from processing the same email
