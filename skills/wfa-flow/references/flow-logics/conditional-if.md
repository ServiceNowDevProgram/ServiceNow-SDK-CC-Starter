# Conditional Logic: if/elseIf/else

Conditional branching allows flows to execute different actions based on dynamic conditions evaluated at runtime.

## When to Use

- Route flow execution based on field values (priority, status, category)
- Implement business logic with multiple decision paths
- Handle different scenarios based on trigger data or action results
- Validate lookup results before processing

## Best Practices

1. **Use Template Literals** - Always wrap data pill conditions: `` `${wfa.dataPill(...)}=value` ``

2. **Single Equals for Comparison** - Use `=` not `==` (ServiceNow encoded query format)

3. **Check Empty Values** - Use `ISEMPTY`/`ISNOTEMPTY` operators for null, undefined, or empty string checks

4. **Complex Conditions** - Use `^` for AND, `^OR` for OR:
   - AND: `` `${wfa.dataPill(field1)}=1^${wfa.dataPill(field2)}=2` ``
   - OR: `` `${wfa.dataPill(field1)}=1^OR${wfa.dataPill(field2)}=2` ``

5. **Reference Field Comparisons** - Compare sys_id values using `.value`, not display values

6. **Avoid Deep Nesting** - Prefer elseIf chains over nested if statements for readability

7. **Validate Lookup Results** - Check `status='0'` before using lookup results

## Common Use Cases

- **Priority-Based Routing** - Route incidents based on priority (1=critical → on-call, 2=high → senior, else → standard queue)
- **State Transition Logic** - Different actions for resolved (state=6) vs closed (state=7)
- **Approval Result Handling** - Check approval_state (approved, rejected, cancelled)
- **Lookup Result Validation** - Check `Count>0` before processing results
- **Category-Based Processing** - Different workflows for hardware/software/service requests

## Important Notes

- **Condition Syntax** - Uses ServiceNow encoded query format (same as GlideRecord filters)
- **Field Types Matter** - Boolean uses `true`/`false`, choice fields use internal values (not labels)
- **Runtime Evaluation** - Conditions evaluated at runtime using current data pill values
- **Sequence Matters** - If/elseIf/else blocks evaluate in order. Once matched, subsequent blocks are skipped.

### ⚠️ CRITICAL: JavaScript Functions NOT Supported

Flow logic conditions (if, elseIf, else) do NOT support JavaScript functions like `javascript:gs.daysAgoStart(30)` or `javascript:gs.beginningOfThisWeek()`. Only use data pill comparisons with static values or data pills from trigger/action outputs.

```typescript
// ❌ WRONG - JavaScript function not supported
wfa.flowLogic.if({
  condition: `${wfa.dataPill("trigger.record.sys_updated_on")}<javascript:gs.daysAgoStart(30)`
}, () => { ... });

// ✅ CORRECT - Data pill comparison
wfa.flowLogic.if({
  condition: `${wfa.dataPill("trigger.record.priority")}=1`
}, () => { ... })
wfa.flowLogic.elseIf({
    condition: `${wfa.dataPill(lookup.Count, "integer")}>0`
  }, () => { ... });
```

## Integration Notes

**With Actions:**

- Conditional blocks can contain any actions (create, update, lookup, send)
- Action outputs from previous steps can be used in condition evaluation

**With Loops:**

- Conditionals can be used inside forEach loops for per-item processing
- Combine with exitLoop/skipIteration for advanced loop control

## Next Steps

For Fluent API signatures, parameters, condition syntax operators, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_LOGICS** knowledge source.
