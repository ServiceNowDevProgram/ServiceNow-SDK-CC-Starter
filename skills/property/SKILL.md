---
name: property
description: Guide for creating ServiceNow System Properties [sys_properties] to store application configuration values. Use when user mentions properties, system settings, configuration values, feature flags, or application preferences. Also use proactively when building applications that need configurable behavior without code changes.
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


# System Properties

## When to use this skill

- When adding configurable settings to an application (feature flags, thresholds, URLs)
- When storing values that admins should be able to change without modifying code
- When creating properties that business rules, scheduled scripts, or client code read at runtime

## Instructions

1. **Name with scope prefix:** Property names must start with the application scope (e.g., `x_snc_example.my_setting`). Use dot notation to organize hierarchically (e.g., `x_snc_example.email.enabled`, `x_snc_example.email.sender`).
2. **Choose the right type:** Use `string` for text, `integer` for numbers, `boolean` for flags, `choicelist` for constrained options, `password`/`password2` for secrets. The `value` field type must match.
3. **Set access roles:** Use the `roles` property to control who can read and write the property. Restrict write access to admin roles for sensitive settings.
4. **Use `isPrivate` for sensitive data:** Set `isPrivate: true` for properties that should not be visible in the property list UI (e.g., API keys, internal thresholds).
5. **Provide a description:** Always add a `description` explaining what the property controls and what valid values are — this is what admins see when configuring the app.

## Key concepts

System properties store key-value configuration for an application. They are read at runtime using `gs.getProperty('name')` in server-side scripts. Properties let admins tune application behavior without deploying code changes.

### When to use properties vs other patterns

- **Properties** — For values that change between environments or that admins tune (timeouts, feature flags, URLs).
- **Business rules** — For logic that responds to data changes. Don't store logic in properties.
- **Script includes** — For reusable code. Properties store data, not behavior.

### Caching

Properties are cached by default for performance. Set `ignoreCache: true` only for properties that must reflect changes immediately (rare). Most properties should keep the default (`false`).

## Avoidance

- **Never omit the scope prefix from the name** — properties without scope prefix will fail validation
- **Never store large data in properties** — they are for simple configuration values, not data blobs
- **Never use `ignoreCache: true` unless necessary** — it forces a database read on every `gs.getProperty()` call, impacting performance

## Next Steps

- For Fluent API properties and code examples, use `get_knowledge_source` tool to get the PROPERTY knowledge source.
- For role definitions referenced by property access control, see the `implementing-security` skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
