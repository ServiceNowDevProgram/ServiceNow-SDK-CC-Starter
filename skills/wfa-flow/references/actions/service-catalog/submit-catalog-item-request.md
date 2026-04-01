# submitCatalogItemRequest Action

Submit catalog item requests programmatically with configurable blocking/non-blocking execution.

## When to Use

- Programmatic catalog ordering (hardware, software, services)
- Bulk provisioning workflows triggered by incidents or changes
- Automated request submission based on business rules
- Integration with external systems for catalog fulfillment
- Self-service automation for standard requests

## Best Practices

1. **Use run_query Tool** - Fetch catalog_item sys_ids dynamically instead of hardcoding. Ensures catalog items exist and are active.

2. **Check Status Output** - Always verify `status` field (0=success, 1=error, 2=timeout) before using `requested_item` output.

3. **Format Variables Correctly** - Use `^`-delimited format for `catalog_item_inputs`: `variable1=value1^variable2=value2`

4. **Non-Blocking by Default** - Leave `wait_for_completion` unset or false for better performance. Only use blocking when you need the request to complete before continuing.

5. **Capture Error Messages** - Store `error_message` output for debugging and user notifications when status ≠ 0.

## Important Notes

- **Status Codes** - 0=success, 1=error, 2=timeout. Always check before using requested_item.
- **Blocking Behavior** - `wait_for_completion: true` pauses flow until request fulfills (can be slow for complex items)
- **Error Handling** - Use conditional logic (if/elseIf/else) to handle all three status outcomes
- **Variable Format** - catalog_item_inputs uses encoded query format with `^` delimiter
- **Quantity Default** - sysparm_quantity defaults to 1 if not specified
- **User Context** - Use sysparm_requested_for to specify who the request is for (defaults to current user)
- **Error Suppression** - Set `_snc_dont_fail_on_error: true` to continue flow even if submission fails

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_SERVICE_CATALOG** knowledge source.
