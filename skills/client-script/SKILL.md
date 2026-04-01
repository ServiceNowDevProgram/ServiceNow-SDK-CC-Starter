---
name: client-script
description: Guide for creating ServiceNow Client Scripts [sys_script_client] that run JavaScript in the browser on form events. Use when user mentions client scripts, form behavior, field validation, dynamic forms, onLoad, onChange, onSubmit, or onCellEdit. Also use proactively when creating forms that need dynamic field behavior, client-side validation, or user interaction logic.
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


# Client Scripts

## When to use this skill

- When adding client-side behavior to forms (show/hide fields, set values, validate input)
- When responding to form events: load, submit, field change, or cell edit
- When configuring dynamic form behavior based on user interaction
- When adding client-side validation before form submission

## Instructions

1. **Choose the type first:** Select the correct event type — see guidance below.
2. **Use `Now.include` for scripts:** Reference external script files (`Now.include('../client/script.js')`) rather than inline scripts for anything non-trivial. This enables proper syntax highlighting and two-way sync with the platform.
3. **Set the `field` property for onChange/onCellEdit:** The `field` property is required when `type` is `onChange` or `onCellEdit` — it specifies which field triggers the script. It does not apply to `onLoad` or `onSubmit`.
4. **No module imports:** JavaScript module imports and third-party libraries are not supported in client scripts. Use only the `g_form` API and standard browser JavaScript.
5. **Table scoping:** Always use the full scoped table name.

## Key concepts

Client scripts run JavaScript on the client (web browser) when form-based events occur. They configure forms, form fields, and field values while the user is interacting with the form.

### Choosing the right type

- **`onLoad`** — Runs when the form is first displayed. Use for initial form setup: hiding fields, setting defaults, showing messages.
- **`onChange`** — Runs when a specific field value changes. Use for reactive behavior: updating one field based on another, showing/hiding sections based on selection. Requires the `field` property.
- **`onSubmit`** — Runs when the user submits the form. Use for validation: confirming required fields, checking data consistency, prompting confirmation. Return `false` to cancel submission.
- **`onCellEdit`** — Runs when a cell is edited in a list view. Use for inline list editing validation. Requires the `field` property.

### Script file pattern

Client scripts reference external files using `Now.include`:

```
script: Now.include('../client/my-script.js')
```

This enables syntax highlighting and two-way synchronization — changes from the platform UI sync to the file and vice versa.

## Avoidance

- **Never import JavaScript modules or third-party libraries** — client scripts do not support module imports
- **Never use `onChange` or `onCellEdit` without setting the `field` property** — the script won't know which field to watch
- **Avoid heavy logic in `onLoad`** — it runs every time the form opens and delays form display
- **Never assume `onSubmit` will always run** — programmatic saves or API calls may bypass client scripts

## Next Steps

- For Fluent API properties and code examples, use `get_knowledge_source` tool to get the CLIENT_SCRIPT knowledge source.
- For server-side record logic, see the `business-rule` skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
