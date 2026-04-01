# createTask Action

Creates a task record in any task-based table.

## When to Use

- Follow-up tasks (verify fix or document solution after resolution)
- Approval workflows (generate approval tasks for change requests)
- Catalog fulfillment (create fulfillment tasks for service requests)
- Work decomposition (break large work items into subtasks)
- SLA escalations (create escalation tasks when SLA breaches)

## Best Practices

1. **Use Appropriate Task Table** - Choose correct table: `task`, `sc_task`, `sysapproval_approver`, `change_task`, `incident_task`, `problem_task`, `rm_task`

2. **Set Assignment Properly** - Use `run_query` to fetch user/group sys_ids dynamically. Set both `assigned_to` and `assignment_group`.

3. **Use wait Parameter Carefully** - Only set `wait: true` when flow must pause for task completion. Use for approval workflows.

4. **Wrap in TemplateValue** - Mandatory: `field_values: TemplateValue({ ... })`

5. **Capture Output** - **CRITICAL:** Output field is `Record` (uppercase), not `record`. Use `wfa.dataPill(task.Record, "reference")`.

6. **Link to Parent Records** - Use `parent` field for hierarchy. Set table-specific fields (`request_item`, `change_request`, `problem`).

## Task Table Types

| Table                  | Purpose                     |
| ---------------------- | --------------------------- |
| `task`                 | Generic tasks               |
| `sc_task`              | Service catalog tasks       |
| `sysapproval_approver` | Approval tasks              |
| `change_task`          | Change implementation tasks |
| `incident_task`        | Incident response tasks     |
| `problem_task`         | Problem investigation tasks |
| `rm_task`              | Release management tasks    |

## Pattern: Parent-Child Task Relationship

```typescript
// Create parent task
const parentTask = wfa.action(
  action.core.createTask,
  { $id: Now.ID["create_parent"] },
  {
    task_table: "task",
    field_values: TemplateValue({
      short_description: "Main project task",
      assigned_to: "<manager_sys_id>",
      priority: 2
    }),
    wait: false
  }
);

// Create subtask referencing parent
const subtask = wfa.action(
  action.core.createTask,
  { $id: Now.ID["create_subtask"] },
  {
    task_table: "task",
    field_values: TemplateValue({
      short_description: "Subtask: Research phase",
      parent: wfa.dataPill(parentTask.Record, "reference"), // Link to parent
      assigned_to: "<analyst_sys_id>"
    }),
    wait: false
  }
);
```

## createTask vs createRecord

**Use createTask when:**

- Creating records in task-extended tables
- Need wait functionality (`wait: true`)
- Building task hierarchies with parent-child relationships

**Use createRecord when:**

- Creating non-task tables (incident, change_request, user, cmdb_ci)
- Creating custom tables that don't extend task

## Important Notes

- **Output Field Casing:** `Record` (UPPERCASE), not `record` (unlike createRecord)
- **wait Parameter:** Blocks flow until task closed. Monitor for timeout issues.
- **Default State:** Varies by table (usually `-5` for New/Pending)

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_TASK** knowledge source.
