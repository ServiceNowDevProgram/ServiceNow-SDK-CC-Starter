---
name: business-rule
description: Guide for creating ServiceNow Business Rules [sys_script] to run server-side logic on record operations. Use when user mentions business rules, server-side scripts, record triggers, auto-populating fields, data validation, cascading updates, or audit logging. Also use proactively when creating tables that need automated behavior on insert, update, delete, or query.
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


# Business Rules

## When to use this skill

- When creating server-side logic that runs automatically on record operations
- When auto-populating or transforming field values on insert or update
- When validating data before a record is saved
- When cascading changes to related records after an operation
- When restricting or filtering queries server-side
- When logging or auditing record changes

## Instructions

1. **Naming:** Use descriptive names for your business rules.
2. **Timing first:** Choose the correct `when` value before writing any logic. This is the most critical decision — see guidance below.
3. **Scope the action:** Only subscribe to the actions you need (insert, update, delete, query). Never use all four unless truly required.
4. **Scripts in modules:** For anything beyond a one-liner, import functions from JavaScript modules (`src/server/`). Keep inline scripts for simple field sets only.
5. **Table scoping:** Always use the full scoped table name (e.g., `x_myapp_tablename`).
6. **Order matters:** Set an appropriate execution order for rules that may interact. Lower numbers run first. Default is 100.
7. **Conditions over scripts:** Use the most specific conditions possible for better performance. Prefer `filterCondition` or `condition` to limit when the rule fires rather than putting guard clauses inside the script. The platform evaluates conditions before loading the script.
8. **Role conditions:** Consider role-based conditions to limit execution scope.
9. **Messages:** Add descriptive messages when appropriate for user feedback.

## Key concepts

Business rules are server-side scripts that automatically run when certain conditions are met during record operations. They can be used to automatically change values in form fields, validate data, trigger notifications, and more.

### Choosing the right timing

Use `before` when modifying the current record before save or validating/aborting. Use `after` when you need the final saved state or need to update related records. Use `async` for expensive operations that shouldn't block the user. Use `display` for calculated values shown on the form but not stored.

### Before vs After

- `before` rules can modify `current` and the changes persist — the record hasn't been written yet
- `after` rules cannot modify `current` effectively — the record is already saved. To change fields after save, you must do a separate GlideRecord update
- `before` rules that set `abortAction: true` will prevent the record operation entirely
- `after` rules cannot abort — the operation has already completed

### Query business rules

Business rules with `action: ["query"]` run every time the table is queried. They modify the query itself (e.g., adding conditions to restrict results). Use sparingly — they run on every query and can cause significant performance impact.

## Avoidance

- **Never use `async` for logic that must complete before the user sees the result** — async rules run in a separate transaction with no guarantee of timing
- **Never modify `current` in an `after` rule** expecting it to save — it won't
- **Never use `query` action without careful consideration** — it runs on every single query against the table, including system queries
- **Never use `before` for operations that depend on the record's sys_id** — on insert, the sys_id may not be finalized until after the save
- **Avoid broad actions** — subscribing to all actions when you only need one wastes execution cycles

## Next Steps

- For Fluent API properties and code examples, use `get_knowledge_source` tool to get the BUSINESS_RULE knowledge source.
- For script organization patterns, see the `module` skill.
- For securing tables with business rules, see the `implementing-security` skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
