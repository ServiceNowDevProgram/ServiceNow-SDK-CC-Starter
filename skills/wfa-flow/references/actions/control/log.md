# Log Action

Write custom messages to flow execution log for debugging, monitoring, and audit.

## When to Use

- Debugging complex flow logic
- Monitoring flow execution progress
- Auditing critical operations
- Recording data transformations
- Tracking decision points in conditional logic

## Best Practices

1. **Use Only When Explicitly Requested** - Only include logs when user specifically asks. Avoid adding by default (performance impact, clutter).

2. **Choose Appropriate Log Levels:**
   - `"info"` - Normal operational messages (execution start, completion, progress)
   - `"warn"` - Unexpected but non-blocking conditions (high record counts, missing optional data)
   - `"error"` - Failures and error conditions (failed operations, validation errors)

3. **Avoid Sensitive Data** - Never log passwords, API keys, PII, tokens, or confidential data.

4. **Keep Messages Concise** - 255-character limit. Keep short and focused.

5. **Use Meaningful Context** - Include identifiable info (record numbers, IDs, names):

```typescript
// ✅ Good: Includes context
wfa.action(
  action.core.log,
  { $id: Now.ID["log"] },
  {
    log_level: "info",
    log_message: `Updated incident ${wfa.dataPill(incident.number, "string")} to Resolved`
  }
);

// ❌ Bad: No context
log_message: "Updated record";
```

## Log Levels Guide

- **info** - Informational messages for normal flow execution
- **warn** - Warning messages for unexpected but non-critical conditions
- **error** - Error messages for failures requiring attention

## When NOT to Use

- Don't log in production flows unless required for audit/compliance
- Don't use as primary error handling (use proper error handling patterns)
- Don't log every action (creates noise, impacts performance)

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_CONTROL** knowledge source.
