# updated Trigger

Fires when an existing record is modified.

## When to Use

- Notify teams when incident priority escalates to critical
- Send assignment change notifications to newly assigned users
- Trigger workflows on state transitions (resolved, closed, cancelled)
- Escalate when SLAs are breached
- Proceed with change implementation after approval granted

## Trigger Strategy Options

- **once** - Fires once per transaction (use for simple notifications)
- **unique_changes** - Fires once per unique field change set (⭐ Recommended for most cases)
- **every** - Fires every time condition is met (can cause multiple executions)
- **always** - Fires on every update regardless of conditions (use with caution)

## Best Practices

1. **Choose Right Strategy** - Use `trigger_strategy: 'unique_changes'` to prevent duplicate executions while tracking field changes

2. **Monitor changed_fields** - Access `_params.trigger.changed_fields` array to determine which fields triggered the update

3. **Avoid Update Loops** - Don't update the same record field that triggered the flow. Use different fields or add loop prevention conditions

4. **Combine Conditions** - Use trigger conditions to narrow scope, then check `changed_fields` in flow logic for precision

## Important Notes

- `changed_fields` output contains array of field names that changed
- Trigger provides NEW values (after update), not old values
- Condition is evaluated against NEW record values
- Trigger strategy only affects updates (creates always fire once)
- Multiple updates in one transaction behave per strategy: once (1x), unique_changes (1x per unique set), every/always (Nx)

## Next Steps

For Fluent API signatures, parameters, output fields, changed_fields structure, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_RECORD** knowledge source.
