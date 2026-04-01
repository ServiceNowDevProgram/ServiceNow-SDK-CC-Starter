---
name: record
description: "Guide for using the ServiceNow Record API to add data to any table, including tables without a dedicated Fluent API. Use when user mentions inserting records, seed data, demo data, menu categories, or adding rows to platform tables. Also use proactively when populating custom tables or creating metadata records that don't have their own API."
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


# Record API

## When to use this skill

- When inserting seed or demo data into custom or platform tables
- When creating metadata records that have no dedicated Fluent API (e.g., `sys_app_category`)
- When populating a custom table defined with the Table API
- When adding records to platform tables like `incident`, `cmdb_ci_server`, etc.

## Instructions

1. **Use Record for any table:** The Record API is a generic escape hatch — it can insert a row into any table by name. Use it when no dedicated Fluent API exists for that table.
2. **Always set `$id` and `table`:** Both are required. `$id` uses `Now.ID[<value>]` format and gets hashed into a sys_id at build time.
3. **Match `data` fields to the table schema:** Field names in `data` must match the column names on the target table. Use the correct value types (strings for references, booleans, numbers).
4. **Use `Now.include` for file content:** For fields that hold scripts or CSS, reference external files with `Now.include('path/to/file')` rather than inlining large content.
5. **Use `$meta.installMethod` for conditional loading:** Set `installMethod: 'demo'` for demo data (loaded only when "Load demo data" is selected) or `installMethod: 'first install'` for data loaded only on first install.
6. **Use date format conventions:** `DateColumn` expects `yyyy-mm-dd`. `DateTimeColumn` expects `yyyy-mm-dd HH:mm:ss`.

## Key concepts

The Record API is the generic fallback for inserting data into any ServiceNow table. While dedicated APIs like `Table`, `BusinessRule`, or `ApplicationMenu` handle specific record types with validation and structure, the Record API handles everything else.

### Common uses

- **Seed data** — Pre-populate custom tables with initial records
- **Menu categories** — Create `sys_app_category` records for application navigator styling
- **Application modules** — Create `sys_app_module` records as child items of an ApplicationMenu
- **Platform records** — Insert into `incident`, `cmdb_ci_server`, or any other platform table

### Relationship to other APIs

The Record API complements — not replaces — dedicated APIs. Prefer `Table()` for creating tables, `BusinessRule()` for business rules, etc. Use `Record()` only when no dedicated API covers your target table.

## Avoidance

- **Never omit `$id` or `table`** — both are required; missing either causes a build error
- **Never use field names that don't exist on the target table** — the record will fail silently or error at deploy time
- **Never hardcode sys_ids for references when a `Now.ID` reference is available** — use `Now.ID` references to other records in the same app instead
- **Never inline large scripts in `data`** — use `Now.include('path/to/file')` for script or CSS content

## Next Steps

- For Fluent API properties and code examples, use `get_knowledge_source` tool to get the RECORD knowledge source.
- For creating tables that records are inserted into, see the `table` skill.
- For application navigator records, see the `application-menu` skill.
- For script includes, see the `script-include` skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
