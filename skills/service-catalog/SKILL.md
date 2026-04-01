---
name: service-catalog
description: Build ServiceNow Service Catalog components using Fluent API — catalog items [sc_cat_item], record producers [sc_cat_item_producer], variables [item_option_new], variable sets [item_option_new_set], UI policies [catalog_ui_policy], and client scripts [catalog_script_client]. Use when user mentions service catalog, catalog items, record producers, variables, variable sets, UI policies, client scripts, form validation, dynamic fields, onChange, onLoad, onSubmit, ordering, self-service, or any catalog-related development. Includes property references, code examples, field mapping, scripting patterns, and best practices for end-to-end Service Catalog configuration. Only supported in SDK 4.3.0 or higher.
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


## When to use this skill

- When creating **catalog items** for ordering goods/services
- When creating **record producers** for task record creation (incidents, changes, problems)
- When defining **catalog variables** (form fields) for user input
- When creating **variable sets** for reusable variable groups
- When implementing **catalog UI policies** for show/hide, mandatory, read-only, simple value setting.
- When adding **catalog client scripts** for complex validation, dynamic calculations, API/async calls (GlideAjax), form submission control (onSubmit).
- When user mentions "service catalog", "catalog item", "record producer", "variables", "UI policy", "client script", "onChange", "onLoad", "onSubmit", or "form validation"

## Instructions

1. **Catalog, Category & Taxonomy:** Items must be assigned to at least one catalog and category, and optionally a taxonomy topic. Use queries to find existing sys_ids.
2. **Variable Naming:** Use `snake_case` for variable names. Use `order` increments of 100.
3. **Record Producer Tables:** Only use for task-based tables (incident, change_request, problem). Never for sc_req_item, sc_request, sc_task.
4. **Field Mapping:** Use `mapToField: true` for simple mappings, scripts for complex logic.
5. **UI Policy vs Client Script:** Use UI policies for simple show/hide/mandatory. Use client scripts for validation, calculations, async calls.
6. **onChange Guard:** Always start onChange scripts with `if (isLoading) return;`.
7. **onSubmit:** Avoid GlideAjax in onSubmit (async issues). Return `false` to block submission.
8. **Variable References:** Use object references in properties (e.g., `catalogItem.variables.urgency`), strings inside script code (e.g., `g_form.getValue('urgency')`).
9. **Variable Sets:** Use for reusable variable groups. UI policies and client scripts can be scoped to a variable set with `appliesTo: 'set'`.
10. **DOM Manipulation:** Never manipulate DOM directly — always use `g_form` API.
11. **Variable Name Conflicts:** Don't use the same variable name as a target table field name.
12. **Record Producer Scripts:** Never call `current.update()` or `current.insert()` in pre-insert script.
13. **Circular Dependency (Flow + CatalogItem):** When a flow uses `getCatalogVariables` with a catalog item's variables, the flow file imports the CatalogItem, and the CatalogItem references the flow using `Now.ref()` (NO import) to break the cycle.

## Key Concepts

### Catalog Item vs Record Producer

| Aspect          | Catalog Item                             | Record Producer                                         |
| --------------- | ---------------------------------------- | ------------------------------------------------------- |
| **Creates**     | REQ + RITM + Fulfillment Tasks           | Record in target table (incident, change_request, etc.) |
| **Fulfillment** | Flow Designer / Workflow / Delivery Plan | Server-side scripts                                     |
| **Use when**    | Ordering goods/services with approvals   | Creating task records directly                          |
| **Examples**    | "Request Laptop", "Software License"     | "Report Incident", "Submit HR Case"                     |

**Key Rule:** Ordering/requesting something → Catalog Item. Creating a task record → Record Producer.

### Taxonomy & Access (Catalog Items & Record Producers)

**Taxonomy** (`taxonomy_topic`): Hierarchical classification on catalog items. Organizes items from broad categories to specific subcategories, improving searchability and navigation — particularly in Employee Center, where it maps items to topics and appears above the item name in search results. In Employee Center contexts, taxonomy often serves as the primary organization method, replacing traditional category structures. Assign topics to a catalog item using the `assignedTopics` property.

**Catalog & Category Assignment**: Items must belong to at least one Catalog (`sc_catalog`) and Category (`sc_category`). Categories can be nested into subcategories. Items can appear in multiple catalogs and categories simultaneously.

**Visibility**: Controlled via user criteria on the catalog item: `availableFor` grants access, `notAvailableFor` restricts it. `notAvailableFor` always overrides `availableFor` when both are present.

### Agent Decision Tree

When user requests a Service Catalog component:

1. Ordering goods/services → Catalog Item with variables and Flow Designer
2. Creating task records (incident, change, problem) → Record Producer with field mapping
3. Reusable form fields across items → Variable Set (singleRow or multiRow)
4. Simple show/hide/mandatory logic → Catalog UI Policy
5. Complex validation, calculations, async calls → Catalog Client Script
6. Grid/table data entry → Multi-Row Variable Set (MRVS)

### UI Policy vs Client Script

| Use Case                 | UI Policy     | Client Script |
| ------------------------ | ------------- | ------------- |
| Show/hide variables      | **Preferred** | Supported     |
| Make variables mandatory | **Preferred** | Supported     |
| Make variables read-only | **Preferred** | Supported     |
| Set variable values      | Supported     | Supported     |
| Complex validation       | Limited       | **Preferred** |
| Dynamic calculations     | Limited       | **Preferred** |
| API calls / async        | Not supported | Supported     |
| Form submission control  | Not supported | Supported     |

### Common Validation Scenarios

| Validation                      | Implementation                         | Script Type          |
| ------------------------------- | -------------------------------------- | -------------------- |
| No past dates                   | Client Script                          | onChange             |
| Date range (start < end)        | Client Script                          | onChange             |
| Min/max numeric values          | Client Script                          | onChange             |
| Text min/max length             | Client Script                          | onSubmit             |
| Format validation (regex)       | Client Script                          | onChange or onSubmit |
| Required based on another field | UI Policy (preferred) or Client Script | onChange             |
| Lookup / async validation       | Client Script with GlideAjax           | onChange             |

## Avoidance

- Never use catalog items for creating task records directly (use Record Producers)
- Never create record producers for `sc_request`, `sc_req_item`, `sc_task`
- Never call `current.update()` or `current.insert()` in pre-insert scripts
- Never call `current.setAbortAction()` in Record Producer scripts
- Never use GlideAjax in onSubmit scripts (async issues)
- Never manipulate DOM directly — always use `g_form` API
- Never use the same variable name as a target table field name
- Never skip the `order` property on variables
- Never skip catalogs or categories assignment
- Never hard-code sys_ids without documenting their source

## Important Notes

- Variables without names cannot be accessed by client scripts
- Mandatory variables without values cannot be hidden by UI policies
- Multi-row variable sets have restrictions on certain variable types (no attachments, containers, HTML, macros)
- Container variables must be properly paired (Start/Split/End)
- Use `Now.include(...)` for external script files for maintainability

## Next Steps

Use `load_skill_resource` with skill name "service-catalog" to load detailed references:

| Resource                          | Content                                                                |
| --------------------------------- | ---------------------------------------------------------------------- |
| `references/catalog-item.md`      | Fulfillment configuration, pricing, access control, UI display options |
| `references/record-producer.md`   | Field mapping methods, script types, unsupported tables                |
| `references/catalog-variables.md` | Variable types, choice and reference examples                          |
| `references/variable-set.md`      | Attaching to items, MRVS constraints, role-based access                |
| `references/ui-policy.md`         | Condition syntax, priority rules, variable type limitations            |
| `references/client-script.md`     | Script types, g_form API, GlideAjax, variable set scripts              |

For working templates and examples, use `get_knowledge_source` tool with:

- **CATALOG_ITEM** — Catalog item templates, properties, pricing, fulfillment
- **CATALOG_ITEM_RECORD_PRODUCER** — Record producer templates, field mapping, server-side scripts
- **CATALOG_VARIABLES** — All variable types, container layout, pricing, field mapping
- **CATALOG_VARIABLE_SET** — Single-row and multi-row variable set templates, MRVS constraints
- **CATALOG_UI_POLICY** — UI policy templates, condition syntax, actions, variable set policies
- **CATALOG_CLIENT_SCRIPT** — onLoad, onChange, onSubmit scripts, GlideAjax, g_form API

For access control, activate the `role` skill.

For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
