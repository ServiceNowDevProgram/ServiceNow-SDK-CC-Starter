# SLA Task Trigger

Triggers when an SLA reaches specific stages (50%, 75%, 100% breached) or when a task_sla record is created or updated.

## Best Practices

1. **Select Appropriate SLA Stage:** Choose the right SLA stage for your escalation strategy
   - Use 50% for early warning notifications to assignees
   - Use 75% for manager escalation
   - Use 100% (breach) for critical escalations requiring immediate action

2. **Filter by Table:** Specify target tables to avoid triggering on all SLA records
   - Use condition: `task.sys_class_name=incident` for incident SLAs only
   - Target specific tables: incident, change_request, problem, sc_task

3. **Use Background Execution for High-Volume:** Set `run_flow_in: 'background'` for tables with many active SLAs
   - Prevents performance impact on user transactions
   - Allows parallel processing of multiple SLA events

4. **Check SLA State Before Acting:** Verify SLA is still active and not paused/cancelled
   - SLA could be paused or repaired after trigger fires
   - Check `_params.trigger.current.active` and `_params.trigger.current.stage` values

5. **Access Task Record via Relationship:** Use dot-walking to access the related task
   - `_params.trigger.current.task` provides access to parent incident/task
   - Example: `_params.trigger.current.task.assigned_to.email`

6. **Avoid Duplicate Triggers:** Ensure your conditions prevent multiple executions for the same SLA event
   - Add state checks to prevent re-triggering on subsequent updates
   - Use specific stage values in trigger conditions

7. **Log SLA Events:** Always log SLA trigger events for audit and troubleshooting
   - Include task number, SLA name, stage, and scheduled breach time
   - Helps diagnose SLA-related escalation issues

8. **Handle Timezone Considerations:** SLA times are in system timezone
   - Use appropriate timezone conversions for notifications
   - Consider user local times for escalation notifications

9. **Test with Non-Production SLAs:** Validate flows with test SLAs before production
   - Create short-duration test SLAs to verify trigger behavior
   - Test all stages (50%, 75%, breach) independently

10. **Monitor SLA Flow Performance:** Track execution times and failures
    - SLA triggers can fire frequently in busy systems
    - Set up monitoring for failed escalations or stuck flows

## Common Use Cases

- **Progressive Escalation:** Escalate to different levels at 50%, 75%, and breach stages
- **Automated SLA Notifications:** Send reminders to assignees when SLA approaches breach
- **SLA Breach Response:** Automatically create escalation tasks or notify management on breach
- **SLA Milestone Tracking:** Log or record SLA milestone events for reporting and analysis
- **Priority Elevation:** Automatically increase priority when SLA reaches critical thresholds
- **Resource Allocation:** Assign additional resources when SLA hits 75% threshold

## Important Notes

- **Trigger Timing:** Fires when SLA percentage crosses the configured threshold
  - 50% stage: Fires at midpoint of SLA duration
  - 75% stage: Fires at three-quarters of SLA duration
  - Breach stage: Fires when SLA breach time is reached

- **Task SLA Record Access:** Trigger provides access to task_sla record and related task
  - `_params.trigger.current` = task_sla record
  - `_params.trigger.current.task` = related task record (incident, change, etc.)
  - Dot-walk to task fields: `_params.trigger.current.task.short_description`

- **SLA Pauses and Repairs:** SLA can be paused or repaired after trigger fires
  - Check `_params.trigger.current.paused` field
  - Check `_params.trigger.current.stage` for current stage value
  - Handle stage changes appropriately in flow logic

- **Multiple SLAs per Task:** A single task can have multiple SLAs
  - Filter by SLA definition name if targeting specific SLA
  - Use condition: `sla.definition.name=Resolution Time`

- **Performance Impact:** SLA triggers can fire frequently in high-volume environments
  - Use background execution for non-critical escalations
  - Implement efficient lookups and minimize external calls
  - Consider batching notifications if possible

- **Stage Values:** Common stage field values in task_sla
  - `in_progress` - SLA is actively running
  - `paused` - SLA is paused (pause condition met)
  - `completed` - SLA completed successfully
  - `breached` - SLA has breached

## Performance Considerations

- **Filter Early:** Use specific conditions in trigger to reduce unnecessary executions
- **Background vs Foreground:** Use background execution for non-urgent escalations
- **Minimize Lookups:** Cache frequently accessed data or use trigger data directly
- **Batch Operations:** When possible, batch multiple SLA actions together
- **Monitor Execution Times:** Track and optimize slow SLA workflows
- **Avoid Recursive Triggers:** Ensure SLA updates in flow don't cause re-triggering

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_APPLICATION** knowledge source.
