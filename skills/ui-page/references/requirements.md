# Essential Guidelines and Implementation Requirements

When developing UI Pages with Fluent, follow these guidelines to ensure proper functionality. Failure to follow these requirements may result in build or runtime errors.

## Core Requirements

1. **UiPage API Usage**: UI Pages must be created using the `UiPage` API found in `@servicenow/sdk/core`.

2. **HTML Reference**: Always use imports (not `Now.include()`) to reference HTML files in UI Page definitions with TSX/React support. Example: `import myPage from '../../client/index.html'`. HTML files should only be placed in the `src/client` directory. Do not nest HTML files deeper than that.

3. **Script Management**: TypeScript/TSX code should be implemented in separate files and loaded via script tags with the `type="module"` attribute in the HTML files. Do not embed or inline TypeScript directly in the HTML - maintain a clean separation by using a single script tag that loads your main TSX file entrypoint. Do not load scripts from CDNs. Use `.tsx` extension for files containing TSX syntax.

4. **Use of HTML**: Ensure that your HTML files are valid HTML. This means using self-closing tags for void elements (like `<img />`, `<br />`, etc.) and ensuring all tags are properly nested and closed.

5. **No DOCTYPE**: Never add `<!DOCTYPE html>` declarations in your HTML files. Never add an XML preamble: `<?xml version="1.0" encoding="utf-8"?>`.

6. **No Jelly**: Do not include Jelly elements in HTML files when using Fluent for UI Pages.

7. **No Client Script**: Do not include `client_script` or `processing_script` fields in the UI Page record definition. These are not supported in Fluent.

8. **No Script Includes**: Do not use `<g:script src="...">...</g:script>` or `<g:include src="...">...</g:include>` in your HTML files. Use `<script src="..."></script>` instead.

9. **No g_form**: Do not reference g_form in your UI Pages.

10. **Use React**: Always use React for UI Pages. Do not use pure HTML or other frameworks.

11. **Modularize**: UI Pages should not be defined in a single file. Follow the given examples.

12. **Dependencies**: All third-party libraries (such as React) must be declared in your project's package.json file. Do not use CDNs or other external sources to load libraries. When adding new dependencies, NEVER modify the versions of existing dependencies - only add the new dependency while preserving all existing versions.

13. **Scope Access**: When you need to reference your application's scope or scope ID, extract this information from the now.config.json file if not otherwise available.

14. **Event Handling**: When using ES modules, use event listeners in your TypeScript instead of inline event handlers. Inline event handlers (like onclick="function()") won't work with modules.

15. **Component Modularization**: Break applications into small, focused React components in separate files. Keep files under 100 lines when possible. Each component should have a single responsibility. Minimize comments - only add them when absolutely necessary for complex logic. Import components rather than defining everything in one file.

16. **Error Handling**: All service layer API calls must implement comprehensive error handling:
    - Wrap fetch operations in try/catch blocks
    - Check response.ok but still parse JSON for error details (ServiceNow returns error messages in JSON body)
    - Provide user-friendly error messages
    - Log errors for debugging purposes
    - Handle network failures gracefully
    - Account for JSON parse errors

17. **Authentication**: ServiceNow authentication is handled automatically when you include the `X-UserToken: window.g_ck` header in your fetch requests. This token is provided by ServiceNow globally and ensures requests are authenticated as the current user. Always include this header for Table API calls.

18. **Accessibility**: Follow WCAG 2.1 AA standards, use semantic HTML, and ensure keyboard navigation works for all interactive elements.

19. **Build and Install**: NEVER instruct users to run custom npm commands or create build configuration files (webpack, vite, etc.). The ServiceNow IDE handles all build and installations automatically - it installs dependencies from package.json and runs the appropriate now-sdk commands. Focus only on writing application code, not build tooling.

20. **Field Extraction**: Always extract primitive values from ServiceNow fields before rendering. Reference, choice, and sys_id fields become objects when using `sysparm_display_value=all`. Use `typeof field === 'object' ? field.display_value : field` for display values and `typeof field === 'object' ? field.value : field` for sys_id values.

21. **Ampersand Character**: Use `$[AMP]` instead of `&` in text content within HTML files.

## Technology Stack (MANDATORY)

- **React 18.2.0** - All UI Pages MUST use React exclusively
- **@servicenow/react-components ^0.1.0** - MUST install with caret `^` for ServiceNow UI components
- **ServiceNow Table API** - Primary integration method for CRUD operations
- **Single Page Application (SPA)** - For complex multi-view experiences
- **Fluent DSL** - TypeScript-based configuration language
- **Build System** - ServiceNow IDE handles ALL build processes automatically

## Agent Decision Tree

When user requests a UI Page:

1. Record lists → Use `RecordListConnected` from `@servicenow/react-components`
2. Record forms → Use `RecordProvider` + `RecordField` from `@servicenow/react-components`
3. Custom data (non-record) → React component with Table API + `@servicenow/react-components` UI elements
4. Multi-step workflows → SPA with state-based routing
5. Dashboard/analytics → Multiple React components combining the above patterns
6. Any UI request → ALWAYS use React, NEVER vanilla JS or jQuery
7. Build configuration → NEVER create (IDE handles everything)

## Limitations

When developing UI Pages with React in ServiceNow Fluent, be aware of the following limitations:

- **No media support**: Audio, video, and WASM files are not supported
- **Deterministic paths**: No hashed output paths; file paths must be predictable
- **No preloading**: `<link rel="preload">` is not supported
- **CSS limitations**:
  - No CSS Modules (`.module.css` with locally scoped classes)
  - No @import statements within CSS files
  - No CSS-in-CSS imports or relative stylesheet references
  - Must use ESM imports in TypeScript/TSX files only
- **Routing**: Use URLSearchParams for navigation (e.g., `?view=list&id=123`), NOT hash routing (`#/route`)
- **No server-side rendering**: React server components and SSR are not available
