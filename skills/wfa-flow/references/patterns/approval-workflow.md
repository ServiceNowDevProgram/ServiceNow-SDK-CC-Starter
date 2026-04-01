# Approval Workflow Pattern

Complex approval processes with rules and conditional routing.

## Table of Contents

- [When to Use](#when-to-use)
- [Pattern Structure](#pattern-structure)
- [Example 1: Manager Approval with Result Handling](#example-1-manager-approval-with-result-handling)
- [Example 2: Multi-Tier Approval (Priority-Based)](#example-2-multi-tier-approval-priority-based)
- [Approval Rule Types](#approval-rule-types)
- [Approval States](#approval-states)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)

## When to Use

- Multi-tier approval requirements
- Conditional approval routing
- Complex approval rules (any, all, count, percent)
- Post-approval processing based on result

## Pattern Structure

```
Trigger → Lookup Approvers → Ask for Approval → Check Result → Route Based on State
```

## Example 1: Manager Approval with Result Handling

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["expense_approval_workflow"],
    name: "Expense Approval Workflow",
    description: "Manager approval for expenses over $500"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["expense_created"] },
    {
      table: "incident",
      condition: "priority=1^active=true",
      run_on_extended: "false",
      run_flow_in: "background",
      run_when_user_list: [],
      run_when_setting: "both",
      run_when_user_setting: "any"
    }
  ),

  _params => {
    const manager = wfa.action(
      action.core.lookUpRecord,
      { $id: Now.ID["find_manager"] },
      {
        table: "sys_user",
        conditions: "user_name=manager.user^active=true"
      }
    );

    const approval = wfa.action(
      action.core.askForApproval,
      { $id: Now.ID["request_approval"] },
      {
        record: wfa.dataPill(_params.trigger.current, "reference"),
        table: "incident",
        approval_reason: "Approval required for critical incident",
        approval_field: "approval",
        journal_field: "approval_history",
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
                    users: [wfa.dataPill(manager.Record, "string")],
                    groups: [],
                    manual: false
                  }
                ]
              ]
            }
          ]
        }),
        due_date: ""
      }
    );

    wfa.flowLogic.if(
      {
        $id: Now.ID["check_approved"],
        condition: `${wfa.dataPill(approval.approval_state, "string")}=approved`
      },
      () => {
        wfa.action(
          action.core.updateRecord,
          { $id: Now.ID["mark_approved"] },
          {
            table_name: "incident",
            record: wfa.dataPill(_params.trigger.current, "reference"),
            values: TemplateValue({
              state: 2
            })
          }
        );

        wfa.action(
          action.core.sendEmail,
          { $id: Now.ID["notify_approved"] },
          {
            ah_to: "manager@company.com",
            ah_subject: "Incident Approved",
            ah_body: "Your incident has been approved by your manager."
          }
        );
      }
    );

    wfa.flowLogic.elseIf(
      {
        $id: Now.ID["check_rejected"],
        condition: `${wfa.dataPill(approval.approval_state, "string")}=rejected`
      },
      () => {
        wfa.action(
          action.core.updateRecord,
          { $id: Now.ID["mark_rejected"] },
          {
            table_name: "incident",
            record: wfa.dataPill(_params.trigger.current, "reference"),
            values: TemplateValue({
              state: 7
            })
          }
        );
      }
    );
  }
);
```

## Example 2: Multi-Tier Approval (Priority-Based)

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["tiered_incident_approval"],
    name: "Tiered Incident Approval",
    description: "Approval routing based on incident priority"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["po_created"] },
    {
      table: "incident",
      condition: "priority=1^state=1",
      run_on_extended: "false",
      run_flow_in: "background",
      run_when_user_list: [],
      run_when_setting: "both",
      run_when_user_setting: "any"
    }
  ),

  _params => {
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_critical"],
        condition: `${wfa.dataPill(_params.trigger.current.priority, "string")}=1`
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

        const vp = wfa.action(
          action.core.lookUpRecord,
          { $id: Now.ID["find_vp"] },
          {
            table: "sys_user",
            conditions: "user_name=finance.vp"
          }
        );

        wfa.action(
          action.core.askForApproval,
          { $id: Now.ID["director_vp_approval"] },
          {
            record: wfa.dataPill(_params.trigger.current, "reference"),
            table: "incident",
            approval_reason:
              "High-priority incident requires Director AND VP approval",
            approval_field: "approval",
            journal_field: "approval_history",
            approval_conditions: wfa.approvalRules({
              conditionType: "OR",
              ruleSets: [
                {
                  action: "Approves",
                  conditionType: "AND",
                  rules: [
                    [
                      {
                        ruleType: "All",
                        users: [
                          wfa.dataPill(director.Record, "string"),
                          wfa.dataPill(vp.Record, "string")
                        ],
                        groups: [],
                        manual: false
                      }
                    ]
                  ]
                }
              ]
            }),
            due_date: ""
          }
        );
      }
    );

    wfa.flowLogic.elseIf(
      {
        $id: Now.ID["check_high"],
        condition: `${wfa.dataPill(_params.trigger.current.priority, "string")}=2`
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
            table: "incident",
            approval_reason: "Manager approval required",
            approval_field: "approval",
            journal_field: "approval_history",
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
                        users: [wfa.dataPill(manager.Record, "string")],
                        groups: [],
                        manual: false
                      }
                    ]
                  ]
                }
              ]
            }),
            due_date: ""
          }
        );
      }
    );

    wfa.flowLogic.else({ $id: Now.ID["auto_approve"] }, () => {
      wfa.action(
        action.core.updateRecord,
        { $id: Now.ID["approve_small"] },
        {
          table_name: "incident",
          record: wfa.dataPill(_params.trigger.current, "reference"),
          values: TemplateValue({
            state: 2
          })
        }
      );
    });
  }
);
```

## Approval Rule Types

**Any** - Approved when ANY single approver approves:

```typescript
{ ruleType: 'Any', users: ['<sys_id>'], groups: [], manual: false }
```

**All** - Approved when ALL approvers approve (consensus):

```typescript
{ ruleType: 'All', users: ['<sys_id1>', '<sys_id2>'], groups: [], manual: false }
```

**Count** - Approved when N approvers approve (e.g., 2 out of 5):

```typescript
{ ruleType: 'Count', count: 2, users: ['<id1>', '<id2>', '<id3>'], groups: [], manual: false }
```

**Percent** - Approved when percentage approve (e.g., 50% of group):

```typescript
{ ruleType: 'Percent', percent: 50, users: [], groups: ['<group_sys_id>'], manual: false }
```

## Approval States

- `approved` - All required approvers approved
- `rejected` - One or more rejected (approval fails immediately)
- `requested` - Pending approval
- `not_required` - No approval needed
- `cancelled` - Approval cancelled

## Best Practices

1. **Dynamic Lookups** - Use `run_query` tool to get current approver sys_ids (avoid hardcoding)
2. **Handle All States** - Always check approval_state and handle both approved/rejected outcomes
3. **Due Dates** - Set realistic due dates using wfa.approvalDueDate()
4. **Choose Rule Type** - Any (flexibility), All (consensus), Count/Percent (quorum)
5. **Notify Approvers** - Use sendEmail or sendNotification to alert approvers

## Common Use Cases

- Expense/purchase approval workflows (amount-based tiers)
- Change request approvals (CAB approval)
- Access request approvals (manager + security)
- Multi-tier spending authority (manager → director → VP)

## Next Steps

For Fluent API signatures, parameters, outputs, and complete coding examples, use `get_knowledge_source` tool to retrieve:

- **WFA_FLOW_TRIGGER_RECORD** - Record trigger API (created, updated, createdOrUpdated)
- **WFA_FLOW_ACTIONS_TABLE** - Table action APIs (lookUpRecords, updateRecord)
- **WFA_FLOW_ACTIONS_APPROVAL** - Approval action API (askForApproval) with approval rules and helper functions
- **WFA_FLOW_LOGICS** - Flow logic API (if/elseIf/else, forEach) with condition syntax

These knowledge sources contain authoritative API documentation with all parameters, data types, and working examples.
