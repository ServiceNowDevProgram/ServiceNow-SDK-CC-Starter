# updateRecord Action

Updates an existing record in any ServiceNow table.

## When to Use

- Updating record state/status during workflow progression
- Assigning records to users or groups
- Adding work notes or comments to records
- Updating reference fields based on business logic
- Modifying record fields after approval or validation

## Best Practices

1. **Wrap Values in TemplateValue** - Mandatory: `values: TemplateValue({ ... })`

2. **Use Explicit sys_id** - Provide exact sys_id via `record` parameter

3. **Update Only Changed Fields** - Include only fields needing updates to minimize business rule execution

4. **Validate Record Exists** - Check lookup results before updating to avoid errors

5. **Proper Data Pill Types** - Use `'reference'` for sys_ids and reference fields

## Example: Update with TemplateValue

```typescript
const updated = wfa.action(
  action.core.updateRecord,
  { $id: Now.ID["update"] },
  {
    table_name: "incident",
    record: wfa.dataPill(incident.Record, "reference"),
    values: TemplateValue({
      assignment_group: wfa.dataPill(group.Record, "reference"),
      state: 2,
      work_notes: "Auto-assigned by workflow"
    })
  }
);
```

## Important Notes

- **Server-Side Validation** - Data policy and business rules ARE enforced
- **Output Field** - Returns `record` field with updated record's sys_id
- **Concurrent Updates** - Other processes may modify record between lookup and update
- **ACL Permissions** - Flow user must have write access to table and fields

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_TABLE** knowledge source.
