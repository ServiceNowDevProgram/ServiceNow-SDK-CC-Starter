---
name: module
description: Guide for creating JavaScript modules [sys_module] to organize reusable server-side code in ServiceNow applications. Use when user mentions modules, imports, exports, code organization, third-party libraries, or @servicenow/glide. Also use proactively when writing server-side scripts that need shared utility functions or Glide API access.
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


# JavaScript Modules

## When to use this skill

- When organizing reusable server-side code into importable files
- When importing Glide APIs (gs, GlideRecord, etc.) for use in modules
- When adding third-party npm libraries to an application
- When creating Script Include classes in module files

## Instructions

1. **Import Glide APIs explicitly:** In module files, `gs`, `GlideRecord`, and other Glide APIs are NOT automatically available. You must import them from `@servicenow/glide`. Analyze your script for ALL ServiceNow APIs used and import each one.
2. **Exception — Script Include classes:** When writing Script Include class files (Class.create pattern), do NOT import Glide APIs. They are automatically available in Script Include execution context. Only import other Script Include classes from `@servicenow/glide/<scopeName>`.
3. **Use `export`/`import` in modules, `require` in scripts:** Modules use ES module syntax (`export`/`import`). Business rules, script includes, and other server scripts use `require` to consume module exports.
4. **Declare dependencies in package.json:** Third-party npm libraries must be declared in `dependencies`. Never modify versions of existing dependencies unless explicitly requested.
5. **Verify Glide API methods exist:** Only use methods explicitly defined in `@servicenow/glide` type definitions. Do not assume methods exist based on naming conventions. If uncertain, flag it rather than guess.
6. **Use `GlideDateTime` instead of `gs.nowDateTime()`** — `gs.nowDateTime()` is not allowed in scoped applications.

## Key concepts

JavaScript modules let you organize reusable code in files that can be shared across your application. They are stored in the `sys_module` table and support ES module syntax.

### Import patterns

- **Glide APIs in modules:** `import { gs, GlideRecord } from '@servicenow/glide'`
- **Namespaced APIs:** `import { RESTAPIRequest } from '@servicenow/glide/sn_ws_int'`
- **Script Include classes:** `import { MyClass } from '@servicenow/glide/x_my_scope'`
- **Module code in scripts:** `const { myFunction } = require('path/to/module')`

### Script Include module rules

This is the most common source of errors:

- **Module files with normal functions** → MUST import Glide APIs from `@servicenow/glide`
- **Module files with Script Include classes (Class.create)** → must NOT import Glide APIs (they're auto-available)
- **Consuming Script Include classes from other modules** → import from `@servicenow/glide/<scopeName>`

### Limitations

- Modules work only within the application scope — no cross-scope module sharing
- Node.js APIs are not supported
- Third-party libraries cannot access ServiceNow APIs
- CommonJS modules from third-party libs aren't supported unless they define exports
- Only a subset of ECMAScript features are supported

## Avoidance

- **Never use Glide APIs without importing them in module files** — they are NOT globally available in module context
- **Never import Glide APIs in Script Include class files** — they ARE globally available in that context
- **Never use methods not in `@servicenow/glide` type definitions** — ServiceNow's Glide objects have specific, limited APIs; don't assume methods exist
- **Never modify existing dependency versions in package.json** — only add new dependencies
- **Never use `gs.nowDateTime()` in scoped apps** — use `new GlideDateTime().getDisplayValue()` instead

## Next Steps

- For Fluent API reference and import/export syntax examples, use `get_knowledge_source` tool to get the MODULE knowledge source.
- For script include class patterns, see the `script-include` skill.
- For business rule scripts that consume modules, see the `business-rule` skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
