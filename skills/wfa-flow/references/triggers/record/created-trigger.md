# created Trigger

Fires when a new record is created in the specified table.

## When to Use

- Auto-assign high priority incidents to on-call teams
- Provision new user accounts (create workspace, send welcome email)
- Trigger approval workflows for high-value service requests
- Schedule CAB reviews for high-risk change requests
- Notify teams of new records requiring immediate attention

## Best Practices

1. **Use Specific Conditions** - Filter at trigger level with encoded query syntax (e.g., `"priority=1^assignment_groupISEMPTY"`)

2. **Background vs Foreground** - Use `run_flow_in: 'background'` (default) for async. Use `'foreground'` only for validation/blocking

3. **Extended Tables** - Keep `run_on_extended: 'false'` to avoid duplicate executions on child tables

4. **Condition Operators** - Use `^` (AND), `^OR` (OR), `ISEMPTY`, `ISNOTEMPTY` for precise filtering

## Important Notes

- Trigger fires **after** record is inserted into database (sys_id exists, all fields populated)
- Access trigger data via `_params.trigger.current`
- Multiple flows can trigger on same table (execution order not guaranteed)
- Background triggers fire after transaction completes; foreground triggers fire during transaction

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_RECORD** knowledge source.
