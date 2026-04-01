# Record Producer Reference

For properties and templates, use `get_knowledge_source` with knowledge source `CATALOG_ITEM_RECORD_PRODUCER`.

## Field Mapping Methods

| Scenario                       | Recommended Method |
| ------------------------------ | ------------------ |
| Simple text/choice mapping     | `mapToField: true` |
| System values (gs.getUserID()) | Script             |
| Conditional logic              | Script             |
| Calculated values              | Script             |
| Variables in Variable Sets     | Script             |

- **mapToField** (Recommended): Set `mapToField: true` and `field` on the variable to map directly to the target table field.
- **Script-Based**: Use `script` (pre-insert) or `postInsertScript` (post-insert) with `Now.include(...)` for external files.
- **Variable Name Matching**: Variable name matches target field name for automatic mapping. Less explicit than mapToField.

## Script Types

| Script             | Timing        | Can call update()? |
| ------------------ | ------------- | ------------------ |
| `script`           | Before insert | **No**             |
| `postInsertScript` | After insert  | **Yes**            |
| `saveScript`       | On step save  | No                 |

### Available Script Objects

| Object              | Description                                        |
| ------------------- | -------------------------------------------------- |
| `current`           | GlideRecord of the record being created            |
| `producer.var_name` | Form variable values                               |
| `cat_item`          | Record Producer definition (postInsertScript only) |
| `gs`                | GlideSystem                                        |

### Script Rules

- **Never** call `current.update()` or `current.insert()` in pre-insert script
- **Never** call `current.setAbortAction()`
- **Never** set `current.sys_class_name`
- Use `postInsertScript` for post-creation updates, related records, notifications

## Unsupported Tables

Do **not** create record producers for:

- `sc_request`, `sc_req_item`, `sc_task` — Use Catalog Items instead.

## Redirect Options

- **generatedRecord** (default): Redirects to the created record
- **catalogHomePage**: Redirects to catalog home page

For complete examples and templates, use `get_knowledge_source` with knowledge source `CATALOG_ITEM_RECORD_PRODUCER`.
