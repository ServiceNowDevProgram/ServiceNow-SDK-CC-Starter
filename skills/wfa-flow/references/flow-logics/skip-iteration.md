# skipIteration

Skips the current forEach iteration and continues with the next item in the loop.

## Table of Contents

- [When to Use](#when-to-use)
- [Best Practices](#best-practices)
- [Important Notes](#important-notes)
- [Next Steps](#next-steps)

## When to Use

- Filter items based on criteria
- Skip items that don't meet conditions
- Skip already-processed items
- Handle edge cases gracefully

## Best Practices

1. **Early Exit Pattern:** Place skip checks at beginning of loop for efficiency. This avoids executing unnecessary logic for items that will be skipped.

   ```typescript
   wfa.flowLogic.forEach(items, { $id: Now.ID["loop"] }, item => {
     // Skip check first - saves processing time
     wfa.flowLogic.if(
       {
         $id: Now.ID["check"],
         condition: `${wfa.dataPill(item.priority, "string")}=5`
       },
       () => {
         wfa.flowLogic.skipIteration({
           $id: Now.ID["skip"],
           annotation: "Skipping low priority item"
         });
       }
     );

     // Main processing only runs for non-skipped items
   });
   ```

2. **Add Annotations:** Explain why iteration is skipped for better debugging and flow execution history understanding.

3. **Use with Conditionals:** skipIteration should always be inside if/elseIf blocks. Unconditional skipIteration makes the loop pointless.

4. **Multiple Skip Conditions:** Use multiple if statements with skipIteration for different skip reasons. Each can have its own annotation for clarity.

5. **skipIteration vs exitLoop:**
   - skipIteration: "Skip inactive users but process active ones" (continues loop)
   - exitLoop: "Found the user we need, stop searching" (stops entire loop)

6. **Performance Benefit:** skipIteration improves performance by avoiding unnecessary processing. Place checks early to maximize savings.

## Important Notes

- **Continues Loop:** skipIteration skips the current iteration and immediately moves to the next item. The loop continues processing remaining items.
- **Immediate Effect:** Once skipIteration executes, no further code in that iteration runs. Control jumps to the next iteration.
- **No Return Value:** skipIteration does not return a value. Use counters or logs to track skipped items if needed.
- **Nested Loops:** In nested loops, skipIteration only skips the current iteration of the innermost loop.
- **Performance Impact:** Properly using skipIteration improves performance by avoiding unnecessary work.

## Next Steps

For Fluent API signatures, parameters, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_LOGICS** knowledge source.
