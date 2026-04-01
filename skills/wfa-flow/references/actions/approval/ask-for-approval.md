# askForApproval Action

Requests approval for a record and waits for approver response.

## When to Use

- Change request approvals (CAB, manager, multi-level)
- Expense/purchase approvals with threshold-based routing
- Access request approvals (manager, security, dual approval)
- Service catalog fulfillment approvals
- Contract or agreement approvals (legal, procurement, executive)

## Best Practices

1. **Use run_query Tool** - Fetch current user/group sys_ids dynamically instead of hardcoding. Ensures approvers exist and are active.

2. **Choose Appropriate Rule Type** - `Any` (single approver), `All` (consensus), `Count` (specific number), `Percent` (percentage-based)

3. **Set Due Dates** - Use `wfa.approvalDueDate()` for relative or specific due dates (relative to record field, or none)

4. **Check Approval State** - Always verify `approval_state` output (approved, rejected, requested, not_required, cancelled) and handle both approved/rejected outcomes.

5. **Validate Approver Lists** - Verify approver lists are not empty and contain valid sys_ids before creating approval.

## Important Notes

- **Blocking Action** - Flow execution pauses until approval completes (approved, rejected, or cancelled)
- **No Auto-Timeout** - Approvals do not automatically timeout. Implement your own timeout logic if needed.
- **Rejection Handling** - Flow continues after rejection. Use conditional logic to handle appropriately.
- **Sequential vs Parallel** - Multiple actions for sequential (manager → director → VP), single action with rules for parallel (legal + finance)
- **Group Performance** - For >20 approvers, use groups instead of individual users
- **Approval Records** - Creates records in `sysapproval_approver` table with state, comments, and approval date

## Next Steps

For Fluent API signatures, parameters, output fields, approval rules structure, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_APPROVAL** knowledge source.
