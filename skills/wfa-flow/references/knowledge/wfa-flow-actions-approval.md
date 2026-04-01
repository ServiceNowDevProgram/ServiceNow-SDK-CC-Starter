# WFA_FLOW_ACTIONS_APPROVAL

The Approval Actions API reference provides complete technical specifications for `action.core.askForApproval`, `wfa.approvalRules()`, and `wfa.approvalDueDate()`. This document contains API signatures, parameter definitions, types, and syntax examples. For conceptual guidance, design patterns, best practices, when-to-use information, and approval workflow strategies, refer to skill references.

## action.core.askForApproval

Creates approval records on any ServiceNow record with configurable rule sets.

### Signature

```typescript
wfa.action(
  action.core.askForApproval,
  { $id: Now.ID['action_id'] },
  {
    record: reference,
    table?: string,
    approval_conditions: object,
    approval_reason?: string,
    approval_field?: string,
    journal_field?: string,
    due_date?: datetime
  }
)
```

### Parameters

**Input:**

| Parameter             | Type      | Required | Description                                                  |
| --------------------- | --------- | -------- | ------------------------------------------------------------ |
| `record`              | reference | Yes      | Record sys_id requiring approval                             |
| `table`               | string    | No       | Table name of the record                                     |
| `approval_conditions` | object    | Yes      | Approval rules configuration (use `wfa.approvalRules()`)     |
| `approval_reason`     | string    | No       | Reason for the approval request (max 160 characters)         |
| `approval_field`      | string    | No       | Field name to store approval state                           |
| `journal_field`       | string    | No       | Field name to store approval history and comments            |
| `due_date`            | datetime  | No       | Due date for approval response (use `wfa.approvalDueDate()`) |

**Output:**

| Field            | Type   | Description                                                                                     |
| ---------------- | ------ | ----------------------------------------------------------------------------------------------- |
| `approval_state` | choice | Approval state (approved, rejected, requested, not_required, cancelled, not requested, skipped) |

**Example:**

```typescript
const approval = wfa.action(
  action.core.askForApproval,
  { $id: Now.ID["request_approval"] },
  {
    record: wfa.dataPill(_params.trigger.current, "reference"),
    table: "change_request",
    approval_reason: "Please approve this change request",
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
                users: ["user_sys_id_1", "user_sys_id_2"],
                groups: [],
                manual: false
              }
            ]
          ]
        }
      ]
    })
  }
);

wfa.flowLogic.if(
  {
    $id: Now.ID["check_approved"],
    condition: `${wfa.dataPill(approval.approval_state, "choice")}=approved`
  },
  () => {
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["update"] },
      {
        table_name: "change_request",
        record: wfa.dataPill(_params.trigger.current, "reference"),
        values: TemplateValue({ state: 3 })
      }
    );
  }
);
```

---

## wfa.approvalRules()

Creates approval rules configuration string for `askForApproval` action.

### Signature

```typescript
wfa.approvalRules(approvalRules: ApprovalRulesType): string
```

### Parameters

**1. approvalRules** - Approval rules configuration object with the following structure:

```typescript
{
  conditionType?: 'OR',  // Rule sets connected by OR logic
  ruleSets?: Array<{
    action: 'Approves' | 'Rejects' | 'ApprovesRejects',
    conditionType?: 'AND',  // Rules within set connected by AND logic
    rules?: Array<Array<{
      ruleType: 'Any' | 'All' | 'Res' | 'Count' | 'Percent',
      users?: Array<unknown>,  // User sys_ids (supports datapills)
      groups?: Array<unknown>,  // Group sys_ids (supports datapills)
      count?: number,  // For 'Count' rule type (supports datapills)
      percent?: number,  // For 'Percent' rule type (supports datapills)
      manual?: boolean
    }>>
  }>
}
```

### ruleType Values

**Type:** `'Any' | 'All' | 'Res' | 'Count' | 'Percent'`

- `'Any'` - Approved when ANY single approver approves
- `'All'` - Approved when ALL approvers approve
- `'Res'` - All responded and anyone approves or rejects
- `'Count'` - Specific number of approvals required
- `'Percent'` - Percentage of approvals required

**Example:**

```typescript
// Any approver
ruleType: 'Any',
users: ['user_sys_id_1', 'user_sys_id_2'],
groups: [],
manual: false
```

```typescript
// All approvers
ruleType: 'All',
users: ['cab_member_1', 'cab_member_2', 'cab_member_3'],
groups: [],
manual: false
```

```typescript
// Count-based
ruleType: 'Count',
count: 2,
users: ['user_1', 'user_2', 'user_3', 'user_4', 'user_5'],
groups: [],
manual: false
```

```typescript
// Percentage-based
ruleType: 'Percent',
percent: 50,
users: [],
groups: ['group_sys_id'],
manual: false
```

---

### action Values

**Type:** `'Approves' | 'Rejects' | 'ApprovesRejects'`

- `'Approves'` - Can approve only
- `'Rejects'` - Can reject only
- `'ApprovesRejects'` - Can both approve and reject

**Example:**

```typescript
wfa.approvalRules({
  conditionType: "OR",
  ruleSets: [
    {
      action: "Approves",
      conditionType: "AND",
      rules: [
        [
          {
            ruleType: "Any",
            users: ["manager_sys_id_1"],
            groups: [],
            manual: false
          }
        ]
      ]
    },
    {
      action: "Rejects",
      conditionType: "AND",
      rules: [
        [
          {
            ruleType: "Any",
            users: ["security_lead_sys_id"],
            groups: [],
            manual: false
          }
        ]
      ]
    }
  ]
});
```

---

## wfa.approvalDueDate()

Creates approval due date configuration string for `askForApproval` action.

### Signature

```typescript
wfa.approvalDueDate(approvalDueDate: ApprovalDueDateType): string
```

### Parameters

**1. approvalDueDate** - Approval due date configuration object with the following structure:

```typescript
{
  action: 'none' | 'approve' | 'reject' | 'cancel',
  dateType?: 'actual' | 'relative',
  date?: string,  // For 'actual': '{}', for 'relative': datapill or date string
  duration?: number,
  durationType?: 'years' | 'quarters' | 'months' | 'weeks' | 'days' | 'hours' | 'minutes' | 'seconds',
  daysSchedule?: string  // Schedule sys_id for business hours
}
```

### action Values

**Type:** `'none' | 'approve' | 'reject' | 'cancel'`

- `'none'` - No automatic action on due date
- `'approve'` - Auto-approve on due date
- `'reject'` - Auto-reject on due date
- `'cancel'` - Cancel approval on due date

**Example:**

```typescript
// No action on due date
wfa.approvalDueDate({
  action: "none",
  dateType: "actual",
  date: "{}",
  duration: 3,
  durationType: "days",
  daysSchedule: ""
});
```

```typescript
// Auto-reject on due date
wfa.approvalDueDate({
  action: "reject",
  dateType: "actual",
  date: "{}",
  duration: 4,
  durationType: "hours",
  daysSchedule: ""
});
```

---

### dateType Values

**Type:** `'actual' | 'relative'`

- `'actual'` - Use `date: '{}'` with `duration` for relative time from now
- `'relative'` - Use `date` field with datapill or specific date string

**Example (Actual with Duration):**

```typescript
wfa.approvalDueDate({
  action: "none",
  dateType: "actual",
  date: "{}",
  duration: 3,
  durationType: "days",
  daysSchedule: ""
});
```

**Example (Relative with DataPill):**

```typescript
wfa.approvalDueDate({
  action: "approve",
  dateType: "relative",
  date: wfa.dataPill(_params.trigger.current.due_date, "glide_date_time"),
  duration: 15,
  durationType: "days",
  daysSchedule: "08fcd0830a0a0b2600079f56b1adb9ae"
});
```

---

### daysSchedule

**Type:** `string`

Schedule sys_id for business hours calculation. Empty string for calendar days.

**Example:**

```typescript
// Calendar days
wfa.approvalDueDate({
  action: "reject",
  dateType: "actual",
  date: "{}",
  duration: 5,
  durationType: "days",
  daysSchedule: ""
});
```

```typescript
// Business days
wfa.approvalDueDate({
  action: "reject",
  dateType: "relative",
  date: wfa.dataPill(_params.trigger.current.due_date, "glide_date_time"),
  duration: 10,
  durationType: "days",
  daysSchedule: "08fcd0830a0a0b2600079f56b1adb9ae"
});
```

---

## Complete Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["change_approval_workflow"],
    name: "Change Request Approval Workflow"
  },
  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["change_created"] },
    { table: "change_request", condition: "type=normal" }
  ),
  _params => {
    const approval = wfa.action(
      action.core.askForApproval,
      { $id: Now.ID["cab_approval"] },
      {
        record: wfa.dataPill(_params.trigger.current, "reference"),
        table: "change_request",
        approval_reason: "CAB approval required",
        approval_field: "approval",
        journal_field: "work_notes",
        approval_conditions: wfa.approvalRules({
          conditionType: "OR",
          ruleSets: [
            {
              action: "ApprovesRejects",
              conditionType: "AND",
              rules: [
                [
                  {
                    ruleType: "Percent",
                    percent: 50,
                    users: [],
                    groups: ["f6498860436012103cf32f0b76b8f2ce"],
                    manual: false
                  }
                ]
              ]
            }
          ]
        }),
        due_date: wfa.approvalDueDate({
          action: "reject",
          dateType: "actual",
          date: "{}",
          duration: 5,
          durationType: "days",
          daysSchedule: ""
        })
      }
    );

    wfa.flowLogic.if(
      {
        $id: Now.ID["check_approved"],
        condition: `${wfa.dataPill(approval.approval_state, "choice")}=approved`
      },
      () => {
        wfa.action(
          action.core.updateRecord,
          { $id: Now.ID["approve_change"] },
          {
            table_name: "change_request",
            record: wfa.dataPill(_params.trigger.current, "reference"),
            values: TemplateValue({ state: 3, comments: "CAB approved" })
          }
        );
      }
    );

    wfa.flowLogic.elseIf(
      {
        $id: Now.ID["check_rejected"],
        condition: `${wfa.dataPill(approval.approval_state, "choice")}=rejected`
      },
      () => {
        wfa.action(
          action.core.updateRecord,
          { $id: Now.ID["reject_change"] },
          {
            table_name: "change_request",
            record: wfa.dataPill(_params.trigger.current, "reference"),
            values: TemplateValue({ state: 3, comments: "CAB rejected" })
          }
        );
      }
    );
  }
);
```

---
