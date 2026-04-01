# SYS_SECURITY_DATA_FILTER

## Properties Reference

The Record API accepts the following properties:

| Property | Type             | Required | Default | Description                                                                                                           |
| -------- | ---------------- | -------- | ------- | --------------------------------------------------------------------------------------------------------------------- |
| `$id`    | `Now.ID[string]` | Yes      | -       | Unique identifier for the metadata object. Format: `$id: Now.ID['value']`                                             |
| `table`  | `string`         | Yes      | -       | The ServiceNow table name where this record will be created. For Security Data Filter, use `sys_security_data_filter` |
| `data`   | `object`         | Yes      | -       | Object containing field names and values for the target table                                                         |
| `$meta`  | `object`         | No       | -       | Metadata for installation control (demo, first install)                                                               |

### Table-Specific Fields (`sys_security_data_filter`)

Security Data Filter fields available in the `sys_security_data_filter` table:

| Field                | Type        | Required | Default | Description                                                                           |
| -------------------- | ----------- | -------- | ------- | ------------------------------------------------------------------------------------- |
| `description`        | `string`    | Yes      | -       | Descriptive name for the security data filter                                         |
| `table_name`         | `string`    | Yes      | -       | Target table name where the filter applies                                            |
| `mode`               | `string`    | Yes      | -       | Filter mode: `if` (filter if condition met) or `unless` (filter unless condition met) |
| `security_attribute` | `reference` | Yes      | -       | Reference to sys_security_attribute record that defines the security condition        |
| `filter`             | `string`    | No       | -       | Encoded query condition that defines which records to filter                          |
| `active`             | `boolean`   | No       | true    | Whether the security data filter is active and will be applied                        |
| `show_in_ui`         | `boolean`   | No       | false   | Whether to show this filter in the UI for debugging purposes                          |

### Field Validation Requirements

- `table_name` must be a valid ServiceNow table name
- `mode` must be either `if` or `unless`
- `security_attribute` must reference a valid sys_security_attribute record
- `filter` must be a valid encoded query string if provided
- Use proper encoded query syntax for complex conditions

### Related Table: Security Attribute (`sys_security_attribute`)

Security Attributes define the conditions that determine when filters apply:

| Field         | Type      | Required | Description                              |
| ------------- | --------- | -------- | ---------------------------------------- |
| `name`        | `string`  | Yes      | Unique name for the security attribute   |
| `description` | `string`  | No       | Description of what the attribute checks |
| `type`        | `string`  | No       | Type of security attribute               |
| `active`      | `boolean` | No       | Whether the attribute is active          |

## Examples

### Basic Security Data Filter

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const filterFinancialRecords = Record({
  $id: Now.ID["filter-financial-records"],
  table: "sys_security_data_filter",
  data: {
    description:
      "Restrict access to high-value financial transactions to authorized personnel only",
    table_name: "finance_transaction",
    mode: "unless",
    security_attribute: hasFinanceRoleAttribute, // Direct reference to Security Attribute
    filter: "amount>10000^ORclassification=confidential",
    active: true,
    show_in_ui: false
  }
});
```

### Advanced Example: Personal Information Protection

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const filterPersonalData = Record({
  $id: Now.ID["filter-personal-data"],
  table: "sys_security_data_filter",
  data: {
    description:
      "Restrict access to employee personal information to HR personnel only",
    table_name: "hr_profile",
    mode: "unless",
    security_attribute: hasHRRoleAttribute, // Direct reference to Security Attribute
    filter:
      "contains_pii=true^ORsalary!=NULL^ORssn!=NULL^ORemployee_idDYNAMIC90d1921e5f510100a9ad2572f2b477fe", // Include current user check
    active: true,
    show_in_ui: false
  }
});
```

### Advanced Example: Multi-Condition Filter with Dynamic Conditions

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const filterSensitiveData = Record({
  $id: Now.ID["filter-sensitive-data"],
  table: "sys_security_data_filter",
  data: {
    description:
      "Hide sensitive incidents from non-security personnel unless assigned to current user",
    table_name: "incident",
    mode: "if",
    security_attribute: hasSecurityRoleAttribute, // Direct reference to Security Attribute
    filter:
      "(category=security^ORcategory=privacy^ORimpact=1)^assigned_to!=DYNAMIC90d1921e5f510100a9ad2572f2b477fe", // Dynamic condition for current user
    active: true,
    show_in_ui: true // Show for debugging
  }
});
```

### Security Attribute Definition

The Security Attribute referenced in the examples above:

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

export const hasAdminRoleAttribute = Record({
  $id: Now.ID["has-admin-role-attribute"],
  table: "sys_security_attribute",
  data: {
    name: "HasAdminRole",
    description: "Check if user has admin role",
    type: "compound",
    condition: "Role=admin",
    active: true
  }
});
```
