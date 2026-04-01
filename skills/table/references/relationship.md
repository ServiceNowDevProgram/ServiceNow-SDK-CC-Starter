# Relationships and Related Lists

## Contents

- Determine the relationship path
- Implicit relationship (reference-based)
- Explicit relationship (custom query)
- Existing platform relationships
- Naming conventions
- query_with script format
- Basic vs advanced fields
- Avoidance

## Determine the relationship path

Choose the correct approach based on what exists between the tables:

- **Reference field exists** between the tables → use implicit relationship (no `sys_relationship` record needed)
- **No reference field** or custom query logic needed → use explicit relationship (create `sys_relationship` record)
- **Adding an existing platform relationship** (e.g., Attachments) → use the known `REL:` ID directly

## Implicit relationship (reference-based)

When table A has a `ReferenceColumn` pointing to table B, create only two records:

1. `sys_ui_related_list` — the list container for the target table and view
2. `sys_ui_related_list_entry` — set `related_list` to `tablename.reference_field` format (e.g., `custom_task.department`)

No `sys_relationship` record is needed — the platform infers the relationship from the reference field.

## Explicit relationship (custom query)

When no reference field exists between the tables, create three records in order:

1. `sys_relationship` — define the relationship with `basic_apply_to` (parent table), `basic_query_from` (child table), and optionally a `query_with` script
2. `sys_ui_related_list` — list container for the target table and view
3. `sys_ui_related_list_entry` — set `related_list` to `REL:${relationshipRecord.$id}` format

Always reference records via variable (e.g., `list_id: \`\${listRecord.$id}\``), never by hardcoded sys_id.

## Existing platform relationships

For well-known relationships, create only `sys_ui_related_list` + `sys_ui_related_list_entry` using the hardcoded `REL:` ID. No `sys_relationship` record needed.

Common platform relationship IDs:

- Attachments: `REL:b9edf0ca0a0a0b010035de2d6b579a03`
- Applications with Role: `REL:66c422fac0a80a880012fadcb8c2480e`
- Approval History: `REL:247c6f15670303003b4687cb5685ef32`

## Naming conventions

- Use the user-specified name if provided.
- Otherwise generate a descriptive name: `[TableA] to [TableB]` for simple references, `[TableA] to [TableB] by [condition]` for conditional relationships.

## query_with script format

The `query_with` field must use the wrapper format:

```
(function refineQuery(current, parent) {
    current.addQuery('field', parent.field);
})(current, parent);
```

Add query conditions inside using `current.addQuery()`. The `current` variable represents the child table query, and `parent` represents the record on the form where the related list appears.

## Basic vs advanced fields

- **Basic:** Use `basic_apply_to` and `basic_query_from` with table name strings. Set `advanced: false`.
- **Advanced:** Use `apply_to` and `query_from` with script functions. Set `advanced: true`.
- Never mix basic and advanced fields in the same relationship record.

## Avoidance

1. **Never mix basic and advanced relationship fields** — use either `basic_apply_to`/`basic_query_from` or `apply_to`/`query_from`, not both.
2. **Never omit the wrapper in query_with scripts** — must follow the `(function refineQuery(current, parent) { ... })(current, parent);` format.
3. **Never hardcode sys_id strings in record references** — use record object variable references (`${record.$id}`) for portability across environments.
4. **Never create sys_ui_related_list_entry without a sys_ui_related_list** — the entry must reference a list container.

## Next Steps

- For Relationship API properties and code examples → retrieve RELATIONSHIP knowledge source via `get_knowledge_source`.
