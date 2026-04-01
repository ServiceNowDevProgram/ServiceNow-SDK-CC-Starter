# ROLE

## Role object

Create a role [sys_user_role] to control access to applications and their features.

Properties

| Name              | Type    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ----------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name              | String  | A name for the role beginning with the application scope in the following format: <scope>.<name>.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| assignableBy      | String  | Other roles that can assign this role to users.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| canDelegate       | Boolean | Flag that indicates if the role can be delegated to other users. Valid values: _ true: The role can be delegated to other users. _ false: The role can't be delegated to other users. Default: true                                                                                                                                                                                                                                                                                                                                                                                                |
| description       | String  | A description of what the role can access.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| elevatedPrivilege | Boolean | Flag that indicates whether manually accepting the responsibility of using the role before you can access the features of the role is required. For more information about elevated privileges. Valid values: _ true: You must manually accept the responsibility of using the role before you can access its features. _ false: You don't need to manually accept the responsibility of using the role to access its features. Default: false                                                                                                                                                     |
| grantable         | Boolean | Flag that indicates whether the role can be granted independently. Valid values: _ true: The role can be granted independently. _ false: The role can't be granted independently. Default: true                                                                                                                                                                                                                                                                                                                                                                                                    |
| containsRoles     | Array   | An array of Record<'sys_user_role'>, either sys_ids or Role objects. It is optional, it indicates other Role objects that this role contains. If using a new role, use the Role object. If using an existing role, use the `runQuery` tool to look up valid sys_id values from the `sys_user_role` table `- no`label` field for roles.                                                                                                                                                                                                                                                             |
| scopedAdmin       | Boolean | Flag that indicates whether the role is an Application Administrator role. Valid values: _ true: The role is an Application Administrator. _ false: The role isn't an Application Administrator. Default: false                                                                                                                                                                                                                                                                                                                                                                                    |
| $meta             | Object  | Metadata for the application metadata. With the installMethod property, you can map the application metadata to an output directory that loads only in specific circumstances. `javascript $meta: { installMethod: 'String' } ` Valid values for installMethod: _ demo: Outputs the application metadata to the metadata/unload.demo directory to be installed with the application when the Load demo data option is selected. _ first install: Outputs the application metadata to the metadata/unload directory to be installed only the first time an application is installed on an instance. |

### Examples

```javascript
import { Role } from "@servicenow/sdk/core";

const managerRole = Role({
  name: "x_snc_example.manager"
});

const adminRole = Role({
  name: "x_snc_example.admin",
  containsRoles: [managerRole]
});
```

#### Create a named Role with or without contained roles

```typescript
import { Role } from "@servicenow/sdk/core";

const managerRole = Role({
  name: "sn_xxxx.manager" // role name
});

const supervisorRole = Role({
  name: "sn_xxxx.supervisor", // role name
  containsRoles: [managerRole, "282bf1fac6112285017366cb5f867469"] // managerRole Fluent object and sys_id of itil role. This establishes a hierarchy where the sn_xxxx.supervisor role encompasses the sn_xxxx.manager and itil roles. needs to use runQuery tool to get the EXACT sys_id of the record in table `sys_user_role`
});
```

## Notes

- Use the `runQuery` tool to retrieve the exact `sys_id` of the role from the `sys_user_role` table.
