# ACL

## ACL object

Configure a custom ACL rule [sys_security_acl] to secure access to new objects or to change the default security behavior.

ACLs must include one or more roles, a security attribute, a condition, or a script.

Properties

| Name              | Type             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ----------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $id               | String or Number | Required. A unique ID for the metadata object provided in the following format, where `<value>` is a string or number. `javascript $id: Now.ID[<value>] ` When you build the application, this ID is hashed into a unique sys_ID.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| table             | String           | The name of the table to which the ACL applies. This value MUST be a string literal for the name of the table - it should not be a reference to a table object. This property only applies and is required if the type property is one of the following values: ux_data_broker, ux_page, ux_route, pd_action, or record.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| field             | String           | The name of a field on the table to secure. You can use the wildcard character (`"*"`) to select all fields. This property only applies and is required if the type property is one of the following values: ux_data_broker, ux_page, ux_route, pd_action, or record. Note: For table level ACLs the field should not be included.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| name              | String           | The name of the ACL. This property only applies and is required if the type property is one of the following values: rest_endpoint, ui_page, processor, graphql, client_callable_flow_object, or client_callable_script_include.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| operation         | String           | Required. The operation this ACL rule secures. An ACL rule can only secure one operation. To secure multiple operations, create a separate ACL rule for each. The operation must be execute if the type property is client_callable_flow_object, client_callable_script_include, graphql, processor, or rest_endpoint, Valid values: execute, create, read, write, delete, edit_task_relations, edit_ci_relations, save_as_template, add_to_list, report_on, list_edit, report_view, personalize_choices                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| type              | String           | Required. The type of object this ACL rule secures. The type determines what operations are available. After creating an ACL rule, if you want to change the type, you must delete the ACL and create a new one with the correct type. Valid values: `record`, `rest_endpoint`, `ui_page`, `processor`, `graphql`, `pd_action`, `ux_data_broker`, `ux_page`, `ux_route`, `client_callable_flow_object`, `client_callable_script_include`. If this ACL is being defined to secure a table, use the `record` type.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| active            | Boolean          | Flag that indicates whether the ACL rule is enforced. Valid values: _ true: The ACL rule is enforced. _ false: The ACL rule isn't enforced. Default: true                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| roles             | Array            | The variable identifiers of Role objects or sys_ids of roles that a user must have to access the object. ACLs must include one or more roles, a security attribute, a condition, or a script. Note: Users with the admin role always pass this permissions check because the admin role automatically grants users all other roles.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| script            | Script           | A function or inline script. The custom script should define the permissions required to access the object. ACLs must include one or more roles, a security attribute, a condition, or a script. Note: If the type property is graphql, scripts aren't supported. For functions, use the name of a function, function expression, or default function exported from a JavaScript module and imported into the .now.ts file. Inline scripts can use string literals or template literals for scripts with multiple lines. ``javascript script: `Script`, `` The script can use the values of the `current` and `previous` global variables and system properties. The script must generate a true or false response in one of two ways: _ return an `answer` variable set to a value of true or false _ evaluate to true or false In either case, users only gain access to the object when the script evaluates to true and the user meets any conditions the ACL rule has. Both the conditions and the script must evaluate to true for a user to access the object. Note: If the evaluated item is in a related list, `current` points to the item the related list is on, not to the current item the ACL is for. However, If the item you are evaluating the ACL for is not in a related list, `current` points to the actual item. |
| adminOverrides    | Boolean          | Flag that indicates whether users with the admin role automatically pass the permissions check for this ACL rule. Valid values: _ true: Administrators automatically pass the permissions check. _ false: Administrators must meet the permissions defined in this ACL rule to gain access to the secured object. Use the condition or script properties to create a permissions check that administrators must pass. Admin users pass regardless of what script or role restrictions apply. However, the nobody role, which only ServiceNow personnel can assign, takes precedence over the admin override option. If an ACL is assigned the nobody role, admin users cannot access the resource even when adminOverrides is true. Default: true                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| condition         | String           | A filter query that specifies the fields and values that must be true for users to access the object. ACLs must include one or more roles, a security attribute, a condition, or a script.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| description       | String           | A description of the object or permissions this ACL rule secures.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| localOrExisting   | String           | The type of security attribute to apply. Valid values: _ Local: A security attribute based on the condition property that is saved only for the ACL it is created in. _ Existing: An existing security attribute to reference in the securityAttribute property. Default: Local                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| securityAttribute | String           | Pre-defined conditions to use. For example, whether a user is impersonating another user. ACLs must include one or more roles, a security attribute, a condition, or a script. Note: For security attributes with the Is localized field set to true, the localOrExisting property of the ACL should be set to Local. If the Is localized field is false, the localOrExisting property should be set to Existing. Valid values: role_explicit, group_explicit, user_is_authenticated, impersonating, interactive_session, has_admin_role, role, logged_in, network_criteria, group, allow_unauth_roleless_acl                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| decisionType      | String           | Whether the ACL should allow or deny access. Valid values: _ allow: The ACL allows access. _ deny: The ACL denies access. Default: allow                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| $meta             | Object           | Metadata for the application metadata. With the installMethod property, you can map the application metadata to an output directory that loads only in specific circumstances. `javascript $meta: { installMethod: 'String' } ` Valid values for installMethod: _ demo: Outputs the application metadata to the metadata/unload.demo directory to be installed with the application when the Load demo data option is selected. _ first install: Outputs the application metadata to the metadata/unload directory to be installed only the first time an application is installed on an instance.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

### Example

```javascript
import { Acl } from "@servicenow/sdk/core";

export default Acl({
  $id: Now.ID["task_delete_acl"],
  active: true,
  adminOverrides: true,
  type: "record",
  table: "task",
  field: "description",
  operation: "delete",
  roles: [adminRole, managerRole]
});
```

The roles referenced are defined using the Role object:

```javascript
import { Role } from "@servicenow/sdk/core";

const managerRole = Role({
  $id: Now.ID["manager_role"],
  name: "x_snc_example.manager"
});

const adminRole = Role({
  $id: Now.ID["admin_role"],
  name: "x_snc_example.admin",
  containsRoles: [managerRole]
});
```

### Additional Examples

Example 1: Field-level ACL with multiple roles

```javascript
import { Acl, Role } from "@servicenow/sdk/core";

const travelAgentRole = Role({
  $id: Now.ID["travel_agent_role"],
  name: "sn_travel_app.travel_agent"
});

const travelManagerRole = Role({
  $id: Now.ID["travel_manager_role"],
  name: "sn_travel_app.travel_manager"
});

// ACL for booking status field - only agents and managers can modify
export default Acl({
  $id: Now.ID["booking_status_write_acl"],
  type: "record",
  table: "sn_travel_app_booking",
  field: "status",
  operation: "write",
  roles: [travelAgentRole, travelManagerRole], // Use roles property
  active: true,
  adminOverrides: true
});
```

Example 2: When script is appropriate (ownership check)

```javascript
// Complex logic: user must own the record AND it must be in pending status
export default Acl({
  $id: Now.ID["booking_delete_owner_acl"],
  type: "record",
  table: "sn_travel_app_booking",
  operation: "delete",
  roles: [travelerRole], // Still define role requirement
  script: `
        // Script is appropriate here for ownership check
        var isOwner = (current.sys_created_by == gs.getUserName());
        var isPending = (current.status == 'pending');
        answer = isOwner && isPending;
    `,
  active: true,
  adminOverrides: true
});
```

Key takeaway: Always use the `roles` property for role checks. Only use scripts when you need complex business logic that goes beyond simple role verification.

### Deny-Unless ACL

Deny-Unless ACLs (`decisionType: 'deny'`) evaluate before Allow ACLs and deny access unless conditions are met, requiring at least one Allow ACL to grant actual resource access.

**Important**: Deny-Unless ACLs on their own do not grant access for any resource. If Deny ACL evaluation returns true, it only means that access is not denied. At least one Allow ACL evaluation need to return true in order to grant access.

#### Examples

The following examples demonstrate how to implement Deny-Unless ACLs in various scenarios:

Example 1: Basic Deny-Unless ACL with Role Requirement

```javascript
import { Acl, Role } from "@servicenow/sdk/core";

const itilRole = Role({
  $id: Now.ID["itil_role"],
  name: "itil"
});

const adminRole = Role({
  $id: Now.ID["admin_role"],
  name: "admin"
});

// Access would be denied unless user has itil role
export const incidentDenyUnlessItil = Acl({
  $id: Now.ID["incident_deny_unless_itil"],
  type: "record",
  table: "incident",
  operation: "read",
  decisionType: "deny", // This makes it a Deny-Unless ACL
  roles: [itilRole], // Only users with itil role are not denied
  description: "Deny access to incidents unless user has itil role",
  active: true,
  adminOverrides: true
});

// Corresponding Allow ACL: Grant actual access to specific users
export const incidentAllowRead = Acl({
  $id: Now.ID["incident_allow_read"],
  type: "record",
  table: "incident",
  operation: "read",
  decisionType: "allow", // Traditional Allow ACL
  roles: [itilRole], // Grant access to itil users
  active: true,
  adminOverrides: true
});
```

Example 2: Multiple Deny ACLs (AND Logic)

```javascript
// Scenario: Incident deletion requires both itil AND incident_manager roles
// This creates a secure baseline - users must have both roles to not be denied

// Deny ACL 1: Require itil role
export const incidentDenyUnlessItil = Acl({
  $id: Now.ID["incident_delete_deny_itil"],
  type: "record",
  table: "incident",
  operation: "delete",
  decisionType: "deny",
  roles: [itilRole],
  description: "Deny incident deletion unless user has itil role",
  active: true
});

// Deny ACL 2: Require incident_manager role
export const incidentDenyUnlessManager = Acl({
  $id: Now.ID["incident_delete_deny_manager"],
  type: "record",
  table: "incident",
  operation: "delete",
  decisionType: "deny",
  roles: [incidentManagerRole],
  description: "Deny incident deletion unless user has incident_manager role",
  active: true
});

// Allow ACL: Grant actual deletion rights
export const incidentAllowDelete = Acl({
  $id: Now.ID["incident_allow_delete"],
  type: "record",
  table: "incident",
  operation: "delete",
  decisionType: "allow",
  roles: [incidentManagerRole], // Only managers can actually delete
  condition: "state!=6", // Cannot delete closed incidents
  active: true
});
```

Example 3: Field-Level Deny ACL

```javascript
// Deny access to sensitive incident fields unless user has privacy_officer role
export const incidentSensitiveFieldDeny = Acl({
  $id: Now.ID["incident_sensitive_field_deny"],
  type: "record",
  table: "incident",
  field: "work_notes",
  operation: "read",
  decisionType: "deny",
  roles: [privacyOfficerRole, itilRole], // Must have either role
  description:
    "Deny access to work_notes unless user has privacy_officer and itil roles",
  active: true,
  adminOverrides: true
});
```

### Query ACLs

Query ACLs control query operations against columns to prevent blind query attacks and data extraction.

- query_match: Controls safe operators (EQUALS, IN, etc.)
- query_range: Controls dangerous operators (CONTAINS, >=, etc.)

Use for sensitive columns with partial access controls.

```javascript
// Prevent salary range queries to avoid data extraction
export const payrollSalaryQueryRange = Acl({
  $id: Now.ID["payroll_salary_query_range"],
  type: "record",
  table: "payroll",
  field: "salary",
  operation: "query_range",
  decisionType: "deny",
  roles: [hrAdminRole],
  active: true
});
```

Use the same tool to fetch related documents before proceeding:

- Role
