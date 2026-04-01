# slaPercentageTimer Action

Pauses flow execution until a percentage of SLA time has elapsed.

## When to Use

- Progressive escalation workflows (50% → manager, 75% → director, 90% → VP)
- Automated reminders at SLA milestones
- Resource allocation based on SLA progress
- Customer notifications at service commitment milestones
- Priority escalation as SLA time progresses

## Best Practices

1. **Choose Strategic Percentages** - Align with escalation policies:
   - Early warning: 25-30%
   - Standard escalation: 50-60%
   - Urgent escalation: 75-80%
   - Critical escalation: 90-95%

2. **Check Status Output** - Always verify `status` output (completed, paused, repair, cancelled, skipped) before proceeding.

3. **Progressive Escalation** - Use multiple slaPercentageTimer actions in sequence for graduated escalation.

4. **Handle SLA Pauses** - SLAs can be paused or in repair mode. Timer respects pause periods and recalculates on repair.

## SLA Timer Behavior

**Percentage Calculation:**

- 0% = SLA start time, 100% = SLA breach time
- Example: For 4-hour SLA, 75% = 3 hours elapsed

**Flow Execution:**

- Flow pauses until percentage is reached
- Other flows and system operations continue normally
- Flow resumes automatically when percentage is reached

**SLA States:**

- Timer respects pause periods (status: 'paused')
- Repair mode recalculates timeline (status: 'repair')

## Important Notes

- **Blocking Action** - Flow execution pauses until specified percentage is reached
- **SLA Record Required** - Workflow should be triggered by SLA-enabled table with active SLAs
- **Percentage Range** - Must be between 0 and 100
- **Multiple Timers** - Use multiple actions in sequence for multi-stage escalation
- **Output Fields** - `total_duration` (complete SLA duration), `scheduled_end_date_time` (expected breach time)
- **Table Support** - Works with any SLA-enabled table (incident, change_request, problem, task, etc.)
- **Not for Simple Delays** - Use regular wait actions for non-SLA workflows

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_SLA** knowledge source.
