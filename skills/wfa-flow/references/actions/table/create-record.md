# createRecord Action

Creates a new record in any ServiceNow table.

## When to Use

- Creating incidents from emails or catalog requests
- Creating child records (tasks, approvals, change requests)
- Creating audit or log records in custom tables
- Duplicating records with modifications (template-based workflows)
- Integration record creation from external system data

## Best Practices

1. **Wrap Values in TemplateValue** - Mandatory: `values: TemplateValue({ field: value })`

2. **Use Proper Data Pill Types** - `'reference'` for reference fields, `'string_full_utf8'` for text, static values for choices

3. **Fetch Reference IDs Dynamically** - Use `run_query` tool to get sys_ids instead of hardcoding

4. **Validate Required Fields** - Use `get_table_schema` tool to discover mandatory fields before creation

5. **Capture Output** - Access created record's sys_id via lowercase `record` field: `wfa.dataPill(result.record, 'reference')`

## Important Notes

- **Server-Side Validation** - Data policy and business rules ARE enforced. UI policies do NOT apply.
- **Output Field** - Returns lowercase `record` field with created record's sys_id
- **Error Handling** - Missing mandatory fields or invalid references cause flow to fail
- **ACL Permissions** - Flow execution user must have create rights on table

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_TABLE** knowledge source.
