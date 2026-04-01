---
name: acl
description: Guide for creating ServiceNow ACL rules [sys_security_acl] to secure tables, fields, records, and resources. Use when user mentions ACLs, security, access control, permissions, roles, or restrict access. Also use proactively when creating tables with sensitive data, custom resources (UI Pages, REST endpoints, Script Includes), or applications requiring role-based access control. Includes Deny-Unless ACLs for layered security and Query ACLs to prevent data extraction attacks.
---

# Access Control Lists (ACL)

## When to use this skill

- When user explicitly asks for ACLs or security rules
- When creating or modifying ACL rules to secure tables, fields, or resources
- When implementing role-based access control
- When implementing conditional access based on user roles, permissions, or other table fields
- When implementing script-based access control using custom scripts or script includes.
- When setting up Deny-Unless ACLs for layered security
- When configuring Query ACLs to prevent data extraction attacks

## Instructions

1. **Dependency:** This skill requires Role objects defined via the `role` skill.
2. **The Trinity:** To grant access, the `roles`, `condition`, and `script` must all evaluate to true.
3. **Field vs Table:**
   - For table-level access, leave the `field` property empty.
   - For all fields, use `field: "*"`.

## Key concepts

### ACL Requirements

All access control list rules specify:

- The decision type, rule type and operation which defines the ACL
- The object being secured
- The conditions required to access the object

### Components of ACL

- Decision type
- Object
- Operation
- Conditions

#### Decision Type

The decision type defines whether users are allowed to access the object if conditions are met or denies access the object unless conditions are met.

- Deny-Unless: Restrict access to resource by explicitly denying access unless conditions are passed. Check `references/deny-unless.md` for more information.
- Allow-If: Allow access to resource if conditions are passed.

#### Object

The object is the target to which access needs to be controlled. Each object consists of a type and name that uniquely identifies a particular table, field, or record.
Type of objects:

- record
- rest_endpoint
- ui_page
- processor
- graphql
- pd_action
- ux_data_broker
- ux_page
- ux_route
- client_callable_flow_object
- client_callable_script_include

Default: record
Note: After creating an ACL rule, if you want to change the object, you must delete the ACL and create a new one with the correct object.

#### Operation

The operation that this ACL rule secures. An ACL rule can only secure one operation. To secure multiple operations, create a separate ACL rule for each.

- Each operation describes a valid action the system can take on the specified object.
- Some objects, such as records, support multiple operations, while other objects, such as a REST_Endpoint, only support one operation.

Check acl knowledge source (using `get_knowledge_source` tool) for more information.
Note: The operation must be execute if the object type property is client_callable_flow_object, client_callable_script_include, graphql, processor, or rest_endpoint.

#### Conditions

The conditions specify when someone can access the named object and operation. Security administrators can specify condition requirements by adding:

- One or more user roles to the Requires role list
- One or more security attributes need to be evaluated to be true.
- One or more data conditions.
- A script that evaluates to true or false or sets the answer variable to true or false.

### Deny-Unless ACLs

Deny-Unless ACLs are evaluated with a "deny-unless" approach. The ACL defines the users that will NOT be denied. Said another way, the user will be denied access unless the role, condition, and script requirements are met.

**IMPORTANT**: Deny-Unless ACLs will take priority against Allow-If ACLs in ACL Evaluation, as it will be evaluated first.

Check `references/deny-unless.md` for more information.

### Query ACLs

Query ACLs allow you to define more granular access control by explicitly defining who can query the data. Consider query ACLs when some users have access to some rows or columns and not others.
They protect against blind query attacks:

- `query_match` - Controls safe operators (EQUALS, IN)
- `query_range` - Controls dangerous operators (CONTAINS, >=)

Check `references/query-acls.md` for more information.

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the ACL knowledge source.
- For Deny-Unless ACLs, see `references/deny-unless.md`.
- For Query ACLs, see `references/query-acls.md`.
- For Role, use role skill.
- Use `load_skill_resource` with skill name "acl" and the file path to get references.
