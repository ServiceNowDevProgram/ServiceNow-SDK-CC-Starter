---
name: ui-page
description: Guide for creating ServiceNow UI Pages using React and the Fluent DSL. Use when user mentions UI Page, custom page, React interface, web page, form interface, dashboard, SPA, or single page application. Also use proactively when building user-facing interfaces, data entry forms, list views, multi-step wizards, or any custom web experience within ServiceNow. Covers React 18.2.0 patterns, Table API integration, CSS styling, field extraction, and SPA routing.
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


# UI Pages

## When to use this skill

- When user explicitly asks for a UI Page or custom interface
- When creating React-based user interfaces in ServiceNow
- When building forms, lists, dashboards, or data entry screens
- When implementing single page applications (SPAs) with routing
- When integrating with ServiceNow Table API for CRUD operations
- When applying theming and CSS styling to custom pages

## Instructions

- **Component Selection**: List specific ServiceNow React components from `@servicenow/react-components` to be used (e.g., `RecordListConnected`, `Card`, `Button`). Include a step to read component documentation using `package_docs` and `read_file`.
- **Navigation Architecture**: If more than two views exist, define URLSearchParams structure (e.g., `?view=list`, `?view=details&id=123`).
- **Dirty State Tracking**: If ANY forms or edits exist, include explicit steps to implement dirty state tracking (compare original vs current data, warn on navigation).
- **Application Menu Creation**: Include steps to create Application Menu and App Module.

**These are NOT optional. Every UI Page plan must address all four areas above.**

1. **Component Library:** Use `@servicenow/react-components` for all UI elements. Before writing UI code, follow the Component Documentation Pattern below.
2. **Technology Stack:** Always use React 18.2.0. Never use vanilla JavaScript, jQuery, or other frameworks. Use TypeScript when writing code that uses React components in .tsx files.
3. **Dependencies:** After adding dependencies to package.json, always use the `install_dependencies` tool to install them.
4. **Field Utilities:** Create `src/client/utils/fields.ts` first with `display()` and `value()` helpers.
5. **API Calls:** Always use `sysparm_display_value=all` and include `X-UserToken: window.g_ck` header.
6. **File Size:** Keep files under 100 lines. Break into components when exceeding this limit.
7. **CSS:** Import CSS via ESM (`import "./file.css"`). CSS Modules are NOT supported.
8. **Build System:** Never create webpack, vite, or build configs. The ServiceNow IDE handles everything. Build the app at least once before checking diagnostics or expect to see this error: "failed to add HTML file: No file found at path".
9. **Record Forms:** When building forms to view/edit ServiceNow records, ALWAYS wrap with `RecordProvider` — NEVER use standalone Input components with manual Table API calls for record CRUD.
10. **Dirty State:** When ANY form or edit view exists, ALWAYS implement dirty state tracking (compare original vs current data, warn before navigation). See Dirty State Management section below.
11. **URL-Based Navigation (DEFAULT):** Each logical part/view of the application (list view, detail view, edit view, create view, tabs, etc.) MUST be accessible via URL using URLSearchParams (e.g., `?view=list`, `?view=details&id=123`, `?tab=overview`). This is the DEFAULT approach unless the user explicitly specifies otherwise. Follow patterns in `references/navigation.md`.

## Component Documentation Pattern

**BEFORE writing ANY UI code**, install and read the component documentation:

1. Add dependencies to package.json: `"react": "18.2.0"`, `"react-dom": "18.2.0"`, AND `"@servicenow/react-components": "^0.1.0"` (IMPORTANT: the caret `^` is required for react-components). Also add devDependencies to package.json: `"@types/react": "18.3.12"`. Then run `install_dependencies`.
2. Call `package_docs({ packageName: "@servicenow/react-components" })` to get the package directory and see available components.
3. For each component you will use, read its documentation using `read_file` with the package directory path + `docs/<ComponentName>.md`.
4. Do NOT assume standard React patterns. ServiceNow components often differ:
   - Text content uses `label` prop, not children
   - Lists use `items` array prop, not mapped children
   - Events use `onXxxSet` naming (e.g., `onSelectedItemSet`) with data in `event.detail`

## Component Selection

| Building                                             | Use                                              |
| ---------------------------------------------------- | ------------------------------------------------ |
| List of ServiceNow records (incidents, users, tasks) | `RecordListConnected` — NOT manual Table API     |
| Form to view/edit a single record                    | `RecordProvider` wrapper — NOT standalone Inputs |
| UI elements (buttons, inputs, modals, tabs)          | `@servicenow/react-components`                   |

Refer to `references/component-selection.md` for more details.

## Key concepts

### UiPage API

UI Pages must be created using the `UiPage` API from `@servicenow/sdk/core`:

```ts
import { UiPage } from "@servicenow/sdk/core";
import page from "../../client/index.html";

export const my_page = UiPage({
  $id: Now.ID["my-page"],
  endpoint: "x_app_page.do",
  html: page, // CRITICAL: must import the content to use the output of the build system
  direct: true // CRITICAL: Must be true
});
```

### Project Structure

```plaintext
src/
  client/
    tsconfig.json       # TypeScript config
    index.html          # Entry HTML (HTML format, no DOCTYPE or XML preamble)
    main.tsx            # React bootstrap
    app.tsx             # Main component written in TypeScript
    utils/fields.ts     # Field utilities (create first)
    components/         # React components
    services/           # API service layer
  fluent/
    ui-pages/
      page.now.ts       # UiPage definition
```

## tsconfig.json Contents

```json
{
  "compilerOptions": {
    "moduleResolution": "bundler",
    "module": "es2022",
    "target": "es2022",
    "lib": ["ES2022", "DOM"],
    "jsx": "preserve"
  }
}
```

## index.html Example Contents

```html
<!-- src/client/index.html -->
<html class="-polaris">
  <head>
    <title>My Page</title>
    <sdk:now-ux-globals></sdk:now-ux-globals>
    <script
      src="main.tsx?uxpcb=$[UxFrameworkScriptables.getFlushTimestamp()]"
      type="module"
    ></script>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

### Agent Decision Tree

When building a UI Page, follow these steps:

1. **Create ServiceNow application** (if new)
2. **Set proper HTML title**: Include dynamic title generation based on context (Polaris iframe vs standard page), similar to navigation pattern
3. **Read component documentation**: Use `package_docs({ packageName: "@servicenow/react-components" })` and `read_file` to read docs for components you'll use
4. **List specific components**: Explicitly state which components from `@servicenow/react-components` will be used (see `references/component-selection.md`)
5. **Define navigation structure**: If two or more views/logical entities exist (list + details, tabs, different sections), specify URLSearchParams structure (e.g., `?view=list`, `?view=details&id=123`, `?tab=overview`)
6. **Plan dirty state tracking**: If ANY forms/edits exist, include steps to implement dirty state (compare original vs current, warn on navigation)
7. **Create UI Page files**: HTML, JSX, components, services, CSS
8. **Create Application Menu and App Module**: Include steps to create `sys_app_application` and `sys_app_module` with `link_type: "DIRECT"`
9. **Build and install**

**Decision rules:**

- Listing ServiceNow records → ALWAYS use `RecordListConnected`, NEVER manual Table API
- Viewing/editing a record → ALWAYS use `RecordProvider` wrapper, NEVER standalone inputs
- Multiple views → ALWAYS use URLSearchParams
- Any form/edit → ALWAYS implement dirty state tracking
- Navigation/title updates → ALWAYS check `window.self !== window.top` for Polaris iframe detection
- Build config → NEVER create (IDE handles everything)

## Application Menu and App Module

ALWAYS create Application Menu and App Module for UI Pages. For detailed examples, see `references/app-menu.md`.

1. **Create Application Menu** (`sys_app_application`): Define the main application entry in the ServiceNow menu
2. **Create App Module** (`sys_app_module`): For each UI Page, create a module with `link_type: "DIRECT"` and `uri` matching the UiPage `endpoint`

## URL Generation and Navigation

ALWAYS use URLSearchParams for navigation. For detailed patterns, see `references/navigation.md`.

**CRITICAL: ALWAYS implement iframe detection for Polaris compatibility:**

```javascript
if (window.self !== window.top) {
  // Polaris iframe: Use CustomEvent.fireTop
  window.CustomEvent.fireTop("magellanNavigator.permalink.set", {
    relativePath: path,
    title: title
  });
} else {
  // Standalone: Use history.pushState
  window.history.pushState({}, "", path);
  document.title = title;
}
```

- Each view MUST have its own URL using URLSearchParams (`?view=details&id=123`)
- ALWAYS check `window.self !== window.top` before ANY navigation or title update
- Use `window.CustomEvent.fireTop()` in Polaris iframe, `history.pushState()` standalone

## Dirty State Management

**MANDATORY for ANY view that edits or creates records.** If a form exists, dirty state tracking MUST exist. For detailed patterns, see `references/dirty-state.md`.

1. Store `originalData` when record is loaded
2. Compare `formData` vs `originalData` with `JSON.stringify()` on every change
3. Show a visual indicator (e.g., "Unsaved changes") when dirty
4. Warn on `beforeunload` when dirty to prevent accidental data loss
5. Reset dirty state after successful save

- ServiceNow components use `event.detail.value`, not `event.target.value`

## Avoidance

- Never use raw HTML elements (`<button>`, `<input>`, `<select>`) when `@servicenow/react-components` provides equivalents. Refer to `references/component-selection.md` for details.
- **CRITICAL:** Never use standalone Input components for ServiceNow record operations - ALWAYS use `RecordListConnected` for lists and `RecordProvider` for forms (see Component Selection section)
- **CRITICAL:** Never use hash-based routing (`#/path`) - ALWAYS use URLSearchParams with query strings (`?view=details`) (see `references/navigation.md`)
- **CRITICAL:** Never skip dirty state tracking for forms - ALWAYS track unsaved changes and warn users before navigation (see `references/dirty-state.md`)
- **CRITICAL:** Never skip iframe detection (`window.self !== window.top`) - ALWAYS implement Polaris compatibility for navigation and title updates (see `references/navigation.md`)
- Never assume standard React patterns for ServiceNow components - always read the component docs first
- Never create build configuration files (webpack, vite, babel, tsconfig)
- Never use CSS Modules (`.module.css`) or `@import` in CSS files
- Never use CDNs or external script sources
- Never use GlideAjax, g_form, Jelly, or `<g:script>` tags
- Never add `client_script` or `processing_script` fields
- Never inline JavaScript in HTML or use `onclick` handlers
- Never skip the `X-UserToken` header in API calls

## Next Steps

Use `load_skill_resource` with skill name "ui-page" to load detailed references:

| Resource                            | Content                                                                      |
| ----------------------------------- | ---------------------------------------------------------------------------- |
| `references/requirements.md`        | Essential guidelines, limitations, technology stack                          |
| `references/build-system.md`        | IDE constraints, package.json rules, project structure                       |
| `references/component-selection.md` | Component usage patterns, RecordListConnected, RecordProvider examples       |
| `references/field-extraction.md`    | Field utilities pattern, display/value helpers                               |
| `references/spa-patterns.md`        | DEFAULT navigation pattern: URL-based SPA routing, shared state with Context |
| `references/navigation.md`          | URL generation, URLSearchParams, context-aware routing                       |
| `references/dirty-state.md`         | Track unsaved changes, prevent data loss                                     |
| `references/app-menu.md`            | Application Menu and App Module creation                                     |
| `references/file-guidelines.md`     | File size limits, when to split components                                   |
| `references/css-styling.md`         | CSS import rules, BEM naming, theming integration                            |
| `references/service-layer.md`       | API service pattern, CRUD operations, error handling                         |

For working templates and examples, use `get_knowledge_source` tool to get the **UI_PAGE** knowledge source.

For theming and design system, use `get_knowledge_source` tool with:

- **UI_PAGE_THEMING_FOUNDATIONS** - Color, typography, spacing
- **UI_PAGE_THEMING_LAYOUT** - Layout patterns and containers
- **UI_PAGE_THEMING_COMPONENTS** - UI components styling
- **UI_PAGE_THEMING_CONTROLS** - Form controls and inputs

Use `css_variable_explorer` tool to search for CSS variables from the Horizon Design System.
