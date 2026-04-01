---
name: cross-scope-privilege
description: "Guide for creating ServiceNow Cross-Scope Privileges [sys_scope_privilege] to manage cross-application script access. Use when user mentions cross-scope access, scope privileges, accessing tables or scripts from another scope, or debugging \"operation not allowed\" errors between scoped apps."
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


# Cross-Scope Privileges

## When to use this skill

- When your application needs to read, write, or execute resources from another application scope
- When debugging "operation against ... not allowed" errors at runtime
- When pre-authorizing cross-scope access so it works without manual admin approval
- When your scoped app needs to call script includes or access tables owned by another scope

## Instructions

1. **Identify the target resource:** Determine the exact table, script include, or scriptable object your app needs to access, and which scope owns it.
2. **Choose the right operation:** Tables support `read`, `write`, `create`, `delete`. Script includes and scriptable objects only support `execute`.
3. **Set status to `allowed`:** For pre-authorized access in your application package. Use `requested` if you want admin approval at install time.
4. **One privilege per operation per target:** Create separate `CrossScopePrivilege` records for each operation. If you need read and write on a table, create two records.
5. **Use the correct `targetType`:** `sys_db_object` for tables, `sys_script_include` for script includes, `scriptable` for scriptable objects.

## Key concepts

Cross-scope privileges declare that your application needs to access resources owned by a different application scope. Without these declarations, the platform blocks cross-scope operations at runtime.

### When cross-scope privileges are needed

- Your business rule queries a table owned by another scoped app
- Your script include calls a script include from another scope
- Your scheduled script updates records in another scope's table

### Status values

- **`allowed`** — Access is pre-authorized. The platform permits the operation without admin intervention.
- **`requested`** — Access requires admin approval after installation. Use when you can't guarantee the target scope will be present.
- **`denied`** — Explicitly blocks the operation. Rarely used in application code.

## Avoidance

- **Never use `execute` for table operations** — tables use `read`, `write`, `create`, `delete`; only script includes and scriptable objects use `execute`
- **Never assume cross-scope access works by default** — scoped apps are isolated; any cross-scope operation needs an explicit privilege
- **Avoid over-requesting** — only request the specific operations you actually need on each target

## Next Steps

- For Fluent API properties and code examples, use `get_knowledge_source` tool to get the CROSS_SCOPE_PRIVILEGE knowledge source.
- For table access configuration (`accessible_from`, `caller_access`), see the `table` skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
