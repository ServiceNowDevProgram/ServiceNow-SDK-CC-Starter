# Loop: forEach

Iterate over arrays of records or data, executing actions for each item in the collection.

## Table of Contents

- [When to Use](#when-to-use)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Performance Considerations](#performance-considerations)
- [Important Notes](#important-notes)
- [Next Steps](#next-steps)

## When to Use

- Process multiple records from lookUpRecords
- Bulk operations on multiple items
- Send notifications to multiple users/groups
- Iterate over flow variable arrays

## Best Practices

1. **Limit Record Processing:** Use max_results in lookUpRecords to prevent timeouts. Keep iterations under 100-200 records for optimal performance.

2. **Use skipIteration for Filtering:** Skip items that don't meet criteria instead of wrapping entire loop body in conditional logic.

3. **Use exitLoop for Early Exit:** Stop loop when goal is achieved (e.g., finding first match). This improves performance by avoiding unnecessary iterations.

4. **Consider Batch Actions:** For simple updates without complex logic, updateMultipleRecords is more efficient than forEach + updateRecord.

5. **Validate Loop Input:** Check that the array/records exist before looping to avoid flow errors on empty results.

## Common Use Cases

| Scenario             | Pattern                              | Notes                                    |
| -------------------- | ------------------------------------ | ---------------------------------------- |
| Bulk assignment      | forEach + updateRecord               | Consider updateMultipleRecords if simple |
| Team notifications   | forEach + sendNotification/sendEmail | Personalize message per recipient        |
| Find first available | forEach + if + exitLoop              | Stop when first match found              |
| Filter and process   | forEach + if + skipIteration         | Skip items not meeting criteria          |
| Cascading updates    | forEach + nested lookups + update    | Update related records for each item     |

## Performance Considerations

**ForEach Loop Behavior:**

- **No hard technical limit:** forEach will process any number of records provided
- **Performance recommendation:** Keep iterations under 100-200 records per flow execution for optimal performance
- **Large loops can cause:** Flow execution timeouts, system resource constraints, slower processing

**Best Practices for Large Datasets:**

- Always specify `max_results` based on expected workload
- Use skipIteration to filter early and reduce processing
- Use exitLoop when you find what you need
- For very large datasets, use batch processing with multiple scheduled flow executions

**Warning Signs:**

- ⚠️ lookUpRecords without `max_results` parameter
- ⚠️ forEach loops with >200 iterations
- ⚠️ Heavy operations (API calls, complex updates) within loops
- ⚠️ Nested forEach loops with large outer datasets

## Important Notes

- **Sequential Processing:** forEach processes items sequentially, not in parallel. Each iteration completes before the next begins.
- **Timeout Limits:** Very large loops (>200 items) may timeout. Use batching or multiple flows for large datasets.
- **Item Reference:** The loop item variable references the current iteration's data. Use wfa.dataPill with the item reference to access fields.
- **Break and Continue:** Use exitLoop (like break) to stop the loop entirely, skipIteration (like continue) to skip current iteration only.
- **Data Modification:** Modifying the looped array during iteration can cause unexpected behavior. Complete the loop before modifying source data.

## Next Steps

For Fluent API signatures, parameters, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_LOGICS** knowledge source.
