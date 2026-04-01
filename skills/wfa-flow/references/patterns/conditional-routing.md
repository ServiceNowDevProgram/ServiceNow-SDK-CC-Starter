# Conditional Routing Pattern

Different actions based on conditions using if/elseIf/else branching.

## Table of Contents

- [When to Use](#when-to-use)
- [Pattern Structure](#pattern-structure)
- [Example 1: Priority-Based Incident Routing](#example-1-priority-based-incident-routing)
- [Example 2: Amount-Based Purchase Approval](#example-2-amount-based-purchase-approval)
- [Common Condition Patterns](#common-condition-patterns)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)

## When to Use

- Different actions needed based on data values
- Business rules with multiple outcomes
- Priority-based routing
- Status-dependent processing

## Pattern Structure

```
Trigger → Check Condition → Action A (if)
                         ├→ Action B (elseIf)
                         └→ Action C (else)
```

## Example 1: Priority-Based Incident Routing

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["priority_based_routing"],
    name: "Priority-Based Routing",
    description: "Routes incidents to teams based on priority"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_created"] },
    {
      table: "incident",
      condition: "active=true",
      run_flow_in: "background"
    }
  ),

  _params => {
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_p1"],
        condition: `${wfa.dataPill(_params.trigger.current.priority, "string")}=1`
      },
      () => {
        wfa.action(
          action.core.updateRecord,
          { $id: Now.ID["assign_p1"] },
          {
            table_name: "incident",
            record: wfa.dataPill(_params.trigger.current, "reference"),
            values: TemplateValue({
              assignment_group: "<senior_team_sys_id>",
              state: 2
            })
          }
        );

        wfa.action(
          action.core.sendEmail,
          { $id: Now.ID["notify_p1"] },
          {
            ah_to: "senior-team@company.com",
            ah_subject: `CRITICAL: ${wfa.dataPill(_params.trigger.current.number, "string")}`,
            ah_body: `Critical incident requires immediate attention.`
          }
        );
      }
    );

    wfa.flowLogic.elseIf(
      {
        $id: Now.ID["check_p2"],
        condition: `${wfa.dataPill(_params.trigger.current.priority, "string")}=2`
      },
      () => {
        wfa.action(
          action.core.updateRecord,
          { $id: Now.ID["assign_p2"] },
          {
            table_name: "incident",
            record: wfa.dataPill(_params.trigger.current, "reference"),
            values: TemplateValue({
              assignment_group: "<regular_team_sys_id>",
              state: 2
            })
          }
        );
      }
    );

    wfa.flowLogic.else({ $id: Now.ID["default_assignment"] }, () => {
      wfa.action(
        action.core.updateRecord,
        { $id: Now.ID["assign_general"] },
        {
          table_name: "incident",
          record: wfa.dataPill(_params.trigger.current, "reference"),
          values: TemplateValue({
            assignment_group: "<general_queue_sys_id>",
            state: 1
          })
        }
      );
    });
  }
);
```

## Example 2: Amount-Based Purchase Approval

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["amount_based_approvals"],
    name: "Amount-Based Approvals",
    description: "Different approval tiers by amount"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["po_created"] },
    {
      table: "purchase_order",
      condition: "state=pending_approval",
      run_flow_in: "background"
    }
  ),

  _params => {
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_large"],
        condition: `${wfa.dataPill(_params.trigger.current.amount, "string")}>=10000`
      },
      () => {
        const director = wfa.action(
          action.core.lookUpRecord,
          { $id: Now.ID["find_director"] },
          {
            table: "sys_user",
            conditions: "user_name=finance.director"
          }
        );

        wfa.action(
          action.core.askForApproval,
          { $id: Now.ID["director_approval"] },
          {
            record: wfa.dataPill(_params.trigger.current, "reference"),
            table: "purchase_order",
            approval_reason: `Large purchase requires director approval`,
            approval_conditions: wfa.approvalRules({
              conditionType: "OR",
              ruleSets: [
                {
                  action: "Approves",
                  conditionType: "AND",
                  rules: [
                    [
                      {
                        ruleType: "Any",
                        users: [wfa.dataPill(director.Record.sys_id, "string")],
                        groups: [],
                        manual: false
                      }
                    ]
                  ]
                }
              ]
            }),
            due_date: wfa.dataPill(
              _params.trigger.current.due_date,
              "glide_date_time"
            )
          }
        );
      }
    );

    wfa.flowLogic.elseIf(
      {
        $id: Now.ID["check_medium"],
        condition: `${wfa.dataPill(_params.trigger.current.amount, "string")}>=5000`
      },
      () => {
        const manager = wfa.action(
          action.core.lookUpRecord,
          { $id: Now.ID["find_manager"] },
          {
            table: "sys_user",
            conditions: "user_name=procurement.manager"
          }
        );

        wfa.action(
          action.core.askForApproval,
          { $id: Now.ID["manager_approval"] },
          {
            record: wfa.dataPill(_params.trigger.current, "reference"),
            table: "purchase_order",
            approval_reason: "Manager approval required",
            approval_conditions: wfa.approvalRules({
              conditionType: "OR",
              ruleSets: [
                {
                  action: "Approves",
                  conditionType: "AND",
                  rules: [
                    [
                      {
                        ruleType: "Any",
                        users: [wfa.dataPill(manager.Record.sys_id, "string")],
                        groups: [],
                        manual: false
                      }
                    ]
                  ]
                }
              ]
            }),
            due_date: wfa.dataPill(
              _params.trigger.current.due_date,
              "glide_date_time"
            )
          }
        );
      }
    );

    wfa.flowLogic.else({ $id: Now.ID["auto_approve"] }, () => {
      wfa.action(
        action.core.updateRecord,
        { $id: Now.ID["approve_small"] },
        {
          table_name: "purchase_order",
          record: wfa.dataPill(_params.trigger.current, "reference"),
          values: TemplateValue({
            state: "approved"
          })
        }
      );
    });
  }
);
```

## Common Condition Patterns

```typescript
// Equality: =
condition: `${wfa.dataPill(field, "string")}=value`;

// Comparison: >=, <=, >, <
condition: `${wfa.dataPill(field, "string")}>=100`;

// Not equal: !=
condition: `${wfa.dataPill(field, "string")}!=value`;

// Empty checks: ISEMPTY, ISNOTEMPTY
condition: `${wfa.dataPill(field, "string")}ISNOTEMPTY`;

// AND: ^
condition: `${wfa.dataPill(field1, "string")}=value1^${wfa.dataPill(field2, "string")}=value2`;

// OR: ^OR
condition: `${wfa.dataPill(field1, "string")}=value1^OR${wfa.dataPill(field2, "string")}=value2`;
```

## Best Practices

1. **Order Matters** - Most specific conditions first (if), general last (else)
2. **Use elseIf** - For mutually exclusive conditions (priority=1 vs priority=2 vs priority=3)
3. **Always Include else** - Handles unexpected cases and provides default behavior
4. **Encoded Query Format** - Use ServiceNow encoded query syntax (same as GlideRecord filters)

## Common Use Cases

- Priority-based routing (P1 → senior team, P2 → regular team, else → queue)
- Amount-threshold approvals ($10K+ → director, $5K+ → manager, else → auto-approve)
- Category/department routing
- Multi-tier escalation

## Next Steps

For Fluent API signatures, parameters, outputs, and complete coding examples, use `get_knowledge_source` tool to retrieve:

- **WFA_FLOW_TRIGGER_RECORD** - Record trigger API (created, updated, createdOrUpdated)
- **WFA_FLOW_LOGICS** - Flow logic API (if/elseIf/else) with condition syntax and operators
- **WFA_FLOW_ACTIONS_TABLE** - Table action APIs (lookUpRecord, updateRecord)
- **WFA_FLOW_ACTIONS_COMMUNICATION** - Communication action APIs (sendEmail, sendNotification)

These knowledge sources contain authoritative API documentation with all parameters, data types, and working examples.
