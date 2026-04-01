# exitLoop

Immediately exits the current forEach loop and continues flow execution after the loop.

## Table of Contents

- [When to Use](#when-to-use)
- [Best Practices](#best-practices)
- [Important Notes](#important-notes)
- [Next Steps](#next-steps)

## When to Use

- ✅ Finding first match in search
- ✅ Goal achieved, remaining iterations unnecessary
- ✅ Critical error prevents further processing
- ❌ Filtering items (use skipIteration instead)
- ❌ Terminating entire flow (use endFlow instead)

## Best Practices

1. **Add Annotations:** Explain why loop is exiting for better debugging and flow diagram readability.

   ```typescript
   wfa.flowLogic.exitLoop({
     $id: Now.ID["exit"],
     annotation: "Found first available technician - stopping search"
   });
   ```

2. **Use with Conditionals:** exitLoop should always be inside if/elseIf blocks. Unconditional exitLoop makes the loop pointless.

3. **exitLoop vs skipIteration:**
   - Use exitLoop: "Find first available technician and assign" (stop processing entirely)
   - Use skipIteration: "Process all high-priority incidents, skip low priority" (continue with next item)

4. **exitLoop vs endFlow:**
   - exitLoop: Exits current loop, continues flow execution after loop
   - endFlow: Terminates entire flow immediately

5. **Nested Loops:** In nested loops, exitLoop only exits the innermost loop. To exit multiple levels, use flags or multiple exitLoop statements.

## Important Notes

- **Scope:** exitLoop only exits the current forEach loop. In nested loops, it exits only the innermost loop.
- **Flow Continues:** After exitLoop, flow execution continues with the next action after the loop.
- **No Return Value:** exitLoop does not return a value. Use conditional logic or update records to track exit reason.
- **Immediate Exit:** exitLoop takes effect immediately. Any code after exitLoop in the same iteration will not execute.

## Next Steps

For Fluent API signatures, parameters, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_LOGICS** knowledge source.
