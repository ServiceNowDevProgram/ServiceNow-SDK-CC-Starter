# lookUpRecords Action

Query multiple records from any ServiceNow table based on conditions.

## When to Use

- Finding all records matching specific criteria
- Bulk processing workflows (iterate over results with forEach)
- Conditional logic based on record count
- Checking existence before creating/updating records
- Fetching data for reports or aggregations

## Best Practices

1. **Always Use max_results** - Prevent timeouts. Recommended: 100-200 for forEach loops

2. **Specific Conditions** - Use precise encoded queries to reduce result set

3. **Access Output Fields Correctly** - **CRITICAL:** Use `Count` (uppercase), `Records` (uppercase), `Table` (uppercase)

4. **Check Count Before forEach** - Validate results exist before iteration

5. **Add Sort Order** - Use `order_by` and `order_direction` for predictable ordering

## Example: Output Field Naming (UPPERCASE)

```typescript
const results = wfa.action(
  action.core.lookUpRecords,
  { $id: Now.ID["lookup"] },
  {
    table: "incident",
    conditions: "active=true^priority=1",
    max_results: 100
  }
);

// ✅ CORRECT - Use UPPERCASE field names
wfa.flowLogic.if(
  {
    $id: Now.ID["check"],
    condition: `${wfa.dataPill(results.Count, "integer")}>0`
  },
  () => {
    wfa.flowLogic.forEach(
      wfa.dataPill(results.Records, "array.object"), // Records (uppercase)
      { $id: Now.ID["loop"] },
      record => {
        // Process each record
      }
    );
  }
);

// ❌ WRONG - Lowercase fields don't exist
// results.records - DOES NOT EXIST
// results.count - DOES NOT EXIST
```

## Important Notes

- **Output Fields** - `Count` (integer), `Records` (array.object), `Table` (string) - All uppercase
- **Empty Results** - Returns empty array if no matches (Count = 0)
- **max_results Default** - No default limit (can return thousands - always specify!)
- **ACL Enforcement** - Only returns records user has read access to

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_TABLE** knowledge source.
