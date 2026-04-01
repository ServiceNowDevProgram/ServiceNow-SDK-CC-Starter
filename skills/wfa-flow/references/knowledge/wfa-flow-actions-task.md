# WFA_FLOW_ACTIONS_TASK

# Workflow Automation Flow Actions - Task

The Task Actions API reference provides complete technical specifications for creating task records. This document contains action signatures, parameter definitions, types, and syntax examples. For conceptual guidance, when-to-use recommendations, and best practices, refer to skill references.

---

## action.core.createTask

Creates a task on any ServiceNow task table with dynamically configured fields. The Parent field associates to a Parent record (Incident Task to Incident record). To block the flow until this task is completed, select 'Wait'.

### Input Parameters

| Parameter    | Type          | Default | Mandatory | Description                                                              |
| ------------ | ------------- | ------- | --------- | ------------------------------------------------------------------------ |
| task_table   | string        | -       | Yes       | Task table name (task, sc_task, change_task, sysapproval_approver, etc.) |
| field_values | TemplateValue | -       | No        | Field values for the new task (use `TemplateValue()` helper)             |
| wait         | boolean       | false   | No        | Wait for task completion before continuing flow execution                |

### Output Fields

| Field  | Type      | Description                       |
| ------ | --------- | --------------------------------- |
| Record | reference | sys_id of the created task record |
| Table  | string    | Table name where task was created |

**Note:** The `Record` output field name is **uppercase** (unlike createRecord which uses lowercase `record`). Use `wfa.dataPill(task.Record, "reference")` to reference the created task.

### Usage Examples

**Basic Task Creation:**

```typescript
const task = wfa.action(
  action.core.createTask,
  { $id: Now.ID["create_follow_up"] },
  {
    task_table: "task",
    field_values: TemplateValue({
      short_description: "Follow-up on incident resolution",
      description: wfa.dataPill(_params.trigger.current.close_notes, "string"),
      priority: 3,
      assigned_to: wfa.dataPill(
        _params.trigger.current.assigned_to,
        "reference"
      ),
      parent: wfa.dataPill(_params.trigger.current, "reference")
    }),
    wait: false
  }
);
```

**Service Catalog Task (sc_task):**

```typescript
const scTask = wfa.action(
  action.core.createTask,
  { $id: Now.ID["create_fulfillment_task"] },
  {
    task_table: "sc_task",
    field_values: TemplateValue({
      short_description: "Fulfill laptop request",
      description: "Order and configure laptop per specifications",
      request_item: wfa.dataPill(_params.trigger.current, "reference"),
      assignment_group: "<group_sys_id>",
      priority: 2
    }),
    wait: false
  }
);
```

**Task with Wait for Completion:**

```typescript
const approvalTask = wfa.action(
  action.core.createTask,
  { $id: Now.ID["create_approval_task"] },
  {
    task_table: "sysapproval_approver",
    field_values: TemplateValue({
      short_description: "Approve change request",
      source_table: "change_request",
      sysapproval: wfa.dataPill(_params.trigger.current, "reference"),
      approver: "<manager_sys_id>",
      due_date: wfa.dataPill(
        _params.trigger.current.due_date,
        "glide_date_time"
      ),
      state: "requested"
    }),
    wait: true
  }
);

wfa.flowLogic.if(
  {
    $id: Now.ID["check_approval"],
    condition: `${wfa.dataPill(approvalTask.Record.state, "string")}=approved`
  },
  () => {
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["approve_change"] },
      {
        table_name: "change_request",
        record: wfa.dataPill(_params.trigger.current, "reference"),
        values: TemplateValue({ state: 3 })
      }
    );
  }
);
```

**Creating Multiple Tasks from Loop:**

```typescript
const cis = wfa.action(
  action.core.lookUpRecords,
  { $id: Now.ID["get_cis"] },
  {
    table: "cmdb_ci",
    conditions: "install_status=1^operational_status=1",
    max_results: 50
  }
);

wfa.flowLogic.forEach(
  wfa.dataPill(cis.Records, "array.object"),
  { $id: Now.ID["create_ci_tasks"] },
  ci => {
    wfa.action(
      action.core.createTask,
      { $id: Now.ID["task_for_ci"] },
      {
        task_table: "task",
        field_values: TemplateValue({
          short_description: `Update CI: ${wfa.dataPill(ci.name, "string")}`,
          cmdb_ci: wfa.dataPill(ci, "reference"),
          assignment_group: "<ci_team_sys_id>",
          priority: 3,
          parent: wfa.dataPill(_params.trigger.current, "reference")
        }),
        wait: false
      }
    );
  }
);
```

**Parent-Child Task Hierarchy:**

```typescript
const parentTask = wfa.action(
  action.core.createTask,
  { $id: Now.ID["create_parent"] },
  {
    task_table: "task",
    field_values: TemplateValue({
      short_description: "Main project task",
      assigned_to: "<project_manager_sys_id>",
      priority: 2,
      due_date: wfa.dataPill(
        _params.trigger.current.expected_end,
        "glide_date_time"
      )
    }),
    wait: false
  }
);

const subtask1 = wfa.action(
  action.core.createTask,
  { $id: Now.ID["create_subtask_1"] },
  {
    task_table: "task",
    field_values: TemplateValue({
      short_description: "Subtask: Research phase",
      parent: wfa.dataPill(parentTask.Record, "reference"),
      assigned_to: "<analyst_sys_id>",
      priority: 3,
      due_date: wfa.dataPill(parentTask.Record.due_date, "glide_date_time")
    }),
    wait: false
  }
);
```

**Change Task Creation:**

```typescript
const changeTask = wfa.action(
  action.core.createTask,
  { $id: Now.ID["create_change_task"] },
  {
    task_table: "change_task",
    field_values: TemplateValue({
      short_description: "Implement database upgrade",
      change_request: wfa.dataPill(_params.trigger.current, "reference"),
      assigned_to: "<dba_sys_id>",
      assignment_group: "<dba_group_sys_id>",
      priority: 2,
      planned_start_date: wfa.dataPill(
        _params.trigger.current.start_date,
        "datetime"
      ),
      planned_end_date: wfa.dataPill(
        _params.trigger.current.end_date,
        "datetime"
      )
    }),
    wait: false
  }
);
```

### Task Table Types

Common task tables in ServiceNow:

| Table                  | Purpose                     | Key Fields                                                 |
| ---------------------- | --------------------------- | ---------------------------------------------------------- |
| `task`                 | Generic tasks               | short_description, assigned_to, priority, due_date, parent |
| `sc_task`              | Service catalog tasks       | request_item, assignment_group, short_description          |
| `sysapproval_approver` | Approval tasks              | approver, source_table, sysapproval, state, due_date       |
| `change_task`          | Change implementation tasks | change_request, planned_start_date, planned_end_date       |
| `incident_task`        | Incident response tasks     | parent (incident), assigned_to, priority                   |
| `problem_task`         | Problem investigation tasks | problem, assigned_to, work_notes                           |
| `rm_task`              | Release management tasks    | release, planned_start_date, planned_end_date              |

---
