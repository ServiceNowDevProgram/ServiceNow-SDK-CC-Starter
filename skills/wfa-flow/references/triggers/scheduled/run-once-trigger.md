# runOnce Trigger

Executes once at a specific future date and time.

## When to Use

- Maintenance windows (system updates, database maintenance, config changes)
- Event notifications (send reminders before important events, upcoming deadlines)
- Data migrations (one-time migration from legacy systems, bulk transformations)
- Deferred actions (delay action until specific future date, time-delayed approvals)

## Best Practices

1. **Always Use UTC** - Format: `"YYYY-MM-DD HH:MM:SS"` (e.g., `"2026-03-15 02:00:00"`)

2. **Future Dates Only** - Date/time must be in future. Flow errors if run_in is in past.

3. **Flow Deactivates** - Flow automatically deactivates after execution. Cannot reuse. Clone flow if needed.

4. **Error Handling** - Implement comprehensive error handling. You only get one chance.

5. **Test in Sub-Production** - Always test runOnce flows in sub-prod first. Validate all logic before activating.

## Important Notes

- Fires once at specified date/time in UTC (may be delayed by minutes)
- **Automatic Deactivation** - Flow deactivates after execution. Check execution log to verify.
- **Missed Execution** - If system down at scheduled time, execution skipped and flow deactivates (no retry)
- **Cannot Reuse** - After execution, must clone flow or create new one

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_SCHEDULED** knowledge source.
