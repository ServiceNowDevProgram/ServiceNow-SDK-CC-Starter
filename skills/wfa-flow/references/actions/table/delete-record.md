# deleteRecord Action

Permanently deletes a record from any ServiceNow table.

## When to Use

- Removing temporary/test data records
- Deleting expired or obsolete records
- Cleaning up duplicate records after deduplication
- Removing cancelled requests or tasks
- Purging stale integration queue items

## Best Practices

1. **Use with Extreme Caution** - Deletion is **permanent and irreversible**. No recovery mechanism.

2. **Consider Inactivation Instead** - Setting `active=false` is safer and preserves audit trails:

```typescript
// ✅ SAFER - Inactivate instead of delete
wfa.action(
  action.core.updateRecord,
  { $id: Now.ID["inactivate"] },
  {
    table_name: "incident",
    record: wfa.dataPill(recordToRemove, "reference"),
    values: TemplateValue({ active: false })
  }
);
```

3. **Validate Before Delete** - Check conditions carefully and verify record sys_id exists.

4. **Check Dependencies** - Verify no active references exist before deleting to avoid orphaned records.

5. **Wrap in Conditional Logic** - Use if conditions to prevent accidental deletions.

6. **Log Deletion Operations** - Create audit records before deleting for compliance tracking.

7. **Avoid Business Critical Tables** - Never delete from core system tables (sys_user, cmdb_ci, etc.) without explicit approval.

## Important Notes

- **Permanent Operation** - All field values, audit history, and related records are permanently lost
- **No Rollback** - Cannot be undone once executed
- **ACL Permissions** - Flow user must have delete rights on table
- **Cascading Impact** - Related records may become orphaned

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_TABLE** knowledge source.
