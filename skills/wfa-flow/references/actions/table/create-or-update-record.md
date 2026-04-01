# createOrUpdateRecord Action

Creates new record if no match found, or updates existing record if match exists (upsert operation).

## When to Use

- Synchronizing external system data (create if new, update if exists)
- User provisioning (create user account or update if exists)
- CI/asset management (create or update based on serial number)
- Configuration management (create/update settings by name)
- Idempotent integrations (safe to run multiple times)

## Best Practices

1. **Use Unique Matching Fields** - Include fields that uniquely identify records (email, user_name, serial_number, name)

2. **Wrap in TemplateValue** - Mandatory: `fields: TemplateValue({ ... })` with both unique fields AND fields to set

3. **Check status Output** - Returns `'created'`, `'updated'`, or `'error'`

4. **Verify Table Has Unique Fields** - Use `get_table_schema` to confirm table has unique field constraints

5. **Include All Required Fields** - Provide both unique identifier AND all fields to set/update

## Unique Field Matching

**CRITICAL:** Action uses **table dictionary unique field definitions** to determine create vs update:

| Table          | Unique Fields            |
| -------------- | ------------------------ |
| sys_user       | email, user_name         |
| sys_user_group | name                     |
| cmdb_ci        | serial_number, asset_tag |
| core_company   | name                     |

## Example: User Provisioning

```typescript
const user = wfa.action(
  action.core.createOrUpdateRecord,
  { $id: Now.ID["upsert_user"] },
  {
    table_name: "sys_user",
    fields: TemplateValue({
      email: wfa.dataPill(externalUser.email, "string_full_utf8"), // Unique field
      user_name: wfa.dataPill(externalUser.username, "string"), // Unique field
      first_name: wfa.dataPill(externalUser.first, "string"), // Fields to set
      last_name: wfa.dataPill(externalUser.last, "string"),
      active: true
    })
  }
);

// Check what happened
wfa.flowLogic.if(
  {
    $id: Now.ID["check"],
    condition: `${wfa.dataPill(user.status, "string")}=created`
  },
  () => {
    /* New user created */
  }
);
```

## Important Notes

- **Matching Logic** - If record exists with matching unique fields → update. Otherwise → create.
- **Output Fields** - `status` ('created'/'updated'/'error'), `record` (sys_id), `error_message`
- **Use get_table_schema** - Identify unique fields before using this action

## Next Steps

For Fluent API signatures, parameters, output fields, matching logic, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_TABLE** knowledge source.
