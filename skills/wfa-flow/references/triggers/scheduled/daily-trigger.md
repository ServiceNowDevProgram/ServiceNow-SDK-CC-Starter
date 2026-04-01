# daily Trigger

Executes once per day at a specific time.

## When to Use

- Daily summary reports (send overnight incident summaries to managers)
- Nightly data cleanup (archive closed records, purge stale data)
- Batch processing (bulk updates, external system sync, overnight processing)

## Best Practices

1. **Always Use UTC** - Specify time in UTC timezone: `Time({ hours: 2, minutes: 0, seconds: 0 }, "UTC")`

2. **Off-Peak Scheduling** - Schedule during low-usage hours (2-5 AM UTC) to minimize system impact

3. **Use lookUpRecords** - No trigger data provided. Fetch records using lookUpRecords with date conditions

4. **Data Batching** - Use `max_results` to prevent timeouts when processing large datasets

## Important Notes

- Fires once daily at specified UTC time (may be delayed by minutes)
- No trigger data (must use lookUpRecords to fetch records for processing)
- Missed executions are skipped (no automatic retry if system down)
- Flow must be active at scheduled time

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_SCHEDULED** knowledge source.
