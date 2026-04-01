---
name: application-menu
description: Guide for creating ServiceNow Application Menus [sys_app_application] and Modules [sys_app_module] for the application navigator. Use when user mentions navigation menus, application navigator, sidebar menus, module links, or menu categories. Also use proactively when building applications that need navigator entries.
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


# Application Menus

## When to use this skill

- When adding an application to the ServiceNow navigator ("All" menu)
- When creating menu items that link to tables, UI pages, reports, or URLs
- When organizing navigation with separators and sub-menus
- When restricting menu visibility to specific roles

## Instructions

1. **Create both parts:** An application menu requires two records: an `ApplicationMenu` for the top-level entry in the navigator, and one or more `Record` entries in `sys_app_module` for the child menu items. Neither works alone.
2. **Link modules to their menu:** Set `application: applicationMenu.$id` on each module Record to associate it with the parent menu.
3. **Choose the right `link_type`:** This determines what the module links to — see guidance below.
4. **Set `order` for positioning:** Lower numbers appear higher in the navigator. Use increments of 100 to leave room for future items.
5. **Restrict with roles:** Set `roles` on both the ApplicationMenu (array of Role objects or names) and modules (comma-separated role names) to control visibility.

## Key concepts

The application navigator is the left sidebar in ServiceNow. Application menus create top-level sections, and modules create clickable items within them.

### Common link_type values

- **`LIST`** — Shows a list of records from a table. Set `name` to the table name.
- **`NEW`** — Opens a new record form. Set `name` to the table name.
- **`DIRECT`** — Links to a URL. Set `query` to the relative path (e.g., `my_page.do`).
- **`SEPARATOR`** — Creates a sub-folder/divider to group subsequent modules.
- **`REPORT`** — Links to a report. Set `report` to the report identifier.
- **`FILTER`** — Shows a filtered list. Set `name` to the table and `filter` to the encoded query.

### Menu categories

Use the Record API with table `sys_app_category` to define a category with custom styling. Reference the category from the `ApplicationMenu` to apply a visual style to the menu section.

## Avoidance

- **Never create modules without a parent ApplicationMenu** — orphaned modules won't appear in the navigator
- **Never forget to set `active: true` on modules** — modules default to `active: false` and won't appear
- **Avoid hardcoding sys_ids in `query` or `filter`** — use encoded queries or relative references instead

## Next Steps

- For Fluent API properties and code examples, use `get_knowledge_source` tool to get the APPLICATION_MENU knowledge source.
- For creating sub-menu items with more link types, see the `module` skill.
- For role-based access control, see the `implementing-security` skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
