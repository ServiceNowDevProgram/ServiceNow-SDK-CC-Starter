# repeat Trigger

Executes at regular intervals.

## When to Use

- Polling external systems (check for new data, monitor status)
- Health checks (monitor service availability, validate integrations)
- Queue processing (process work queue items, handle async tasks)
- Data sync (update cache, refresh data, sync with external systems)

## Best Practices

1. **Minimum Interval** - Avoid intervals <5 minutes to prevent system overload. Use `Duration({ minutes: 15 })`

2. **Idempotent Design** - Design flows safe to run multiple times. Check if work needed before processing:

```typescript
const pendingItems = wfa.action(
  action.core.lookUpRecords,
  { $id: Now.ID["check_pending"] },
  { table: "queue_table", conditions: "processed=false", max_results: 10 }
);

wfa.flowLogic.if(
  {
    $id: Now.ID["check_no_work"],
    condition: `${wfa.dataPill(pendingItems.Count, "integer")}=0`
  },
  () => {
    wfa.flowLogic.endFlow({
      $id: Now.ID["end_no_work"],
      annotation: "No pending items"
    });
  }
);
```

3. **Keep Execution Fast** - Repeat triggers run frequently. Optimization critical.

## Important Notes

- **First Execution** - Fires immediately when flow activated, then repeats at interval
- **No Trigger Data** - Use lookUpRecords to fetch data for processing
- **Overlapping Executions** - If execution takes longer than interval, next waits for current to complete
- **Interval Precision** - Actual execution may drift slightly due to system load

## Next Steps

For Fluent API signatures, parameters, Duration helper usage, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_SCHEDULED** knowledge source.
