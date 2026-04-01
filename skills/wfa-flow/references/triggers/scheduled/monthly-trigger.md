# monthly Trigger

Executes once per month on a specific day and time.

## When to Use

- Monthly billing (generate invoices, calculate charges, send statements)
- Compliance reports (audit reports, regulatory documentation)
- Financial processing (close periods, reconcile accounts)

## Best Practices

1. **Day of Month** - Use 1-31. If day exceeds month's days, runs on **LAST DAY** of that month

2. **February Handling** - Day 30/31 run on Feb 28 (or 29 in leap years). Use day 1-28 for consistent execution

3. **Always Use UTC** - `Time({ hours: 2, minutes: 0, seconds: 0 }, "UTC")`

4. **Large Data Volumes** - Monthly flows process significant data. Use batching with `max_results`

## Important Notes

- No trigger data (use lookUpRecords with date range conditions like `javascript:gs.monthStart(-1)@javascript:gs.monthEnd(-1)`)
- Monthly failures can have serious business impact - implement error handling

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_SCHEDULED** knowledge source.
