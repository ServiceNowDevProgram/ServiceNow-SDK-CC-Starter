---
name: implementing-tests
description: Guide for creating ServiceNow Automated Test Framework (ATF) test cases using Fluent APIs across 11 ATF categories — server [atf.server], form [atf.form], REST [atf.rest], catalog [atf.catalog], email [atf.email], app navigator [atf.appnav], reporting [atf.reporting], responsive dashboard [atf.responsiveDashboard], and Service Portal variants [atf.form_SP, atf.catalog_SP]. Use when user mentions automated tests, ATF, test cases, test steps, unit tests, test framework, or any atf.* namespace. Also use proactively when building applications that need test coverage for forms, APIs, catalog items, dashboards, or server-side logic.
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


## When to use

Use this skill when generating automated test cases for a ServiceNow application using the Automated Test Framework (ATF) and Fluent APIs.

## Instructions

### Strategic approach to test generation

1. **Analyze the application context** — examine custom tables, forms, APIs, catalog items, dashboards, and business logic to understand what needs testing.
2. **Develop a test strategy** — propose up to 3 representative test cases reflecting critical user workflows before expanding coverage.
3. **Select ATF categories** — map each test interaction to the appropriate `atf.*` namespace:

| Interaction type          | ATF namespace              | Use for                                                                       |
| ------------------------- | -------------------------- | ----------------------------------------------------------------------------- |
| UI navigation             | `atf.applicationNavigator` | Verify menus/modules visible, navigate to modules                             |
| Form interactions         | `atf.form`                 | Open/submit forms, set/validate fields, click UI actions, modal interactions  |
| Forms in Service Portal   | `atf.form_SP`              | Same as form but in Service Portal context                                    |
| REST API validation       | `atf.rest`                 | Send HTTP requests, assert status codes/headers/payload                       |
| Server-side logic         | `atf.server`               | Impersonation, CRUD operations, record validation, logging                    |
| Service Catalog           | `atf.catalog`              | Open/order catalog items, set/validate variables, record producers            |
| Catalog in Service Portal | `atf.catalog_SP`           | Same as catalog but in portal — plus order guides and multi-row variable sets |
| Email testing             | `atf.email`                | Validate outbound emails, generate inbound emails and replies                 |
| Reporting                 | `atf.reporting`            | Assert report visibility                                                      |
| Dashboards                | `atf.responsiveDashboard`  | Assert dashboard visibility and sharing permissions                           |

4. **Implement test steps** using the category-specific Fluent ATF APIs from the corresponding knowledge source.

### Test file structure

5. Every ATF test file must import `Test` from `@servicenow/sdk/core` and `@servicenow/sdk/global` (or `@servicenow/sdk-core/global`).
6. Wrap all steps in the `Test()` constructor with a unique `$id`, `name`, optional `description`, and `failOnServerError: true`.
7. Each test step requires a globally unique `$id` using `Now.ID['step_name']`.
8. Steps execute sequentially — capture earlier step outputs (e.g., `record_id`, `user`) in variables to pass to later steps.

### Category selection guidance

9. Prefer UI-based categories (`atf.form`, `atf.catalog`) over `atf.server` for interactions that users normally perform through the UI. Use `atf.server` only when backend assertions, data setup, or server-only operations are needed.
10. When the user mentions Service Portal, use the `_SP` variants (`atf.form_SP`, `atf.catalog_SP`) instead of the standard form/catalog APIs.
11. Combine categories within a single test for end-to-end workflows (e.g., `atf.server.impersonate` → `atf.form.openNewForm` → `atf.form.setFieldValue` → `atf.form.submitForm` → `atf.server.recordValidation`).

### Tool dependencies

12. Use `runQuery` to fetch exact `sys_id` values for any reference fields — users, roles, groups, UI actions, catalog items, modules, portals, pages, reports, dashboards, etc.
13. Use `get_table_schema` before setting field values (`setFieldValue`, `recordInsert`, `recordUpdate`) to identify mandatory fields and their types.
14. When the test needs to set values on a table, check the application's Fluent code to ensure all mandatory fields are populated with valid test data.

## Key concepts

- **Test data setup**: Use `atf.server.impersonate` and `atf.server.createUser` to establish user context before testing. Use `atf.server.recordInsert` to create prerequisite data.
- **Assertion chaining**: After `atf.form.submitForm`, follow with `atf.server.recordValidation` to verify the record was created correctly on the server side.
- **Form UI flavors**: Form APIs support `standard_ui`, `service_operations_workspace`, `asset_workspace`, and `cmdb_workspace`.
- **Navigator styles**: Application Navigator supports `ui15`, `ui16`, and `polaris`.
- **Catalog variable format**: Variable values use the `IO:<sys_id>=<value>` format joined with `^` and ending with `^EQ`. Retrieve variable sys_ids from `item_option_new` using `runQuery`.
- **Encoded queries**: Field value conditions and validations use ServiceNow encoded query syntax (e.g., `short_description=Test^priority=1`).

## Avoidance

1. Do not overuse `atf.server` for tasks that form or catalog APIs handle directly — do not use `atf.server.recordInsert` to submit a form when `atf.form.submitForm` exists.
2. Do not hardcode `sys_id` values — always use `runQuery` to fetch them.
3. Do not skip mandatory fields when using `setFieldValue` or `recordInsert` — always check with `get_table_schema` first.
4. Do not call sequence-dependent steps out of order — `orderCatalogItem` requires `openCatalogItem` first, `submitRecordProducer` requires `openRecordProducer` first, etc.
5. Do not create generic or template-based tests — each test should reflect the application's real usage scenarios.

## Next Steps

Retrieve the appropriate ATF knowledge source for the specific test category:

- Navigation → `ATF_APPNAV`
- Forms → `ATF_FORM` or `ATF_FORM_SP` (Service Portal)
- REST APIs → `ATF_REST`
- Server-side logic → `ATF_SERVER`
- Service Catalog → `ATF_CATALOG` or `ATF_CATALOG_SP` (Service Portal)
- Email → `ATF_EMAIL`
- Reports → `ATF_REPORTING`
- Dashboards → `ATF_RESPONSIVEDASHBOARD`
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
