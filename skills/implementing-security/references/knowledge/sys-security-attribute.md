# SYS_SECURITY_ATTRIBUTE

## Properties Reference

The Record API accepts the following properties:

| Property | Type             | Required | Default | Description                                                                                                       |
| -------- | ---------------- | -------- | ------- | ----------------------------------------------------------------------------------------------------------------- |
| `$id`    | `Now.ID[string]` | Yes      | -       | Unique identifier for the metadata object. Format: `$id: Now.ID['value']`                                         |
| `table`  | `string`         | Yes      | -       | The ServiceNow table name where this record will be created. For Security Attribute, use `sys_security_attribute` |
| `data`   | `object`         | Yes      | -       | Object containing field names and values for the target table                                                     |
| `$meta`  | `object`         | No       | -       | Metadata for installation control (demo, first install)                                                           |

### Table-Specific Fields (`sys_security_attribute`)

Security Attribute fields available in the `sys_security_attribute` table:

| Field                 | Type      | Required | Default  | Description                                                                                           |
| --------------------- | --------- | -------- | -------- | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| `name`                | `string`  | Yes      | -        | Unique name for the security attribute (used to reference in ACLs)                                    |
| `type`                | `string`  | Yes      | `string` | Type of security attribute: `true                                                                     | false`, `string`, `integer`, `list`, `compound` |
| `description`         | `string`  | No       | -        | Descriptive text explaining the purpose and behavior of the security attribute                        |
| `label`               | `string`  | No       | -        | Display label for the security attribute in the UI                                                    |
| `active`              | `boolean` | No       | `true`   | Whether the security attribute is active and can be used                                              |
| `script`              | `string`  | No       | -        | JavaScript code that defines the security logic (required for `true                                   | false`, `string`, `integer`, `list` types)      |
| `condition`           | `string`  | No       | -        | Encoded query condition (required for `compound` type) that defines the security logic                |
| `is_localized`        | `boolean` | No       | `false`  | Whether this attribute is created locally for a specific ACL                                          |
| `is_dynamic`          | `boolean` | No       | `true`   | Whether the attribute supports dynamic evaluation with parameters                                     |
| `is_system`           | `boolean` | No       | `false`  | Whether this is a system-provided security attribute                                                  |
| `lookup_table`        | `string`  | No       | -        | Table name for lookup-based security attributes (used with `string`, `integer`, `list` types)         |
| `lookup_table_column` | `string`  | No       | -        | Column name in the lookup table for security evaluation (used with `string`, `integer`, `list` types) |

### Field Validation Requirements

- `name` must be unique across all security attributes
- `type` must be one of: `true|false`, `string`, `integer`, `list`, `compound`
- For `compound` type:
  - `condition` field is required and must contain valid encoded query syntax
  - `script`, `lookup_table`, `lookup_table_column` fields are ignored
- For `true|false` type:
  - `script` field is required and must return a boolean value via `answer` variable
  - `condition`, `lookup_table`, `lookup_table_column` fields are ignored
- For `string`, `integer`, `list` types:
  - `script` field is required and must return value matching the specified type
  - `lookup_table` and `lookup_table_column` are optional for value comparison
  - `condition` field is ignored
- `script` must contain valid JavaScript code when used
- Important: Using 'current' in the script or included script-includes is not allowed
- Important: Only `compound` type security attributes can be referenced in ACLs and Data Filters

### Performance and Caching

Security Attributes provide significant performance benefits through caching:

- Session-level caching (`is_dynamic: false`): Evaluated once per user session
- Transaction-level caching (`is_dynamic: true`): Evaluated once per transaction
- Reuse across ACLs: Same attribute used in multiple ACLs leverages cached result

### Compound Type Security Attribute (Recommended)

Examples

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const hasManagerRole = Record({
  $id: Now.ID["has-manager-role"],
  table: "sys_security_attribute",
  data: {
    name: "HasManagerRole",
    type: "compound",
    label: "Has Manager Role",
    description: "Checks if the current user has the manager role",
    condition: "Role=manager",
    active: true,
    is_dynamic: false // Role checks can be cached per session
  }
});
```

### Multiple Role Security Attribute

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const hrManagerOrAdmin = Record({
  $id: Now.ID["hr-manager-or-admin"],
  table: "sys_security_attribute",
  data: {
    name: "HRManagerOrAdmin",
    type: "compound",
    label: "HR Manager or Admin",
    description: "User must have HR manager role or admin role",
    condition: "Role=hr_manager^ORRole=admin",
    active: true,
    is_dynamic: false // Role checks can be cached per session
  }
});
```

### True|False Type Security Attribute (Boolean Script)

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const hasFinanceRole = Record({
  $id: Now.ID["has-finance-role"],
  table: "sys_security_attribute",
  data: {
    name: "HasFinanceRole",
    type: "true|false",
    label: "Has Finance Role",
    description:
      "Checks if user has finance role or is member of finance group",
    script:
      'answer = gs.hasRole("finance") || gs.getUser().isMemberOf("finance_users");',
    active: true,
    is_dynamic: false // Role/group checks can be cached per session
    // Note: condition, lookup_table, lookup_table_column fields are ignored for true|false type
  }
});
```

### String Type Security Attribute with Lookup

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const userRegionAttribute = Record({
  $id: Now.ID["user-region-attribute"],
  table: "sys_security_attribute",
  data: {
    name: "UserRegion",
    type: "string",
    label: "User Region",
    description: "Returns the region of the current user",
    script: "answer = gs.getUser().getLocation().getRegion();",
    lookup_table: "sys_choice",
    lookup_table_column: "value",
    active: true,
    is_dynamic: false // User region rarely changes
    // Note: condition field is ignored for string type
  }
});
```

### Integration with ACLs and data filters

Security Attributes are commonly used in ACLs and security data filters through the `security_attribute` property:

```typescript
// Security Attribute: Department-based access
export const hasDepartmentAccess = Record({
  $id: Now.ID["has-department-security-attr"],
  table: "sys_security_attribute",
  data: {
    name: "HasDepartmentAccess",
    type: "compound",
    label: "Has Department Access",
    description: "User can only access records from their department",
    condition: "Department=javascript:gs.getUser().getDepartment()",
    is_dynamic: false, // Department rarely changes during session
    active: true
  }
});
```

```typescript
// Example ACL using a custom security attribute
export const incidentReadACL = Acl({
  $id: Now.ID["incident-read-acl"],
  type: "record",
  table: "incident",
  operation: "read",
  security_attribute: hasDepartmentAccess, // Direct reference to security attribute
  local_or_existing: "Existing",
  active: true
});
```

```typescript
// Data Filter using the security attribute
export const hrProfileDepartmentFilter = Record({
  $id: Now.ID["hr-profile-department-filter"],
  table: "sys_security_data_filter",
  data: {
    name: "HR Profile Department Filter",
    table: "hr_profile",
    mode: "if",
    security_attribute: hasDepartmentAccess, // Direct reference to security attribute
    description: "Filter HR profiles to show only same department records",
    active: true
  }
});
```
