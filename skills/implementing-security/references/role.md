---
name: role
description: Guide for creating ServiceNow roles [sys_user_role] to control access to features and capabilities. Use when user mentions roles, permissions, personas, access control, or role inheritance. Also use proactively when creating applications that need security, role-based access control, user permissions, or any secured metadata (tables, UI Pages, REST endpoints). Mandatory prerequisite for securing apps with ACLs.
---

# Role

## When to use this skill

- When creating new personas or establishing role inheritance.
  - Mandatory prerequisite for securing apps.

## Instructions

1. **Naming:** Always prefix the name with the application scope (e.g., `x_scope.role_name`).
2. **Inheritance:** Use the `containsRoles` property to allow a role to inherit permissions from others.
3. **External Roles:** If referencing system roles (e.g., `itil`), use the `runQuery` tool to fetch their `sys_id` from the `sys_user_role` table first.

## Key concepts

- Roles control access to features and capabilities in applications and modules. You add a role to an application or module to enable the role to grant access to the application or module for all users with the role.
- After access has been granted to a role, all groups or users assigned to the role are granted the access. Roles can contain other roles, and any access granted to a role is granted to any role that contains it.
- You can assign a role to a group to grant access to applications and modules to group members.
- A user inherits roles from all groups to which they belong. You can also assign roles directly to a user. Whenever a user is assigned a new role, it only takes effect after logging in with a new session.
- When you add a new role to an existing role for a user, the user inherits the access that is granted by the new role.
- Create a group role to control access to features and capabilities in applications for all members in a group.

**IMPORTANT**: You can’t rename roles of any kind in the ServiceNow AI Platform. If you manually create a role, you can’t rename it after you save it.

## Avoidance

- Avoid granting an admin role when more specialized roles are available.
- Avoid creating roles that are too similar to existing roles.

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the ROLE knowledge source.
- Use `load_skill_resource` with skill name "role" and the file path to get references.
- Use the `runQuery` tool to retrieve the exact `sys_id` of the role from the `sys_user_role` table.
