# weekly Trigger

Executes once per week on a specific day and time.

## When to Use

- Weekly reports (compile incident metrics, team performance, executive summaries)
- Weekly maintenance (archive old records, cleanup stale data)
- Data synchronization (sync with external systems, update data warehouse)

## Best Practices

1. **Day of Week Values** - Use 1 (Sunday) through 7 (Saturday) in UTC timezone

2. **Choose Appropriate Day** - Monday (2) common for reports covering previous week. Sunday (1) or Saturday (7) for off-peak processing

3. **Always Use UTC** - Specify time in UTC: `Time({ hours: 8, minutes: 0, seconds: 0 }, "UTC")`

4. **Large Datasets** - Weekly flows process more data than daily. Use batching with `max_results`

## Important Notes

- Week starts Sunday (1) and ends Saturday (7) in UTC
- Fires once per week at specified day/time in UTC (may be delayed by minutes)
- No trigger data (use lookUpRecords with date range conditions)

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_SCHEDULED** knowledge source.
