# createdOrUpdated Trigger

Fires when a record is either created or updated.

## When to Use

- Data synchronization (sync operational CIs to external CMDB, whether new or updated)
- Compliance validation (apply same validation rules for new and updated records)
- Notification on condition match (notify users regardless of create vs update event)
- Audit logging (track all changes to active user accounts)
- External system integration (send approved requests to provisioning system)

## Best Practices

1. **Use for Sync Logic** - Ideal when create and update logic is identical. Reduces duplication vs separate created/updated flows

2. **Check changed_fields** - Empty array indicates create, populated array indicates update:

   ```typescript
   condition: `${wfa.dataPill(_params.trigger.changed_fields, "array.object")}ISEMPTY`;
   ```

3. **Choose Right Strategy** - Use `trigger_strategy: 'unique_changes'` to prevent duplicate executions on updates

4. **Condition Applies to Both** - Ensure trigger condition makes sense for both creates and updates

## Important Notes

- Trigger strategy (once, unique_changes, every, always) only affects **updates**. Creates always fire once.
- `changed_fields` is empty array for creates, populated for updates
- If create and update logic differ significantly, use separate created/updated triggers instead
- Executes more frequently than separate triggers - use specific conditions to manage volume

## Next Steps

For Fluent API signatures, parameters, output fields, changed_fields structure, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_RECORD** knowledge source.
