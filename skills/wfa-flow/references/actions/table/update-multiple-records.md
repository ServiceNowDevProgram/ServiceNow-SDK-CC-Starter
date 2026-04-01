# updateMultipleRecords Action

Updates multiple records in a single operation based on query conditions.

## When to Use

- Bulk assignment (assign all unassigned incidents to group)
- Mass state transitions (close all resolved incidents)
- Bulk field updates (set priority on all matching records)
- Batch inactivation (inactivate all expired records)
- Data cleanup (update stale fields across multiple records)

## Best Practices

1. **Test Conditions First** - Use lookUpRecords with same conditions to preview which records will be updated

2. **Use Precise Conditions** - Avoid overly broad conditions. Validate query carefully.

3. **Wrap in TemplateValue** - Mandatory: `field_values: TemplateValue({ ... })`

4. **Check Status & Count** - Verify `status='0'` (success) and examine `count` field for records updated

5. **Consider Performance** - Business rules fire **for each record** updated. Large updates (>200 records) can be slow.

## Example: Business Rule Performance Warning

```typescript
// ⚠️ WARNING - Business rules fire for EACH record
const result = wfa.action(
  action.core.updateMultipleRecords,
  { $id: Now.ID["bulk_update"] },
  {
    table_name: "incident",
    conditions: "active=true^priority=1", // If this matches 500 records...
    field_values: TemplateValue({ state: 2 })
  }
);
// ...business rules fire 500 times!

// Check results
wfa.flowLogic.if(
  {
    $id: Now.ID["check"],
    condition: `${wfa.dataPill(result.status, "string")}=0`
  },
  () => {
    // Success - wfa.dataPill(result.count, 'integer') records updated
  }
);
```

## Important Notes

- **Business Rules Fire Per Record** - Each updated record triggers before/after business rules, related updates cascade
- **Performance Impact** - Expect longer execution times for large updates (>200 records)
- **Output Fields** - `status` ('0' success, '1' error), `count` (integer), `message` (error details)
- **Consider Alternatives** - For very large updates (>1000), consider scheduled jobs

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_TABLE** knowledge source.
