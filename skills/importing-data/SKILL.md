---
name: importing-data
description: Guide for importing external data into ServiceNow using Data Sources [sys_data_source], staging tables, Import Sets, and Transform Maps. Supports CSV, Excel, JSON, XML, JDBC, LDAP, and REST data sources. Use when user mentions importing data, data sources, import sets, transform maps, staging tables, data integration, CSV/Excel/JSON/XML import, JDBC connections, LDAP imports, external data, or bulk imports. Also use proactively when building applications that need to ingest data from external systems. Supported in SDK 4.2.0+.
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


## When to use

Use this skill when importing external data into ServiceNow using data sources, staging tables, and transform maps.

## Instructions

### Three-component pattern

1. Every data import requires three separate components created in order:
   - **Staging Table** (Table API): extends `sys_import_set_row`, defines all columns for imported data. Must be created FIRST.
   - **Data Source** (Record API on `sys_data_source`): references the staging table by name in `import_set_table_name`. Defines how to connect and load data. Must be created SECOND.
   - **Import Set / Transform Map** (ImportSet API): references the staging table by name in `sourceTable`. Defines how to map data from staging to target. Must be created THIRD.
2. All three components MUST use the exact same staging table name.

### Discovery workflow (before creating data sources)

3. Before creating any data source, search `sys_data_source` for existing connections that match the user's needs (same database, same LDAP server, same file location, etc.).
4. Present findings to the user and ask them to choose: reuse an existing data source or create a new one.
5. If reusing an existing source, reference its `sys_id` in the transform map — no new data source code needed.
6. If creating new, proceed with code generation.

### Collecting mandatory fields

7. Before generating data source code, collect format-specific mandatory fields from the user:
   - **XML format**: ask for `xpath_root_node` (e.g., `//product`, `/root/items/item`)
   - **JSON format**: ask for `jpath_root_node` (e.g., `$.employees[*]`, `$.data.records`)
   - **JDBC format**: requires either `table_name` or `sql_statement`
   - **LDAP format**: requires complete LDAP chain (see below)
   - **REST format**: ask for `request_action` — search `sys_hub_action_type_definition` where `action_template='DATASOURCE_REQUEST'`
   - **CSV/Excel**: no format-specific mandatory fields
8. Do not generate data source code until all mandatory fields are collected.

### Password handling

9. Pre-populate all configuration fields (hostnames, ports, usernames, database names, file paths) with user-provided values.
10. Leave password fields empty (`''`) with a `// LEAVE EMPTY` comment — passwords are set manually in ServiceNow after deployment.
11. After generating code, include manual password setup instructions specific to the data source type (JDBC, LDAP, or FTP/SCP/SFTP).

### LDAP data sources

12. LDAP imports require a chain of records created in order: `ldap_server_config` → `ldap_ou_config` (references server) → optionally `ldap_server_url` (references server) → `sys_data_source` (references OU via `ldap_target`).
13. Use record object references (not hardcoded `sys_id` strings) for LDAP cross-table references.

### Transform map configuration

14. Set coalesce fields on at least one field to prevent duplicate records during import. Coalesce fields are used for record matching — if a matching record exists, it's updated instead of creating a duplicate.
15. Use simple string mappings (`target_field: 'source_field'`) for direct field copies. Use object mappings for coalesce, date format, reference fields, or source scripts.
16. For complex transformation logic (>10-15 lines), extract to TypeScript functions in server modules instead of inline strings.
17. Transform scripts support these execution points: `onBefore`, `onAfter`, `onReject`, `onStart`, `onForeignInsert`, `onComplete`, `onChoiceCreate`.

## Key concepts

- **Data source types**: File (CSV, Excel, JSON, XML), JDBC (database), LDAP, REST (Integration Hub), Custom (Parse by Script)
- **File retrieval methods**: Attachment (direct upload), FTP, SCP, SFTP, HTTP, HTTPS
- **Import flow**: External Source → Data Source → Staging Table → Transform Map → Target Table
- **Coalesce**: Field-level setting that enables record matching — prevents duplicates by updating existing records instead of inserting new ones
- **enforceMandatoryFields**: Controls mandatory field validation during import: `no`, `onlyMappedFields`, `allFields`
- **runBusinessRules**: Disable for bulk imports (performance); enable selectively for business logic validation

## Avoidance

1. Do not use Script Include or Scripted REST API for data import — use Data Source + ImportSet instead.
2. Do not create a data source without first searching for existing ones (discovery workflow).
3. Do not generate XML/JSON data source code without collecting `xpath_root_node`/`jpath_root_node` from the user — the data source will deploy but fail to import data.
4. Do not create an ImportSet without a corresponding Data Source and Staging Table — all three components are required.
5. Do not use mismatched table names between Data Source (`import_set_table_name`) and ImportSet (`sourceTable`).
6. Do not hardcode `sys_id` strings for LDAP references — use record object references.
7. Do not create LDAP data sources without the complete chain (server config → OU config → data source).
8. Do not include passwords in generated code — leave empty for manual configuration.

## Next Steps

- For transform map field mappings and scripts → retrieve `IMPORT_SET` knowledge source
- For data source configuration by type (File, JDBC, LDAP, REST) → retrieve `SYS_DATA_SOURCE` knowledge source
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
