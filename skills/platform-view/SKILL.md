---
name: platform-view
description: Configure UI controls for ServiceNow platform lists and forms. Covers Views (sys_ui_view), View Rules (sysrule_view), List Controls (sys_ui_list_control), UI Policies (sys_ui_policy), UI Actions (sys_ui_action), UI Formatters (sys_ui_formatter), and Lists (sys_ui_list). Use when user mentions configuring forms or configuring lists for the platform view of tables and record. Example requests might include - buttons on forms or lists, field visibility/mandatory rules, formatting a form, adding sections to a form, activity streams, process flows, list columns, form layout, persona-based layouts, state-driven layout changes, dynamic field behavior, role-restricted list actions, or automatic view switching. Also use proactively when building applications that need interactive form behavior, list configuration, or layout control.
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


# Platform Views & UI Layout Control

Routing guide for all ServiceNow platform UI layout, behavior, and presentation controls.

## When to use this skill

- Adding buttons or actions to forms or lists → **UI Actions**
- Making fields mandatory, read-only, visible, or hidden based on conditions → **UI Policies**
- Adding non-field content to forms (activity streams, process flows, checklists) → **UI Formatters**
- Configuring which columns appear in a list view → **Lists**
- Creating persona-based or role-based form layouts → **Views**
- Automatically switching layouts based on conditions, device, or role → **View Rules**
- Controlling New/Edit button visibility on lists, role-based list access → **List Controls**

## Choosing the right approach

| Need                                                    | Use                        | KS           |
| ------------------------------------------------------- | -------------------------- | ------------ |
| Button/link on form or list                             | UI Actions                 | UI_ACTION    |
| Field visibility/mandatory/read-only based on condition | UI Policies                | UI_POLICY    |
| Non-field content on forms (activity, process flow)     | UI Formatters (Record API) | UI_FORMATTER |
| Configure list columns and order                        | Lists                      | LIST         |
| Different form layout per role/group/persona            | Views                      | UI_VIEW      |
| Auto-switch layout when condition met                   | View Rules                 | SYSRULE_VIEW |
| Hide New/Edit buttons, role-based list actions          | List Controls              | LIST_CONTROL |

**Views vs View Rules vs UI Policies — choose based on scope:**

| Scenario                                                                 | Use              |
| ------------------------------------------------------------------------ | ---------------- |
| Whole form layout changes per role/group (different fields/sections)     | **View**         |
| Whole form layout switches automatically based on condition/device/state | **View Rule**    |
| Specific fields hide/show/mandatory/read-only when condition met         | **UI Policy**    |
| Control list buttons (New/Edit) or disable pagination                    | **List Control** |

**Views vs ACLs — choose based on intent:**

| Intent                                                               | Use                                              |
| -------------------------------------------------------------------- | ------------------------------------------------ |
| Certain fields/sections should not appear in the form for some users | **Views** — fields absent from the form entirely |
| Restrict who can read/write/delete records or fields (data security) | **ACL skill** — security enforcement             |

## Instructions

### UI Actions

1. **Use the `UiAction` API** from `@servicenow/sdk/core`. Every UI Action must have `$id`, `table`, `name`, and `actionName`.
2. **Be explicit about placement:** Set `form.showButton`, `list.showButton`, etc. to control where the action appears. If blank, the action may not appear anywhere useful.
3. **Set visibility mode:** Use `showInsert: true` for new record forms, `showUpdate: true` for existing record forms and list views.
4. **Client vs. server scripts:** Set `client.isClient: true` for client-side execution. When `isClient` is true, use `client.onClick` for the trigger and `script` for the function definition.
5. **Set a `style`** on form/list objects — `'primary'`, `'destructive'`, or `'unstyled'`.
6. **Use `condition`** to control when the action is visible (e.g., `current.canWrite()`).
7. **The `script` field must never return anything.**

### UI Policies

1. **Use the `UiPolicy` API** from `@servicenow/sdk/core`. Every UI Policy must have `$id`, `table`, and `shortDescription`.
2. **Use encoded query syntax for `conditions`** — e.g., `"priority=1^state!=6"`.
3. **Action properties use `boolean | 'ignore'`** — use `true`/`false` to set, `'ignore'` to leave unchanged.
4. **Use `readOnly` (not `disabled`)** — it maps directly to ServiceNow's `disabled` field.
5. **Prefer declarative `actions` over scripts** — better performance and less error-prone.

**Load detailed guidance:** Use `load_skill_resource` with skill `"platform-view"`:

- `references/ui-policy-guide.md` — field actions, related list actions, condition patterns
- `references/ui-policy-examples.md` — comprehensive examples (condition-based, field clearing, complex logic)
- `references/ui-policy-scripts.md` — g_form API and client-side script patterns
- `references/ui-policy-related-lists.md` — related list visibility control

### UI Formatters

1. **Use built-in formatters when possible** — don't create duplicate `sys_ui_formatter` records for Activity, Attached Knowledge, or other built-ins.
2. **Custom formatters are NOT supported in Fluent** — always use the built-in formatters.
3. **A formatter requires a section** — it must reside in a `sys_ui_section`, and the section must have at least one non-formatter element.
4. **Follow the sequential steps:** (1) Check if formatter exists in `sys_ui_formatter`, (2) Check if section exists in `sys_ui_section`, (3) Add `sys_ui_element` with `type: 'formatter'`.
5. **Activity and Attached Knowledge formatters already exist globally** — never create new `sys_ui_formatter` records for these. Skip straight to adding the `sys_ui_element`.
6. **Parent Breadcrumb requires a field named exactly `parent`** — no variations like `parent_task` or `parent_record`.
7. **Process Flow requires stage configuration** — verify `sys_process_flow` records exist for the target table.
8. **Position matters:** Process Flow and Parent Breadcrumb go first in the section. Activity and Checklist go last.

#### Built-in formatters

| Formatter          | Macro                                              | When to use                           |
| ------------------ | -------------------------------------------------- | ------------------------------------- |
| Activity           | `activity.xml`                                     | Journal entries, comments, work notes |
| Process Flow       | `process_flow`                                     | Lifecycle stage visualization         |
| CI Relations       | `ui_ng_relation_formatter.xml`                     | CMDB relationship maps                |
| Parent Breadcrumb  | `parent_crumbs`                                    | Parent hierarchy trail                |
| Contextual Search  | `cxs_table_search.xml`                             | Auto-suggest knowledge articles       |
| Variables Editor   | `com_glideapp_questionset_default_question_editor` | Record producer variables             |
| Checklist          | `inline_checklist_macro`                           | Sub-task tracking                     |
| Attached Knowledge | `attached_knowledge`                               | Linked knowledge articles             |

### Views

Views define which fields, sections, and layout appear on a form or list for a given table.

**If user says "simple", "basic", "default", or "standard":** Use `default_view` from `@servicenow/sdk/core`. No view Record needed. Skip to the relevant metadata (Form, List, etc.).

**Otherwise, before creating any view:** Query the instance first:

```
run_query(table="sys_ui_view", query="name=<proposed_name>^ORtitle=<proposed_title>")
```

Both `name` and `title` are independently unique across all scopes. Write: `"Uniqueness check: name=<X> title=<Y> → <N> record(s) found"` — N must be 0 to proceed.

**Load detailed guidance:** Use `load_skill_resource` with skill `"platform-view"`:

- `references/view-guide.md` — main guide (uniqueness check, view type selection, examples)
- `references/view-role-based.md` — role-based access views
- `references/view-hidden.md` — hidden views (portal/API use)
- `references/view-group-based.md` — group-based access views
- `references/view-user-specific.md` — user-specific views
- `references/view-forms-integration.md` — form/section linking (critical: form appears empty without `sys_ui_form_section` records)

### View Rules

View Rules automatically switch the form layout based on conditions, device type, or script logic.

**Load detailed guidance:** Use `load_skill_resource` with skill `"platform-view"`:

- `references/view-rule-guide.md` — main guide (device, condition, script approaches)
- `references/view-rule-conditions.md` — declarative condition-based rules with encoded queries
- `references/view-rule-advanced.md` — script-based rules (roles, multi-criteria)

### List Controls

List Controls configure UI options on table lists and related lists — role-based New/Edit button visibility, disable pagination counts, conditional button hiding.

**Load detailed guidance:** Use `load_skill_resource` with skill `"platform-view"`:

- `references/list-control-guide.md` — main guide (omit buttons, role restrictions, conditions)
- `references/list-control-examples.md` — comprehensive implementation examples

### Lists

1. **Use the `List` API** from `@servicenow/sdk/core`. Requires `table`, `view`, and `columns`.
2. **For custom views,** create a `sys_ui_view` Record first, then reference it in the List.
3. **Use `default_view`** from `@servicenow/sdk/core` when no custom view is needed.
4. **Dot-walking is supported** in column element names (e.g., `"caller_id.name"`).
5. **Use `parent` and `relationship`** properties to configure related lists appearing on forms.
6. **Aggregates:** Use `sum`, `averageValue`, `minValue`, `maxValue` for column-level calculations on numeric fields.

**Load detailed guidance:** Use `load_skill_resource` with skill `"platform-view"` and file `"references/list-examples.md"` for examples with relationships and aggregates.

## Avoidance

- **Never create `sys_ui_formatter` records for Activity or Attached Knowledge** — they already exist globally
- **Never create custom formatters** — not supported in Fluent
- **Never use `disabled` in UI Policy actions** — use `readOnly` instead (the plugin maps it)
- **Never place a formatter in a section with no other elements** — it won't render
- **Never have a UI Action script return a value** — the `script` field must never return anything
- **Never skip the view uniqueness check** — `sys_ui_view` is global and `title` must be unique across all scopes
- **Never create lists for tables that don't exist yet** — define the table first
- **Never use personal list preferences (sys_ui_list_user) in application code** — those are user-specific
- **Never mix basic and advanced relationship fields in view configurations** — use one pattern or the other

## Next Steps

- For API properties and code examples, use `get_knowledge_source` with: **UI_ACTION**, **UI_POLICY**, **UI_FORMATTER**, **LIST**, **UI_VIEW**, **SYSRULE_VIEW**, **LIST_CONTROL**
- For business rules that run server-side on record changes, see the `business-rule` skill
- For client scripts that run on form events, see the `client-script` skill
- For data access security (ACLs), see the `implementing-security` skill
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
