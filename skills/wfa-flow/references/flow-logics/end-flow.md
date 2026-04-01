# endFlow

Immediately terminates the entire flow execution. No subsequent actions or logic will execute.

## Table of Contents

- [When to Use](#when-to-use)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Important Notes](#important-notes)
- [Next Steps](#next-steps)

## When to Use

Use endFlow when you need to terminate flow based on runtime conditions that cannot be determined at trigger time:

- **Conditions depend on lookup results** (not available at trigger time)
- **Runtime validation needed** (state can change after trigger fires)
- **Complex business logic** that can't be expressed in trigger conditions
- **Duplicate prevention** checks requiring database lookups
- **Permission validation** requiring role/group membership checks

**Prefer trigger conditions when possible:**

```typescript
// Less efficient - flow fires then ends
wfa.flowLogic.if({ $id, condition: `${priority}!=1` }, () => {
  wfa.flowLogic.endFlow({ $id: Now.ID["end"] });
});

// More efficient - flow only fires for priority 1
// Use trigger condition: "priority=1" instead
```

## Best Practices

1. **Use Trigger Conditions First:** Filter at trigger level when possible to avoid unnecessary flow executions. This improves system performance and reduces flow execution logs.

2. **Combined Conditions:** Use AND (^) and OR (^OR) in trigger conditions instead of multiple endFlow checks. More efficient and easier to maintain.

3. **Add Annotations:** Always explain why flow is ending. This helps with debugging and understanding flow execution history.

   ```typescript
   wfa.flowLogic.endFlow({
     $id: Now.ID["end"],
     annotation:
       "Duplicate request detected - ending to prevent double processing"
   });
   ```

4. **Use Conditionally:** Always use endFlow inside conditional blocks. Unconditional endFlow at start of flow makes the trigger pointless.

5. **Handle Edge Cases:** Use endFlow to gracefully handle edge cases and prevent errors from invalid data or states.

## Common Use Cases

- **Duplicate Prevention:** Check if request/record already exists before processing
- **Permission Validation:** Verify user has required role/group membership
- **State Validation:** Re-check current record state (may have changed since trigger)
- **Dependency Check:** Verify parent/related records exist and are valid
- **Business Rules Enforcement:** Exit when business rules determine no action needed

## Important Notes

- **Immediate Termination:** endFlow immediately terminates the entire flow. No subsequent actions, logic, or loops will execute.
- **No Return Value:** endFlow does not return a value or set output variables. The flow simply stops.
- **Execution Status:** Flows that end via endFlow show as "Completed" in execution logs. Use annotations to distinguish.
- **Nested Contexts:** endFlow terminates the entire flow even when called from inside loops, conditionals, or nested logic.
- **Different from exitLoop:** exitLoop exits a loop but continues flow. endFlow terminates the entire flow execution.

## Next Steps

For Fluent API signatures, parameters, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_LOGICS** knowledge source.
