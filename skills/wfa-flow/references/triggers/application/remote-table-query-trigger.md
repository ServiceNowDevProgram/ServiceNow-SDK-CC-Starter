# remoteTableQuery Trigger

Fires when a remote table is queried from an external system. Enables custom logic for remote table data access and transformation.

## Table of Contents

- [⚠️ Critical Limitations](#️-critical-limitations)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Important Notes](#important-notes)
- [Next Steps](#next-steps)

## ⚠️ Critical Limitations

**IMPORTANT:** WFA flows are **declarative**, not imperative JavaScript. The following patterns are **NOT supported**:

❌ **JavaScript String Operations:**

- `queryId.includes("urgent")` - NOT SUPPORTED
- `params.filter((p) => p.active)` - NOT SUPPORTED
- `JSON.parse(queryParams)` - NOT SUPPORTED

❌ **Assigning Data Pills to Variables:**

- `const queryId = wfa.dataPill(...)` - NOT SUPPORTED

✅ **CORRECT - Use data pills directly in flow conditions and actions:**

```typescript
wfa.action(
  action.core.log,
  { $id: Now.ID["log_query"] },
  {
    log_level: "info",
    log_message: `Query ID: ${wfa.dataPill(_params.trigger.query_id, "string")}, Table: ${wfa.dataPill(_params.trigger.table_name, "string")}`
  }
);
```

**For Complex Processing:** Use Script actions or Integration Hub for external API calls, data transformation, and dynamic query processing.

## Best Practices

1. **Specify Table Name:** Use u_table parameter to target specific remote tables. Without it, flow fires for all remote table queries.

2. **Keep Processing Fast:** Remote table queries are synchronous - the calling system waits for flow completion. Target <2 seconds response time.

3. **Implement Caching:** Cache frequently accessed remote data to reduce external API calls and improve response time.

4. **Error Handling:** Implement comprehensive error handling for external system failures. Errors cause the query to fail for the calling system.

## Common Use Cases

- **Logging/Auditing Remote Access:** Track who accesses remote data, monitor query patterns
- **Caching Query Results:** Cache frequently accessed data to reduce external API calls
- **Data Transformation:** Use Script actions to transform external data formats to ServiceNow structure

## Important Notes

- **Synchronous Execution:** Remote table query triggers execute synchronously. Calling system waits for completion. Keep processing VERY fast.
- **Query Parameters:** Access via `_params.trigger.query_id`, `table_name`, `query_parameters`
- **Return Data:** Flow must return data in expected ServiceNow remote table response format
- **No Async Option:** Cannot run in background - must complete before returning
- **Performance Critical:** Poor performance directly impacts user experience
- **Complex Operations:** External API calls, dynamic URL construction, and data transformation require custom Script actions or Integration Hub

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_APPLICATION** knowledge source.
