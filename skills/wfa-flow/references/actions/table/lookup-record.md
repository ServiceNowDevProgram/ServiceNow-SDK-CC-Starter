# lookUpRecord Action

Query a single record from any ServiceNow table based on conditions.

## When to Use

- Finding a specific user/group by name or email
- Looking up reference data before creating/updating records
- Validating record existence before processing
- Fetching parent record details
- Getting configuration or metadata records

## Best Practices

1. **Always Check status** - Verify `status='0'` (success) before using result. Status '1' indicates error/not found.

2. **Use Uppercase Field Names** - **CRITICAL:** Access `Record` (uppercase), `Table` (uppercase), not lowercase.

3. **Use Specific Conditions** - Use unique identifiers (email, number) to match exactly one record.

4. **Handle Multiple Matches** - Set `if_multiple_records_are_found_action`: `'use_first_record'` or `'error'`.

5. **Handle Not Found** - Use if/else to handle cases where record isn't found.

## Example: Status Checking & Uppercase Fields

```typescript
const user = wfa.action(
  action.core.lookUpRecord,
  { $id: Now.ID["find_user"] },
  {
    table: "sys_user",
    conditions: "email=john.doe@company.com",
    if_multiple_records_are_found_action: "use_first_record"
  }
);

// ✅ CORRECT - Check status and use uppercase field names
wfa.flowLogic.if(
  {
    $id: Now.ID["check"],
    condition: `${wfa.dataPill(user.status, "string")}=0`
  },
  () => {
    // Record found - use uppercase Record field
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["update"] },
      {
        record: wfa.dataPill(user.Record, "reference"), // Record (uppercase)
        values: TemplateValue({ active: "false" })
      }
    );
  }
);
wfa.flowLogic.else({ $id: Now.ID["not_found"] }, () => {
  // Handle not found case
});

// ❌ WRONG - Lowercase fields don't exist
// user.record - DOES NOT EXIST
```

## Important Notes

- **Output Fields** - `status` (string '0' or '1'), `Record` (reference, uppercase), `Table` (string, uppercase), `error_message` (string)
- **Multiple Matches** - First record returned by default (use sort_column for control)
- **ACL Enforcement** - Returns error if user lacks read access

## Next Steps

For Fluent API signatures, parameters, output fields, status values, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_TABLE** knowledge source.
