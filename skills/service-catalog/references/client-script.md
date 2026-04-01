# Catalog Client Script Reference

For properties and templates, use `get_knowledge_source` with knowledge source `CATALOG_CLIENT_SCRIPT`.

## Script Types

### onLoad

Runs when the form loads. Use for initial setup (field states, defaults, visibility).

### onChange

Runs when a specific variable changes. **Always guard with `if (isLoading) return;`** to prevent execution during form load.

### onSubmit

Runs on form submission. Return `false` to block submission. **Avoid GlideAjax here** — async calls won't complete before the form submits.

---

## g_form API Reference

| Method                                               | Description                    |
| ---------------------------------------------------- | ------------------------------ |
| `getValue(fieldName)`                                | Get variable value             |
| `setValue(fieldName, value)`                         | Set variable value             |
| `setDisplay(fieldName, display)`                     | Show/hide variable             |
| `setMandatory(fieldName, mandatory)`                 | Set mandatory state            |
| `setReadOnly(fieldName, readOnly)`                   | Set read-only state            |
| `clearValue(fieldName)`                              | Clear variable value           |
| `hasField(fieldName)`                                | Check if field exists          |
| `showFieldMsg(fieldName, message, type, scrollForm)` | Show field message             |
| `hideFieldMsg(fieldName, clearAll)`                  | Hide field message             |
| `addErrorMessage(message)`                           | Add banner error message       |
| `clearOptions(fieldName)`                            | Clear all select options       |
| `addOption(fieldName, value, label)`                 | Add a select option            |
| `getReference(fieldName, callback)`                  | Get referenced record (legacy) |

> **Note on `getReference`**: Legacy convenience method. Works for simple lookups but `GlideAjax` is preferred for complex server-side logic. May make synchronous calls in some versions, which can freeze the UI.

---

## GlideAjax

Use `GlideAjax` to call server-side Script Includes from client scripts. The client sends a request, the Script Include processes it, and returns a result via a callback.

### GlideAjax Types — Quick Decision Guide

| Method           | Execution | Use When                                         | Avoid When                              |
| ---------------- | --------- | ------------------------------------------------ | --------------------------------------- |
| `getXMLAnswer()` | **Async** | Simple lookups, returning a single value/string  | You need the full XML response object   |
| `getXML()`       | **Async** | Need full XML response, complex response parsing | Simple value returns (use getXMLAnswer) |
| `getXMLWait()`   | **Sync**  | Almost never — legacy/global scope only          | Scoped apps, any production code        |

### GlideAjax Parameter Rules

All custom parameters **must** start with `sysparm_`. The first `addParam` call must always be `sysparm_name` with the method name.

```javascript
ga.addParam("sysparm_name", "methodName"); // REQUIRED: always first
ga.addParam("sysparm_user_id", userSysId); // Custom param: prefix with sysparm_
ga.addParam("sysparm_category", selectedCat); // Custom param: prefix with sysparm_
```

---

## Script Include (Server-Side Companion)

Every `GlideAjax` call requires a corresponding **Script Include** on the server. The Script Include must extend `AbstractAjaxProcessor` and be marked **Client Callable**.

### Script Include Setup Checklist

| Property        | Value                                                                    |
| --------------- | ------------------------------------------------------------------------ |
| Name            | Must match the string in `new GlideAjax('ClassName')`                    |
| Client callable | **Checked** (required for GlideAjax access)                              |
| Extends         | `global.AbstractAjaxProcessor`                                           |
| Retrieve params | Use `this.getParameter('sysparm_param_name')`                            |
| Return data     | Use `return` (simple string) or `return JSON.stringify(obj)` for objects |

### Script Include Security

- **Client callable = true**: Only check this when the method is explicitly needed from the client.
- **Access controls**: Runs in the logged-in user's session context. ACLs still apply to GlideRecord queries.
- **Input validation**: Always validate parameters from `this.getParameter()`. Never trust client-side input.

---

## Scripts on Variable Sets

Scope scripts to a variable set using `variableSet` and `appliesTo: 'set'` so they apply to **all** catalog items using that set. Always use `hasField()` checks since the variable may not exist on every item that includes the set.

### Execution Order Note

When multiple variable sets are attached to a catalog item, scripts execute in the order the variable sets are listed on the item. If both a variable set script and an item-level script target the same variable, the item-level script runs last and takes precedence.

---

## Catalog Client Script vs Standard Client Script

| Aspect          | Catalog Client Script                      | Standard Client Script               |
| --------------- | ------------------------------------------ | ------------------------------------ |
| Scope           | Catalog item or variable set               | Table (e.g., Incident)               |
| onChange target | Links to a **variable**                    | Links to a **field**                 |
| Context         | Catalog ordering, RITM, Catalog Task forms | Table forms                          |
| Variable access | Direct by variable name                    | Use `variables.variable_name` prefix |
| Applies to      | `item` or `set`                            | Specific table                       |

---

## Best Practices

- **Guard onChange**: Always start with `if (isLoading) return;` to prevent execution on form load.
- **Use `hasField()` in variable set scripts**: The variable may not exist on every catalog item using the set.
- **Avoid GlideAjax in onSubmit**: Async calls won't complete before submission. Use onSubmit only for synchronous client-side validation.
- **Use `g_form` APIs, not DOM**: Never use `document.getElementById()` or jQuery — it breaks across UI versions (UI16, Workspace, Portal).
- **Prefix for conflicts**: If a variable name conflicts with a table field name, use `variables.variable_name` to reference the catalog variable.
- **Return JSON from Script Includes**: Use `JSON.stringify()` on the server and `JSON.parse()` on the client for structured data.
- **Validate inputs in Script Includes**: Never trust client-side parameters. Check for empty values, validate sys_id formats, and handle edge cases.
- **Use `Now.include()` for shared code**: Extract reusable client logic into UI Scripts and load with `Now.include('script_name')` for maintainability.
- **Prefer `getXMLAnswer` over `getXML`**: Simpler callback, less parsing code. Use `getXML` only when you need the full XML response.
- **Never use `getXMLWait()`**: Synchronous, freezes the UI, not available in scoped apps. Always use async patterns.

For complete examples and templates, use `get_knowledge_source` with knowledge source `CATALOG_CLIENT_SCRIPT`.
