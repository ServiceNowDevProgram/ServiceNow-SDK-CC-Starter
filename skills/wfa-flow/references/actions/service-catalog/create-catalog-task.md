# createCatalogTask Action

Create catalog fulfillment tasks on service catalog request items.

## When to Use

- Creating fulfillment tasks for approved catalog requests
- Workflow-driven provisioning with task tracking
- Multi-stage fulfillment workflows (procurement, setup, delivery)
- Linking tasks to specific request items for visibility

## Best Practices

1. **Use ah\_ Prefix** - ALL parameters use `ah_` prefix: `ah_requested_item`, `ah_short_description`, `ah_fields`, `ah_wait`

2. **Access Output with Bracket Notation** - Output field is `"Catalog Task"` (with space), requires: `result["Catalog Task"]`

3. **Use TemplateValue for Fields** - `ah_fields` parameter uses TemplateValue format: `TemplateValue({ assignment_group: value, priority: "2" })`

4. **Non-Blocking Recommended** - Set `ah_wait: false` for better performance unless you need to wait for task completion.

5. **Use run_query Tool** - Fetch ah_requested_item sys_id dynamically to ensure request item exists.

## Important Notes

- **⚠️ CRITICAL - Parameter Prefix:** ALL parameters use `ah_` prefix (not standard WFA pattern). This is unique to createCatalogTask.
- **⚠️ CRITICAL - Output Field Name:** Output is `"Catalog Task"` with space - MUST use bracket notation: `task["Catalog Task"]`
- **Differs from createTask** - This action is catalog-specific (sc_task table only). Use `createTask` for other task types.
- **Blocking by Default** - `ah_wait` defaults to `true` (flow pauses until task completes). Set to `false` for non-blocking.
- **Table Name ReadOnly** - `ah_table_name` is always 'sc_task' and cannot be changed
- **TemplateValue Required** - `ah_fields` uses TemplateValue wrapper, not plain string format
- **Variable Population** - Optionally use `template_catalog_item` and `catalog_variables` to populate task variables from template

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_SERVICE_CATALOG** knowledge source.
