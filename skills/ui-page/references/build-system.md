# ServiceNow IDE Build System Constraints

## NON-NEGOTIABLE BUILD RULES

The ServiceNow IDE handles ALL build processes automatically. The agent:

**MUST NEVER:**

- Create webpack.config.js, vite.config.js, or any build configs
- Add build scripts to package.json
- Configure babel, typescript compiler, or bundlers
- Create .babelrc, tsconfig.json, or similar files
- Attempt to modify the build pipeline
- Add build tools as dependencies

**MUST ALWAYS:**

- Trust the IDE build system to handle everything
- Use only the file patterns shown in templates
- Place files in exact locations specified
- Use ESM imports (the IDE handles transformation)
- Follow the import rules (CSS via import, no modules)

## What the IDE Handles Automatically

- TSX transformation
- Module bundling
- CSS processing
- Import resolution
- Development server
- Production builds

## Package.json Restrictions

**Mental Model of CORRECT modification:**

```ts
// Read existing
const pkg = JSON.parse(existingContent);
// Ensure dependencies object exists
if (!pkg.dependencies) pkg.dependencies = {};
// ADD ONLY runtime dependencies
pkg.dependencies["react"] = "18.2.0";
pkg.dependencies["react-dom"] = "18.2.0";
pkg.dependencies["@servicenow/react-components"] = "^0.1.0"; // IMPORTANT: caret ^ is required
if (!pkg.devDependencies) pkg.devDependencies = {};
pkg.devDependencies["@types/react"] = "18.3.12";

// Write complete file back

// Do NOT replace or overwrite existing dependencies.
// NO "scripts" section
// NO build configurations (IDE handles build)
```

When adding dependencies, preserve existing versions - never modify them.

**CRITICAL:** After modifying package.json, always use the `install_dependencies` tool to install them. Dependencies will not be available until installed.

## HTML Entry Point Requirements

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

**IMPORTANT**: The `uxpcb` parameter is required to ensure that stale UI Page contents are not mistakenly cached. It can cause issues with the page ui_diagnostics tool. Add this parameter to the script element of index.html if it is missing.

**IMPORTANT**: Including the `<sdk:now-ux-globals></sdk:now-ux-globals>` tag brings in support for theming and other platform support.

## Project Structure

```
src/
  client/
    tsconfig.json       # TypeScript config
    index.html          # Entry HTML (HTML format, no DOCTYPE or XML preamble)
    main.tsx            # React bootstrap
    app.tsx             # Main component
    utils/fields.ts     # Field utilities (create first)
    components/         # React components
    services/           # API service layer
  fluent/
    ui-pages/
      page.now.ts       # UiPage definition
```
