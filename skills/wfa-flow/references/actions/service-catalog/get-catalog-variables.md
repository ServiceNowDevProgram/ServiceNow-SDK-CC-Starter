# getCatalogVariables Action

Retrieve catalog variables from a template catalog item and populate them on a request item.

## When to Use

- Template-based request creation (inherit variable definitions from standard templates)
- Dynamic variable population before submitting catalog requests
- Standardizing request configurations across similar items
- Copying variable sets from existing catalog items

## Best Practices

1. **Use Before submitCatalogItemRequest** - Call this action first to populate variables, then submit the request with those variables.

2. **Use run_query Tool** - Fetch requested_item and template_catalog_item sys_ids dynamically to ensure they exist.

3. **Combine with Templates** - Create standard templates for common requests (e.g., "Standard Laptop", "New Hire Setup") to ensure consistency.

## Important Notes

- **No Outputs** - This is an informational action with side effects only. It populates variables on the request item but doesn't return any values.
- **Both Parameters Mandatory** - Both `requested_item` and `template_catalog_item` are required.
- **Template Reference** - `template_catalog_item` references `st_sys_catalog_items_and_variable_sets` (catalog items AND variable sets)
- **Variable Selection** - Optionally use `catalog_variables` parameter to select specific variables to copy (max 12000 chars)
- **Updates Request Item** - Variables are written to the request item record, modifying its state

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_SERVICE_CATALOG** knowledge source.
