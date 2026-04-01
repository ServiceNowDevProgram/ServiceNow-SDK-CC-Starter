# Catalog Approval and Fulfillment Pattern

Multi-step catalog workflow with conditional approval based on catalog variables and automated fulfillment task creation.

## Table of Contents

- [When to Use](#when-to-use)
- [Pattern Structure](#pattern-structure)
- [Complete Example](#complete-example)
- [Key Considerations](#key-considerations)
- [Variations](#variations)
  - [Variable-Based Conditional Approval Patterns](#variable-based-conditional-approval-patterns)
  - [Multi-Level Approval](#multi-level-approval)
  - [Parallel Approval (Legal + Finance)](#parallel-approval-legal--finance)
  - [Task with Template Variables](#task-with-template-variables)
  - [Conditional Task Assignment](#conditional-task-assignment)
- [Common Mistakes](#common-mistakes)
- [Related Patterns](#related-patterns)
- [Next Steps](#next-steps)

## When to Use

- Conditional approval workflows based on catalog variable values
- High-value purchases requiring approval ($1K+)
- Multi-stage provisioning workflows with approval gates
- Variable-based routing (item type, cost, urgency, department)
- Auto-approve low-risk requests, require approval for high-risk
- Automated fulfillment after approval

## Pattern Structure

```
Trigger → getCatalogVariables → Check Variable Values →
  If Approval Required: Lookup Manager → Request Approval → Check Approval State →
    Approved → Create Fulfillment Task → Notify Requester
    Rejected → Close Request → Notify Requester
  If Auto-Approve: Create Fulfillment Task → Notify Requester
```

## Complete Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";
import { equipmentRequestCatalogItem } from "../catalog-items/equipment-request";

Flow(
  {
    $id: Now.ID["conditional_catalog_approval"],
    name: "Conditional Catalog Approval and Fulfillment",
    description:
      "Approval required for high-cost or hardware items; auto-approve low-cost supplies"
  },

  wfa.trigger(
    trigger.application.serviceCatalog,
    { $id: Now.ID["catalog_trigger"] },
    {
      run_flow_in: "background"
    }
  ),

  _params => {
    // Step 1: Get catalog variables to determine approval requirements
    // equipmentRequestCatalogItem has variables: item_type, estimated_cost, urgency
    const catalogVars = wfa.action(
      action.core.getCatalogVariables,
      {
        $id: Now.ID["get_variables"],
        annotation: "Get catalog item variables"
      },
      {
        requested_item: wfa.dataPill(_params.trigger.request_item, "reference"),
        template_catalog_item: `${equipmentRequestCatalogItem}`,
        catalog_variables: [
          equipmentRequestCatalogItem.variables.item_type,
          equipmentRequestCatalogItem.variables.estimated_cost,
          equipmentRequestCatalogItem.variables.urgency
        ]
      }
    );

    // Step 2: Check if approval is required based on catalog variables
    // Approval required if: estimated_cost > $1000 OR item_type = "hardware"
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_needs_approval"],
        condition: `${wfa.dataPill(catalogVars.estimated_cost, "string")}>1000^OR${wfa.dataPill(catalogVars.item_type, "string")}=hardware`,
        annotation: "Check if approval required (cost > $1000 OR hardware)"
      },
      () => {
        // APPROVAL REQUIRED PATH

        // Step 2a: Look up manager for approval
        const manager = wfa.action(
          action.core.lookUpRecord,
          {
            $id: Now.ID["find_manager"],
            annotation: "Find requester's manager"
          },
          {
            table: "sys_user",
            conditions: `sys_id=${wfa.dataPill(_params.trigger.request_item.opened_by.manager, "reference")}`
          }
        );

        // Step 2b: Request manager approval with 2-day deadline
        const approval = wfa.action(
          action.core.askForApproval,
          {
            $id: Now.ID["request_approval"],
            annotation: "Request manager approval"
          },
          {
            record: wfa.dataPill(_params.trigger.request_item, "reference"),
            table_name: "sc_req_item",
            users: [wfa.dataPill(manager.Record, "reference")],
            approval_rules: {
              rule_type: "Any",
              num_approvals: 1
            },
            due_date: wfa.approvalDueDate({
              action: "approve",
              dateType: "relative",
              date: wfa.dataPill(
                _params.trigger.request_item.sys_created_on,
                "glide_date_time"
              ),
              duration: 2,
              durationType: "days"
            })
          }
        );

        // Step 2c: Handle approval result
        wfa.flowLogic.if(
          {
            $id: Now.ID["check_approved"],
            condition: `${wfa.dataPill(approval.approval_state, "string")}=approved`,
            annotation: "Check if approved"
          },
          () => {
            // APPROVED - Create fulfillment task
            const task = wfa.action(
              action.core.createCatalogTask,
              {
                $id: Now.ID["create_fulfillment_approved"],
                annotation: "Create fulfillment task after approval"
              },
              {
                ah_requested_item: wfa.dataPill(
                  _params.trigger.request_item.sys_id,
                  "reference"
                ),
                ah_short_description: `Fulfill: ${wfa.dataPill(_params.trigger.request_item.short_description, "string")}`,
                ah_fields: TemplateValue({
                  assignment_group: "<procurement_group_sys_id>",
                  priority:
                    wfa.dataPill(catalogVars.urgency, "string") === "high"
                      ? "1"
                      : "2",
                  work_notes: "Manager approved - proceeding with fulfillment"
                }),
                ah_wait: false
              }
            );

            // Update request with task reference
            wfa.action(
              action.core.updateRecord,
              { $id: Now.ID["update_approved"] },
              {
                table_name: "sc_req_item",
                record: wfa.dataPill(_params.trigger.request_item, "reference"),
                values: TemplateValue({
                  work_notes: `Fulfillment task created: ${wfa.dataPill(task["Catalog Task"], "reference")}`,
                  state: 2 // Work in Progress
                })
              }
            );

            // Notify requester
            wfa.action(
              action.core.sendEmail,
              { $id: Now.ID["notify_approved"] },
              {
                ah_to: wfa.dataPill(
                  _params.trigger.request_item.opened_by.email,
                  "string"
                ),
                ah_subject: "Your equipment request has been approved",
                ah_body:
                  "Your request has been approved by your manager and is now being fulfilled.",
                record: wfa.dataPill(_params.trigger.request_item, "reference"),
                table_name: "sc_req_item"
              }
            );
          }
        );
        wfa.flowLogic.elseIf(
          {
            $id: Now.ID["check_rejected"],
            condition: `${wfa.dataPill(approval.approval_state, "string")}=rejected`,
            annotation: "Handle rejection"
          },
          () => {
            // REJECTED - Close request
            wfa.action(
              action.core.updateRecord,
              { $id: Now.ID["reject_request"] },
              {
                table_name: "sc_req_item",
                record: wfa.dataPill(_params.trigger.request_item, "reference"),
                values: TemplateValue({
                  state: 4, // Closed Incomplete
                  work_notes:
                    "Request rejected by manager - budget not approved",
                  active: false
                })
              }
            );

            // Notify requester of rejection
            wfa.action(
              action.core.sendEmail,
              { $id: Now.ID["notify_rejected"] },
              {
                ah_to: wfa.dataPill(
                  _params.trigger.request_item.opened_by.email,
                  "string"
                ),
                ah_subject: "Your equipment request was not approved",
                ah_body:
                  "Your request did not receive approval. Please contact your manager for details.",
                record: wfa.dataPill(_params.trigger.request_item, "reference"),
                table_name: "sc_req_item"
              }
            );
          }
        );
        wfa.flowLogic.else({ $id: Now.ID["Other_approval_state"] }, () => {
          // Other approval state - Log for investigation
          wfa.action(
            action.core.log,
            { $id: Now.ID["log_approval_state"] },
            {
              log_level: "info",
              log_message: `Approval ended with state: ${wfa.dataPill(approval.approval_state, "string")}`
            }
          );
        });
      }
    );
    wfa.flowLogic.else({ $id: Now.ID["auto_approve"] }, () => {
      // AUTO-APPROVE PATH - Low cost, non-hardware items

      // Create fulfillment task immediately (no approval needed)
      const task = wfa.action(
        action.core.createCatalogTask,
        {
          $id: Now.ID["create_fulfillment_auto"],
          annotation: "Create fulfillment task (auto-approved)"
        },
        {
          ah_requested_item: wfa.dataPill(
            _params.trigger.request_item.sys_id,
            "reference"
          ),
          ah_short_description: `Fulfill: ${wfa.dataPill(_params.trigger.request_item.short_description, "string")}`,
          ah_fields: TemplateValue({
            assignment_group: "<office_supplies_group_sys_id>",
            priority: "3",
            work_notes: "Auto-approved - low cost supplies request"
          }),
          ah_wait: false
        }
      );

      // Update request with task reference
      wfa.action(
        action.core.updateRecord,
        { $id: Now.ID["update_auto_approved"] },
        {
          table_name: "sc_req_item",
          record: wfa.dataPill(_params.trigger.request_item, "reference"),
          values: TemplateValue({
            work_notes: `Auto-approved. Fulfillment task: ${wfa.dataPill(task["Catalog Task"], "reference")}`,
            state: 2 // Work in Progress
          })
        }
      );

      // Notify requester
      wfa.action(
        action.core.sendEmail,
        { $id: Now.ID["notify_auto_approved"] },
        {
          ah_to: wfa.dataPill(
            _params.trigger.request_item.opened_by.email,
            "string"
          ),
          ah_subject: "Your equipment request is being processed",
          ah_body:
            "Your request has been received and is being fulfilled by the supplies team.",
          record: wfa.dataPill(_params.trigger.request_item, "reference"),
          table_name: "sc_req_item"
        }
      );
    });
  }
);
```

## Key Considerations

1. **Catalog Variables** - Use `getCatalogVariables` to retrieve user selections from catalog items
2. **Import CatalogItem** - Import existing catalog item definitions to access type-safe variables
3. **Variable-Based Routing** - Use catalog variable values in conditions to determine workflow paths
4. **Conditional Approval** - Gate approval based on business rules (cost thresholds, item types, etc.)
5. **Task Creation** - `createCatalogTask` links fulfillment to request item
6. **⚠️ Bracket Notation** - CRITICAL: Access task output via `task["Catalog Task"]` (note the space!)
7. **Approval States** - Handle approved, rejected, AND other states (cancelled, not_required)
8. **User Notifications** - Keep stakeholders informed at each stage (approval, fulfillment, rejection)
9. **Work Notes** - Document workflow progress in request record for audit trail
10. **Non-Blocking Tasks** - Set `ah_wait: false` for better performance (default is true!)

## Variations

### Variable-Based Conditional Approval Patterns

```typescript
// Pattern 1: Approval based on multiple variable conditions (AND/OR logic)
const catalogVars = wfa.action(action.core.getCatalogVariables, ...);

// Require approval if: (cost > $5000 AND urgency = "high") OR department = "legal"
wfa.flowLogic.if(
  {
    $id: Now.ID["complex_approval_check"],
    condition: `${wfa.dataPill(catalogVars.estimated_cost, "string")}>5000^${wfa.dataPill(catalogVars.urgency, "string")}=high^OR${wfa.dataPill(catalogVars.department, "string")}=legal`
  },
  () => {
    // Request approval...
  }
);

// Pattern 2: Different approval tiers based on variable values
wfa.flowLogic.if(
    {
      $id: Now.ID["check_critical"],
      condition: `${wfa.dataPill(catalogVars.urgency, "string")}=critical^${wfa.dataPill(catalogVars.estimated_cost, "string")}>10000`
    },
    () => {
      // Critical + expensive → Require VP approval
      const vpApproval = wfa.action(action.core.askForApproval, {
        users: [wfa.dataPill(vp.Record, "reference")]
      });
    }
  )
  wfa.flowLogic.elseIf(
    {
      $id: Now.ID["check_standard"],
      condition: `${wfa.dataPill(catalogVars.estimated_cost, "string")}>1000`
    },
    () => {
      // Standard cost → Manager approval
      const managerApproval = wfa.action(action.core.askForApproval, {
        users: [wfa.dataPill(manager.Record, "reference")]
      });
    }
  )
  wfa.flowLogic.else({ $id: Now.ID["auto_approve"] },() => {
    // Low cost → Auto-approve
  });

// Pattern 3: Route to different approvers based on item type
wfa.flowLogic.if(
    {
      $id: Now.ID["check_software"],
      condition: `${wfa.dataPill(catalogVars.item_type, "string")}=software`
    },
    () => {
      // Software → IT Director approval
      const itApproval = wfa.action(action.core.askForApproval, {
        users: [wfa.dataPill(itDirector.Record, "reference")]
      });
    }
  )
  wfa.flowLogic.elseIf(
    {
      $id: Now.ID["check_hardware"],
      condition: `${wfa.dataPill(catalogVars.item_type, "string")}=hardware`
    },
    () => {
      // Hardware → Procurement Manager approval
      const procurementApproval = wfa.action(action.core.askForApproval, {
        users: [wfa.dataPill(procurementManager.Record, "reference")]
      });
    }
  );
```

### Multi-Level Approval

```typescript
// First approval - Manager
const managerApproval = wfa.action(
  action.core.askForApproval,
  { $id: Now.ID["manager_approval"] },
  {
    record: wfa.dataPill(_params.trigger.request_item, "reference"),
    table_name: "sc_req_item",
    users: [wfa.dataPill(manager.Record, "reference")],
    approval_rules: { rule_type: "Any", num_approvals: 1 }
  }
);

wfa.flowLogic.if(
  {
    $id: Now.ID["manager_approved"],
    condition: `${wfa.dataPill(managerApproval.approval_state, "string")}=approved`
  },
  () => {
    // Second approval - Director (for amounts > $5K)
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_amount"],
        condition: `${wfa.dataPill(_params.trigger.request_item.price, "string")}>5000`
      },
      () => {
        const directorApproval = wfa.action(
          action.core.askForApproval,
          { $id: Now.ID["director_approval"] },
          {
            record: wfa.dataPill(_params.trigger.request_item, "reference"),
            table_name: "sc_req_item",
            users: [wfa.dataPill(director.Record, "reference")],
            approval_rules: { rule_type: "Any", num_approvals: 1 }
          }
        );

        // Only create task if director also approves
        wfa.flowLogic.if(
          {
            $id: Now.ID["director_approved"],
            condition: `${wfa.dataPill(directorApproval.approval_state, "string")}=approved`
          },
          () => {
            // Create fulfillment task
          }
        );
      }
    );
  }
);
```

### Parallel Approval (Legal + Finance)

```typescript
const approval = wfa.action(
  action.core.askForApproval,
  { $id: Now.ID["parallel_approval"] },
  {
    record: wfa.dataPill(_params.trigger.request_item, "reference"),
    table_name: "sc_req_item",
    groups: [
      wfa.dataPill(legalGroup.Record, "reference"),
      wfa.dataPill(financeGroup.Record, "reference")
    ],
    approval_rules: {
      rule_type: "All", // BOTH groups must approve
      num_approvals: 2
    }
  }
);
```

### Task with Template Variables

```typescript
const task = wfa.action(
  action.core.createCatalogTask,
  { $id: Now.ID["create_task_with_vars"] },
  {
    ah_requested_item: wfa.dataPill(
      _params.trigger.request_item.sys_id,
      "reference"
    ),
    ah_short_description: "Configure server from template",
    template_catalog_item: `${serverCatalogItem}`,
    catalog_variables: [
      serverCatalogItem.variables.memory,
      serverCatalogItem.variables.storage,
      serverCatalogItem.variables.operating_system
    ],
    ah_fields: TemplateValue({
      assignment_group: "<infrastructure_team_sys_id>",
      priority: 2
    }),
    ah_wait: false
  }
);
```

### Conditional Task Assignment

```typescript
// Route to different teams based on request category
wfa.flowLogic.if(
  {
    $id: Now.ID["check_hardware"],
    condition: `${wfa.dataPill(_params.trigger.request_item.category, "string")}=hardware`
  },
  () => {
    const task = wfa.action(
      action.core.createCatalogTask,
      { $id: Now.ID["hardware_task"] },
      {
        ah_requested_item: wfa.dataPill(
          _params.trigger.request_item.sys_id,
          "reference"
        ),
        ah_short_description: "Procure hardware",
        ah_fields: TemplateValue({
          assignment_group: "<hardware_team_sys_id>"
        }),
        ah_wait: false
      }
    );
  }
);
wfa.flowLogic.elseIf(
  {
    $id: Now.ID["check_software"],
    condition: `${wfa.dataPill(_params.trigger.request_item.category, "string")}=software`
  },
  () => {
    const task = wfa.action(
      action.core.createCatalogTask,
      { $id: Now.ID["software_task"] },
      {
        ah_requested_item: wfa.dataPill(
          _params.trigger.request_item.sys_id,
          "reference"
        ),
        ah_short_description: "License and install software",
        ah_fields: TemplateValue({
          assignment_group: "<software_team_sys_id>"
        }),
        ah_wait: false
      }
    );
  }
);
```

## Common Mistakes

### ❌ WRONG - Using dot notation for task output

```typescript
const task = wfa.action(action.core.createCatalogTask, ...);
wfa.dataPill(task.CatalogTask, "reference");  // FAILS - field name has space!
```

### ✅ CORRECT - Using bracket notation

```typescript
const task = wfa.action(action.core.createCatalogTask, ...);
wfa.dataPill(task["Catalog Task"], "reference");  // Works correctly
```

### ❌ WRONG - Missing ah\_ prefix

```typescript
wfa.action(
  action.core.createCatalogTask,
  { $id: Now.ID["task"] },
  {
    requested_item: "...", // WRONG - missing ah_ prefix!
    short_description: "..." // WRONG - missing ah_ prefix!
  }
);
```

### ✅ CORRECT - Using ah\_ prefix

```typescript
wfa.action(
  action.core.createCatalogTask,
  { $id: Now.ID["task"] },
  {
    ah_requested_item: "...", // Correct - ah_ prefix
    ah_short_description: "..." // Correct - ah_ prefix
  }
);
```

## Related Patterns

For related patterns, use `load_skill_resource` with skill "wfa-flow":

- `references/patterns/sequential-steps.md` - General action chaining pattern
- `references/patterns/approval-workflow.md` - General approval patterns

## Next Steps

For complete API documentation, use `get_knowledge_source` tool to get **WFA_FLOW_ACTIONS_SERVICE_CATALOG** knowledge source.
