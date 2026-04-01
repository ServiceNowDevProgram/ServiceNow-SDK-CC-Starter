---
name: table
description: Guide for creating ServiceNow Tables [sys_db_object], Columns [sys_dictionary], and Relationships [sys_relationship] to define data models. Use when user mentions tables, columns, fields, schema, extending tables, relationships, related lists, or data modeling. Also use proactively when building any application that needs to store data or connect tables.
user-invocable: true
---
> [!NOTE]
> This skill was authored for an agent runtime that provides two tools not
> available here. When you encounter references to these tools, resolve them
> as follows:
>
> **`get_knowledge_source`** — When instructed to use this tool with a
> knowledge source name like `BUSINESS_RULE`, read the file at
> `references/knowledge/<name>.md` where `<name>` is the lowercase,
> hyphenated version (e.g. `BUSINESS_RULE` →
> `references/knowledge/business-rule.md`).
>
> **`load_skill_resource`** — When instructed to use this tool with a file
> path like `references/column.md`, read that file directly from the
> `references/` directory within this skill.


# Tables, Columns, and Relationships

## When to use this skill

- When creating new tables to store application data
- When adding columns (fields) to new or existing tables
- When extending an existing table (e.g., extending `task`)
- When setting up relationships between tables or configuring related lists
- When defining choices, defaults, or dynamic values for fields

## Instructions

### Creating a new table

1. **Table naming:** The table name must start with the application scope prefix (e.g., `x_acme_my_table`). The exported variable name MUST match the `name` property exactly — this is required for proper functionality and typeahead support.
2. **Choose column types carefully:** Only use supported column types (see COLUMN knowledge source for the full list). Only import the types you use to avoid build errors. Common types: `StringColumn`, `IntegerColumn`, `BooleanColumn`, `ReferenceColumn`, `DateTimeColumn`, `DateColumn`, `ChoiceColumn`. See `references/column.md` for detailed type selection guidance.
3. **Extending tables:** Use the `extends` property to inherit all fields from a base table (e.g., `extends: 'task'`). Only extend tables marked as extensible. Extending creates system fields automatically.
4. **Cross-scope access:** Set `accessible_from`, `caller_access`, and `actions` based on whether other scoped apps need access. Default to `package_private` unless public access is needed.

### Adding columns to an existing table

5. **Different scope pattern:** When adding columns to a table in a different scope, provide the table name without the scope prefix followed by `as any`. Prefix column names with your application scope instead.
6. **Import only needed types:** Import only the column types you actually use from `@servicenow/sdk/core` — unused imports cause build errors.

### Setting up relationships and related lists

7. **Load detailed instructions:** Relationships involve multiple records and decision paths. Load `references/relationship.md` using the `load_skill_resource` tool for full instructions on implicit, explicit, and platform relationship patterns.

## Key concepts

- **Tables** define the data model. **Columns** define the fields. **Relationships** connect tables and display related records as lists on forms.
- **Column type selection:** Use `StringColumn` for text, `IntegerColumn` for numbers, `BooleanColumn` for flags, `ReferenceColumn` for foreign keys, `DateColumn`/`DateTimeColumn` for dates, `ChoiceColumn` for dropdowns. For dropdowns on string fields, use `StringColumn` with `choices` and `dropdown` properties.
- **Implicit vs explicit relationships:** If a reference field exists between tables, the relationship is implicit (no extra record needed). If not, create an explicit `sys_relationship` record.

## Avoidance

1. **Never mismatch the exported variable name and the `name` property** — they must be identical for the table to work correctly.
2. **Never use unsupported column types** — only use types explicitly listed in the COLUMN knowledge source; unsupported types cause build errors.

## Next Steps

- For Table API properties and examples → retrieve TABLE knowledge source.
- For Column types and field properties → retrieve COLUMN knowledge source.
- For column type selection guidance → load `references/column.md` via `load_skill_resource`.
- For relationship and related list setup → load `references/relationship.md` via `load_skill_resource`.
- For Relationship API properties and examples → retrieve RELATIONSHIP knowledge source.
- For securing tables with ACLs → activate **implementing-security** skill.
- For server-side record logic on tables → activate **business-rule** skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
