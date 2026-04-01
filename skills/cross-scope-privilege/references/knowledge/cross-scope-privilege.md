# CROSS_SCOPE_PRIVILEGE

The Cross-Scope Privilege API includes objects that define cross-scope privileges [sys_scope_privilege] for runtime access tracking. Runtime access tracking allows administrators to manage script access to application resources by creating a list of script operations and targets that the system authorizes to run.

## CrossScopePrivilege object

Configure cross-scope privileges [sys_scope_privilege] that determine which script operations and targets the system allows to run in the application.

### Properties

| Name        | Type             | Description                                                                                                                                                                                                                                                                                                                                                               |
| ----------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $id         | String or Number | Required. A unique ID for the metadata object provided in the following format, where <value> is a string or number. `$id: Now.ID[<value>]` When you build the application, this ID is hashed into a unique sys_ID.                                                                                                                                                       |
| status      | String           | Required. The authorization for this record. Valid values: `requested`, `allowed`, `denied`                                                                                                                                                                                                                                                                               |
| operation   | String           | Required. The operation that the script performs on the target. The target type determines the available operations. Tables (sys_db_object) support the read, write, create, and delete operations. Script includes (sys_script_include) and script objects (scriptable) only support the execute operation. Valid values: `create`, `delete`, `read`, `write`, `execute` |
| targetName  | String           | Required. The name of the table, script include, or script object being requested.                                                                                                                                                                                                                                                                                        |
| targetScope | String           | Required. The application scope from which resources are requested.                                                                                                                                                                                                                                                                                                       |
| targetType  | String           | Required. The type of request: script include, script object, or table. Valid values: `sys_script_include`, `scriptable`, `sys_db_object`                                                                                                                                                                                                                                 |
| $meta       | Object           | Metadata for the application metadata. With the installMethod property, you can map the application metadata to an output directory that loads only in specific circumstances. `$meta: { installMethod: 'String' }` Valid values for installMethod: `demo`, `first install`                                                                                               |

### Example

```javascript
CrossScopePrivilege({
  $id: Now.ID["cross_1"],
  status: "allowed",
  operation: "execute",
  targetName: "Script type",
  targetScope: "x_snc_example",
  targetType: "scriptable"
});
```
