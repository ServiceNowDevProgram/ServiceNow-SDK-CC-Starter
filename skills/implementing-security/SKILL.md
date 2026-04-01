---
name: implementing-security
description: Guide for implementing ServiceNow application security using ACLs [sys_security_acl], Roles [sys_user_role], Security Attributes [sys_security_attribute], and Security Data Filters [sys_security_data_filter]. Use when user mentions security, access control, permissions, ACLs, roles, data filters, or restricting access. Also use proactively when creating tables with sensitive data or applications needing role-based access control.
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


# Implementing Security

## When to use this skill

- When securing tables, fields, or resources with access control rules
- When creating roles for an application
- When implementing row-level data filtering based on user attributes
- When defining reusable security predicates (Security Attributes)

## Instructions

### Overall security model

1. **Start with Roles:** Define roles first — they're required by ACLs and referenced by Security Attributes.
2. **Then ACLs:** Create ACL rules to secure tables, fields, and resources. Each ACL secures one operation on one object.
3. **Use Security Attributes for reusable predicates:** When the same role/condition logic appears in multiple ACLs, extract it into a Security Attribute.
4. **Add Data Filters for row-level security:** When users should see only certain rows (not the whole table), add Security Data Filters paired with Deny ACLs.

### Roles

1. **Always prefix role names with the application scope** — e.g., `x_my_scope.manager`.
2. **Use `containsRoles` for inheritance** — a supervisor role can contain a manager role.
3. **For existing platform roles** (e.g., `itil`), use the `runQuery` tool to fetch the `sys_id` from `sys_user_role`.

See `references/role.md` for detailed Role instructions.

### ACLs

1. **ACLs require at least one of:** roles, security attribute, condition, or script.
2. **The Trinity:** All specified conditions (roles AND condition AND script) must evaluate to true to grant access.
3. **For table-level access:** Use `type: 'record'` and omit `field`. For field-level, set `field` to the column name or `"*"` for all fields.
4. **One ACL per operation:** Create separate ACLs for read, write, delete, etc.
5. **Use `roles` for role checks, not scripts** — only use scripts for complex business logic (e.g., ownership checks).
6. **Use Deny-Unless ACLs** (`decisionType: 'deny'`) for layered security — they evaluate before Allow ACLs.
7. **Deny-Unless ACLs do not grant access alone** — at least one Allow ACL must also match.

See `references/acl.md`, `references/deny-unless.md`, and `references/query-acls.md` for detailed ACL patterns.

### Security Attributes

1. **Use the Record API** on `sys_security_attribute` — no dedicated Fluent API exists.
2. **Prefer `compound` type** — it's the only type that can be referenced in ACLs and Data Filters.
3. **Use `condition` field for compound types** with encoded query syntax (e.g., `"Role=manager^ORRole=admin"`).
4. **Use script-based types** (`true|false`, `string`, `integer`, `list`) only for complex logic that compound conditions can't express. These cannot be used in ACLs/Data Filters.
5. **Never use `current` in security attribute scripts** — attributes don't have record context.
6. **Set `is_dynamic: false`** for role/group checks that can be cached per session.

### Security Data Filters

1. **Use the Record API** on `sys_security_data_filter`.
2. **Always pair with Deny ACLs** — Data Filters alone don't provide complete security.
3. **Choose the right `mode`:** `"unless"` filters records unless the security attribute matches (most common). `"if"` filters when the attribute matches.
4. **`security_attribute` is required** — always reference a Security Attribute (compound type).
5. **Use dynamic conditions** (e.g., `fieldnameDYNAMIC90d1921e5f510100a9ad2572f2b477fe` for current user) instead of hardcoded values.
6. **Use indexed columns in filters** — complex filters on unindexed columns impact performance.

## Key concepts

### Security layer hierarchy

| Layer               | API                                  | Purpose                              |
| ------------------- | ------------------------------------ | ------------------------------------ |
| Roles               | `Role()`                             | Define personas with permissions     |
| ACLs                | `Acl()`                              | Control access to objects/operations |
| Security Attributes | Record on `sys_security_attribute`   | Reusable security predicates         |
| Data Filters        | Record on `sys_security_data_filter` | Row-level filtering                  |

### ACL evaluation order

1. Deny-Unless ACLs evaluate first — if any fail, access is denied
2. Allow-If ACLs evaluate second — at least one must pass to grant access
3. Within each ACL: roles, condition, and script ALL must pass

### Security Attribute types

| Type                          | Use for                                 | Can use in ACLs? |
| ----------------------------- | --------------------------------------- | ---------------- |
| `compound`                    | Role/group conditions via encoded query | Yes              |
| `true\|false`                 | Complex boolean logic via script        | No               |
| `string` / `integer` / `list` | Value calculations                      | No               |

## Avoidance

- **Never create roles without scope prefix** — use `x_scope.role_name` format
- **Never use scripts in ACLs for simple role checks** — use the `roles` property
- **Never rely on Data Filters alone** — always pair with Deny ACLs
- **Never use `current` in Security Attribute scripts** — no record context available
- **Never hardcode user IDs or names** in ACL scripts or Data Filter conditions
- **Never use non-compound Security Attributes in ACLs** — only compound type is supported

## Next Steps

- For ACL properties and code examples, use `get_knowledge_source` tool to get the ACL knowledge source.
- For Role properties and examples, use `get_knowledge_source` tool to get the ROLE knowledge source.
- For Security Attribute properties and examples, use `get_knowledge_source` tool to get the SYS_SECURITY_ATTRIBUTE knowledge source.
- For Security Data Filter properties and examples, use `get_knowledge_source` tool to get the SYS_SECURITY_DATA_FILTER knowledge source.
- For ACL patterns, see `references/acl.md`, `references/deny-unless.md`, `references/query-acls.md`.
- For Role patterns, see `references/role.md`.
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
