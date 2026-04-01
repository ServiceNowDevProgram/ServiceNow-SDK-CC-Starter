---
name: scripted-rest-api
description: Guide for creating ServiceNow Scripted REST APIs [sys_ws_definition] to define custom web service endpoints. Use when user mentions REST APIs, custom endpoints, web services, HTTP methods, API versioning, or external integrations. Also use proactively when building features that need to expose data or operations to external systems.
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


# Scripted REST APIs

## When to use this skill

- When creating custom REST API endpoints for external integrations
- When exposing application data or operations as web services
- When building APIs with versioning, parameters, and custom headers
- When securing API access with ACLs and authentication

## Instructions

1. **Structure your API hierarchically:** `RestApi` (the service) contains `routes` (the endpoints), which contain `parameters` and `headers`. Each level needs its own `$id`.
2. **Keep handler scripts in modules:** Import handler functions from `src/server/` files rather than writing inline scripts. This keeps route definitions clean and scripts testable.
3. **Use path parameters for resource identifiers:** Define path params with `{id}` syntax in the route path (e.g., `/items/{id}`). Use query parameters for filtering and pagination.
4. **Version your API from the start:** Use the `versions` array on the RestApi and set `version` on each route. This generates versioned URIs (`/api/{scope}/v{version}/{serviceId}/...`) and allows non-breaking evolution.
5. **Set ACLs at the right level:** `enforceAcl` on RestApi applies to all routes. `enforceAcl` on individual routes overrides the API-level setting. Use route-level ACLs when different endpoints need different access control.
6. **One route per HTTP method per path:** Each route handles a single HTTP method (GET, POST, PUT, PATCH, DELETE). Create separate routes for different methods on the same path.

## Key concepts

Scripted REST APIs let you define custom web service endpoints in ServiceNow. The RestApi object defines the service, and routes define individual endpoints with their HTTP methods, scripts, and parameters.

### URI path construction

- With versioning: `/api/{scope_name}/v{version}/{serviceId}/{path}`
- Without versioning: `/api/{scope_name}/{serviceId}/{path}`

The `serviceId` must be unique within the API namespace — it's the key identifier in the URI.

### Security model

- **`authorization`** (on route): Whether users must be authenticated. Default `true` — only set to `false` for truly public endpoints.
- **`authentication`** (on route): Whether ACLs are enforced. Default `true`.
- **`enforceAcl`** (on API or route): Which specific ACLs to apply. Reference ACL objects by variable identifier or sys_id.

### Versioning strategy

When using versions, every route must specify which `version` it belongs to. Mark one version as `isDefault: true` — clients can access it with or without the version prefix in the URI. Deprecated versions still serve requests but are marked in API documentation.

## Avoidance

- **Never skip ACLs for production APIs** — setting `enforceAcl: []` or `authentication: false` removes access control entirely
- **Never forget `version` on routes when using versions** — routes without a version won't be accessible through versioned URIs
- **Never use inline scripts for complex handlers** — import functions from server modules for maintainability and testability
- **Avoid exposing internal table structures** — transform data in handler scripts rather than returning raw GlideRecord fields

## Next Steps

- For Fluent API properties and code examples, use `get_knowledge_source` tool to get the SCRIPTED_REST_API knowledge source.
- For securing API endpoints with ACLs, see the `implementing-security` skill.
- For server-side utility functions called by handlers, see the `script-include` skill.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
