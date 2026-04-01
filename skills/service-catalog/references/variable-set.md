# Variable Set Reference

For properties and templates, use `get_knowledge_source` with knowledge source `CATALOG_VARIABLE_SET`.

## Attaching to Catalog Items

Attach variable sets via `variableSets: [{ variableSet, order }]` on a Catalog Item or Record Producer. Item-specific variables can be added alongside variable sets.

## Multi-Row Variable Set (MRVS)

Use `type: "multiRow"` for grid/table data entry (e.g., multiple team members). Configure with `setAttributes` for row limits and collapsibility.

## MRVS Constraints

### Unsupported Variable Types in Multi-Row Sets

- AttachmentVariable
- ContainerStartVariable / ContainerEndVariable / ContainerSplitVariable
- HtmlVariable
- CustomVariable / CustomWithLabelVariable
- RichTextLabelVariable
- UIPageVariable

### MRVS Limitations

- "Assign to Field" not supported
- Cannot add variables with read roles
- Set row limits using `max_rows` attribute
- Will not display if added to a container

## Role-Based Access

- `readRoles`: Roles that can view the variable set
- `writeRoles`: Roles that can modify values
- `createRoles`: Roles that can create instances (multiRow)

**Important**: Set-level permissions override variable-level permissions when access is denied at the set level.

## Important Notes

- Variable sets allow centralized updates across all using items
- Catalog UI Policies and Client Scripts can be scoped to a variable set using `appliesTo: 'set'`
- Use `displayTitle: true` to show collapsible section headers

For complete examples and templates, use `get_knowledge_source` with knowledge source `CATALOG_VARIABLE_SET`.
