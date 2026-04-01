---
name: script-include
description: Guide for creating ServiceNow Script Includes [sys_script_include] to bundle reusable server-side logic into classes or utilities. Use when user mentions script includes, reusable server code, GlideAjax, client-callable scripts, or shared utility functions. Also use proactively when creating server-side logic that needs to be called from multiple places or from client-side code.
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


# Script Includes

## When to use this skill

- When creating reusable server-side logic (utility classes, helper functions)
- When building server-side APIs callable from client code via GlideAjax
- When sharing business logic across multiple business rules, scheduled scripts, or other server scripts
- When creating AJAX processors for UI Pages or client scripts

## Instructions

1. **Use `Now.include` for scripts:** Keep JavaScript in a standalone file under `src/server/script-includes/` and reference it with `Now.include('../../server/script-includes/file.js')`. This enables syntax highlighting and two-way sync.
2. **Use ES5 syntax:** Use the classic `Class.create()` pattern or plain functions. ES6 `class` syntax is not supported on the ServiceNow platform for script includes.
3. **Match `type` to class name:** In the `Class.create()` prototype, the `type` property must exactly match the class name and the `name` property in the Fluent definition. Mismatches cause runtime failures.
4. **Set `clientCallable` for GlideAjax:** If client-side code (UI Pages, client scripts) needs to call the script include, set `clientCallable: true`. Without this, GlideAjax calls will be rejected.
5. **Extend `AbstractAjaxProcessor` for AJAX:** When the script include is called via GlideAjax, extend `global.AbstractAjaxProcessor` and use `this.getParameter()` to read client parameters.
6. **Scope accessibility:** Set `accessibleFrom: 'public'` only when other scoped apps need access. Default to `'package_private'` for internal use.

## Key concepts

Script includes are server-side JavaScript that bundle reusable business logic into classes or utilities. They can be invoked from business rules, scheduled scripts, other script includes, and client-side code (via GlideAjax).

### Class-based vs classless

- **Class-based** (most common): Use `Class.create()` with a prototype. Best for grouping related methods (e.g., a utility class with multiple operations). The `type` property in the prototype must match the class name.
- **Classless (on-demand)**: A single function that runs when called by name. Best for one-off utility functions. The function name must match the `name` property.

### GlideAjax pattern

To make a script include callable from client-side code:

1. Set `clientCallable: true` in the Fluent definition
2. Extend `global.AbstractAjaxProcessor` in the JavaScript
3. Use `this.getParameter('sysparm_name')` to read parameters passed from the client
4. Return data as JSON strings for structured responses

### Project structure

```
src/
  server/
    script-includes/
      my-utils.js           ← JavaScript business logic
  fluent/
    script-includes/
      my-utils.now.ts       ← Fluent record definition
```

## Avoidance

- **Never use ES6 `class` syntax** — the ServiceNow server runtime does not support it for script includes
- **Never mismatch `type` and class name** — the `type` property in the prototype, the `Class.create()` variable name, and the `name` in the Fluent definition must all match exactly
- **Never forget `clientCallable: true` for GlideAjax** — client calls will silently fail without it
- **Avoid inline scripts** for anything beyond a one-liner — use `Now.include()` for maintainability and syntax highlighting

## Next Steps

- For Fluent API properties and code examples, use `get_knowledge_source` tool to get the SCRIPT_INCLUDE knowledge source.
- For calling script includes from forms, see the `client-script` skill.
- For server-side record event logic, see the `business-rule` skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
