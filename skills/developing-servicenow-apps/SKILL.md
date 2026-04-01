---
name: developing-servicenow-apps
description: "Guides you through ServiceNow app development using the Now SDK: project setup, fluent authoring, build/deploy workflow, and CLI reference. Load this skill FIRST when starting any ServiceNow application project or working with the Now SDK, before loading artifact-specific skills like table, business-rule, or client-script."
user-invocable: true
---

# Developing ServiceNow Apps with the Now SDK

## When to use this skill

Use this skill when:
- Setting up a new ServiceNow application project from scratch
- Working with the ServiceNow SDK (`@servicenow/sdk`) CLI commands
- Scaffolding project structure, modules, or fluent definitions
- Building, deploying, or iterating on a ServiceNow app locally
- Authenticating the SDK against a ServiceNow instance
- Answering questions about SDK capabilities, fluent language, or project configuration

## Agent behavior

When a user asks you to create or modify a ServiceNow app, you are responsible for the entire lifecycle — scaffolding, installing dependencies, writing code, building, and deploying. Do not stop after writing code and ask the user to run CLI commands themselves.

Follow this decision flow:
1. If no project exists, run `init` and `npm install` to scaffold it.
2. Write the fluent source files.
3. Run `npm run build` to compile and validate.
4. Check if auth is already configured with `npx now-sdk auth --list`.
   If credentials exist, proceed. If not, guide the user through auth
   setup (see Authentication section) and wait for confirmation.
5. Run `npm run deploy` to deploy to the instance.

Only pause and ask the user when you need information you cannot infer (e.g., which instance to target, what scope name to use). Do not pause to ask the user to run commands that you can run yourself.

**Authentication is interactive** — `npx now-sdk auth --add` requires the user
to type credentials into prompts, so you cannot run it yourself. When auth is
not configured:
1. Tell the user they need to authenticate before the app can be deployed.
2. Provide the exact command:
   `npx now-sdk auth --add <instance-url> --type <basic|oauth>`
3. Explain the two supported auth types (`basic` for username/password,
   `oauth` for OAuth-enabled instances) and let the user choose the
   appropriate one for their instance.
4. Explain it will prompt for an alias and the relevant credentials.
5. **Stop and wait** for the user to confirm authentication is complete
   before continuing to the install step.
6. Once the user confirms, verify with `npx now-sdk auth --list` and
   then proceed.

## Prerequisites

- **Node.js 20 or later** (LTS recommended)
- **npm** (bundled with Node.js)
- Access to a ServiceNow instance (PDI or enterprise) with admin or developer credentials

## Installation

For **new projects**, scaffold with the `init` command. Pin the SDK version to avoid breaking changes.

### Non-interactive scaffolding (recommended for agents)

Use CLI flags to skip interactive prompts:

```bash
npx @servicenow/sdk@4.3.0 init \
  --appName "My App" \
  --packageName "my-app" \
  --scopeName "x_my_app" \
  --template "typescript.basic"
```

Available `--template` values: `base`, `configuration`, `javascript.react`, `typescript.basic`, `typescript.react`, `typescript.vue`.

> **Note:** `init` creates project files in the **current working directory**.
> It does not create a subdirectory. Make sure you are in the desired project
> directory before running `init`.

After scaffolding, install dependencies:

```bash
npm install
```

### Interactive scaffolding

```bash
npx @servicenow/sdk@4.3.0 init
```

Prompts for app scope, name, and target instance. After it completes, run `npm install`.

For **existing projects** that need the SDK as a dev dependency:

```bash
npm i @servicenow/sdk@4.3.0 --save-dev
```

> Always pin to a specific version.

## CLI commands reference

All commands are invoked via `npx now-sdk <command>` (or `npx @servicenow/sdk <command>` if not using the short alias).

| Command | Purpose |
|---------|---------|
| `init` | Scaffold a new project. Use `--appName`, `--packageName`, `--scopeName`, `--template` flags for non-interactive setup. |
| `auth` | Authenticate against a ServiceNow instance. `--add <url> --type basic\|oauth` to add credentials, `--list` to check existing, `--use <alias>` to set a default. |
| `build` | Compile fluent source files into ServiceNow artifacts. Validates syntax and reports errors. |
| `install` | Push built artifacts to the target instance. Requires prior `auth`. |
| `transform` | Convert existing instance artifacts into fluent source files (reverse-engineer). |
| `download` | Download specific records or update sets from an instance into the local project. |
| `dependencies` | Fetch TypeScript type definitions for platform APIs and scoped app tables. Useful for IDE autocompletion. |
| `clean` | Remove build output and cached artifacts. |
| `pack` | Package the app into an update set XML for manual deployment. |

## Project structure

A freshly scaffolded project (via `init --template "typescript.basic"`) contains:

```
src/
  fluent/
    index.now.ts           # Main fluent entry point
    example.now.ts         # Example fluent definition (can be replaced)
    tsconfig.json          # Fluent TypeScript config
  server/
    script.ts              # Example server-side script
    tsconfig.json          # Server TypeScript config
now.config.json            # App metadata: scope, scopeId, name
package.json
```

As you add artifacts, organize them into subdirectories by type using
kebab-case naming:

```
src/fluent/
  business-rules/
    my-rule.now.ts
  client-scripts/
    my-script.now.ts
```

- **`now.config.json`** — App metadata. Contains the app scope (e.g., `x_myco_myapp`), scopeId, display name, and tsconfig path. Does **not** contain instance connection info — that is managed separately via `auth`.

  Example:
  ```json
  {
    "scope": "x_my_app",
    "scopeId": "26571502d0a642339adf60a7edf6fab9",
    "name": "My App",
    "tsconfigPath": "./src/server/tsconfig.json"
  }
  ```

- **`src/fluent/`** — All fluent definitions live here. Files use the `.now.ts` extension. Subdirectories use kebab-case (e.g., `business-rules/`, `client-scripts/`).
- **`src/server/`** — Server-side TypeScript scripts referenced by fluent definitions. These contain the actual logic (functions) that run on the ServiceNow platform.

## Development workflow

The standard development loop is:

1. **`init`** — Scaffold the project (one-time):
   ```bash
   npx @servicenow/sdk@4.3.0 init --appName "My App" --packageName "my-app" --scopeName "x_my_app" --template "typescript.basic"
   ```
2. **`npm install`** — Install SDK and dependencies (one-time, after init):
   ```bash
   npm install
   ```
3. **`auth`** — Authenticate against your instance (or verify existing auth):
   ```bash
   # Check if auth is already configured
   npx now-sdk auth --list

   # If not configured, add credentials (interactive — user must run this)
   npx now-sdk auth --add <instance-url> --type basic

   # Or set a specific saved credential as default
   npx now-sdk auth --use <alias>
   ```
4. **Write fluent** — Create or edit `.now.ts` files under `src/fluent/`. Each file defines ServiceNow artifacts using the object-based API. Write server-side script functions in `src/server/`.
5. **`build`** — Compile and validate your fluent definitions:
   ```bash
   npm run build
   ```
6. **`deploy`** — Push the compiled artifacts to the instance:
   ```bash
   npm run deploy
   ```
7. **Iterate** — Repeat steps 4-6 as you develop.

For brownfield projects (migrating existing apps), use `transform` to pull instance artifacts into fluent source, then continue with the normal loop.

## Authentication

### Checking existing credentials

Before deploying, always check if auth is already configured:

```bash
npx now-sdk auth --list
```

If credentials exist, proceed directly to build and install.

### Adding credentials

```bash
npx now-sdk auth --add <instance-url> --type <auth-type>
```

The `--type` flag supports two authentication types:
- **`basic`** — Username and password authentication. Suitable for local development and PDIs.
- **`oauth`** — OAuth-based authentication. Suitable for enterprise instances with OAuth configured.

The developer should select the appropriate type for their instance.

For example, with basic auth:

```bash
npx now-sdk auth --add http://localhost:8080 --type basic
```

This will prompt for:
- **Alias**: A name for this credential set (e.g., `default`)
- **Username**: Instance admin or developer username
- **Password**: Instance password

Credentials are stored in a local `.now-sdk/` directory (gitignored by default).

### Setting a default credential

```bash
npx now-sdk auth --use <alias>
```

### Non-interactive (CI/CD)

Set environment variables before running SDK commands:

```bash
export SN_SDK_INSTANCE_URL=https://myinstance.service-now.com
export SN_SDK_USER=admin
export SN_SDK_USER_PWD=password
```

When these environment variables are present, the SDK uses them directly
and skips stored credentials. Environment variables take precedence over
credentials saved via `auth --add`. This is the recommended approach for CI pipelines.

## Key concepts

### Fluent language (SDK 4.x object-based API)

The fluent API is the SDK's TypeScript-based DSL for defining ServiceNow artifacts. Instead of XML or JSON update sets, you write typed definitions using an object-based API:

```typescript
import '@servicenow/sdk/global'
import { BusinessRule } from '@servicenow/sdk/core'
import { myScriptFunction } from '../server/script'

BusinessRule({
  $id: Now.ID['my-rule'],
  name: 'Uppercase Short Description',
  table: 'incident',
  when: 'after',
  action: ['insert', 'update'],
  order: 100,
  script: myScriptFunction,
  active: true,
})
```

Key points:
- Import `@servicenow/sdk/global` to make the `Now` global available (used for `Now.ID`, etc.).
- Import specific artifact types (e.g., `BusinessRule`, `Table`, `ClientScript`) from `@servicenow/sdk/core`.
- Each artifact is defined by calling the artifact function with a configuration object.
- Server-side logic is written as functions in `src/server/` and passed via the `script` property.
- The `build` command compiles these into platform-ready payloads.

### Modules and scoping

Every app has a scope (e.g., `x_myco_myapp`). Modules within the app are purely organizational — they group related artifacts but share the app scope. The SDK enforces that all artifacts reference valid tables and fields within the scope or global namespace.

### Shared constants and utilities

Since fluent files are standard TypeScript, you can use normal imports and shared constants:

```typescript
import '@servicenow/sdk/global'
import { BusinessRule } from '@servicenow/sdk/core'
import { myHandler } from '../server/script'

const commonConfig = {
  when: 'after' as const,
  action: ['insert', 'update'] as const,
  active: true,
}

BusinessRule({
  $id: Now.ID['rule-a'],
  name: 'Rule A',
  table: 'incident',
  ...commonConfig,
  order: 100,
  script: myHandler,
})
```

### TypeScript types

Run `npx now-sdk dependencies` to fetch type definitions for your instance's tables, columns, and platform APIs. This enables IDE autocompletion and type checking for table names, field references, and API calls in your fluent definitions.

## SDK version notes

The current recommended version is **4.3.0**. Key capabilities by version:

- **3.0** — Initial public release. Introduced fluent language, `init`/`build`/`install` workflow, and the `transform` command.
- **4.0** — Breaking changes: moved from method-chaining to object-based fluent API, new `now.config.json` format, added `download` and `pack` commands.
- **4.1** — Flow Designer support via fluent API.
- **4.2** — Service Catalog item definitions, UI Page support (including React-based UI Pages), Import Set fluent API.
- **4.3** — UI Policy support, stability improvements, enhanced `transform` for more artifact types.

When upgrading across major versions, consult the SDK changelog for migration steps. The `transform` command can help re-generate fluent files from instance artifacts if manual migration is too complex.
