# RELATIONSHIP

## Properties Reference

The Record API accepts the following properties:

| Property | Type             | Required | Default | Description                                                                                                  |
| -------- | ---------------- | -------- | ------- | ------------------------------------------------------------------------------------------------------------ |
| `$id`    | `Now.ID[string]` | Yes      | -       | Unique identifier for the metadata object. Format: `$id: Now.ID['value']`                                    |
| `table`  | `string`         | Yes      | -       | The ServiceNow table name where this record will be created. For System Relationship, use `sys_relationship` |
| `data`   | `object`         | Yes      | -       | Object containing field names and values for the target table                                                |
| `$meta`  | `object`         | No       | -       | Metadata for installation control (demo, first install)                                                      |

### Table-Specific Fields (`sys_relationship`)

Relationship table fields available in the `sys_relationship` table:

| Field              | Type             | Required | Default                                   | Description                                                                              |
| ------------------ | ---------------- | -------- | ----------------------------------------- | ---------------------------------------------------------------------------------------- |
| `name`             | `TranslatedText` | No       | -                                         | Descriptive name for the relationship                                                    |
| `basic_apply_to`   | `TableName`      | No       | -                                         | The table where the relationship is defined (for basic relationships)                    |
| `basic_query_from` | `TableName`      | No       | -                                         | The table being referenced (for basic relationships)                                     |
| `apply_to`         | `Script`         | No       | -                                         | Script to determine which table the relationship applies to (for advanced relationships) |
| `query_from`       | `Script`         | No       | -                                         | Script to determine which table to query from (for advanced relationships)               |
| `reference_field`  | `FieldName`      | No       | -                                         | The field that contains the reference to the related record                              |
| `query_with`       | `Script`         | No       | `(function refineQuery(current, parent) { |

// Add your code here
})(current, parent);`| Script to refine the query for the relationship |
|`insert_callback`|`Script`| No | - | Script to execute when a record is inserted |
|`related_list`|`String`| No | - | Name of the related list (read-only, max length: 32) |
|`simple_reference`|`Boolean`| No | - | Whether this is a simple reference relationship |
|`advanced`|`Boolean`| No |`false` | Whether this is an advanced relationship |

### Dependent Tables

#### sys_ui_related_list

The `sys_ui_related_list` table stores configuration for related lists in ServiceNow:

| Field             | Type         | Required | Default  | Description                                                          |
| ----------------- | ------------ | -------- | -------- | -------------------------------------------------------------------- |
| `name`            | `TableName`  | No       | -        | The table where the related list appears (with table reference)      |
| `calculated_name` | `String`     | No       | -        | Auto-generated name combining table label and view (max length: 255) |
| `filter`          | `Conditions` | No       | -        | Filter conditions for the related list (max length: 3500)            |
| `view`            | `Reference`  | No       | -        | Reference to sys_ui_view table for view configuration                |
| `order_by`        | `String`     | No       | -        | Default order by field for the related list                          |
| `view_name`       | `String`     | No       | -        | Display name of the view                                             |
| `related_list`    | `String`     | No       | -        | Related list configuration (max length: 80)                          |
| `sys_user`        | `Reference`  | No       | -        | Reference to sys_user table for user-specific configurations         |
| `sys_domain_path` | `DomainPath` | No       | '/'      | Domain path for access control                                       |
| `sys_domain`      | `DomainId`   | No       | 'global' | Domain to which the related list belongs                             |
| `position`        | `Integer`    | No       | -        | Position/order of the related list                                   |

#### sys_ui_related_list_entry

The `sys_ui_related_list_entry` table stores individual entries for related lists:

| Field            | Type              | Required | Default                  | Description                                                                                                                                                                          |
| ---------------- | ----------------- | -------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `list_id`        | `Reference`       | No       | -                        | Reference to sys_ui_related_list table (cascade delete)                                                                                                                              |
| `related_list`   | `String`          | No       | -                        | The relationship identifier (max length: 165). Format: either REL:<relationship_sys_id> for non-referential relationships or tablename.reference_field for referential relationships |
| `filter`         | `Conditions`      | No       | -                        | Filter conditions with dynamic value based on related_list (max length: 3500). Used for additional filtering in both referential and non-referential relationships                   |
| `order_by`       | `String`          | No       | -                        | Default order by field for the related list                                                                                                                                          |
| `position`       | `Integer`         | No       | -                        | Position/order of the entry in the related list                                                                                                                                      |
| `sys_class_name` | `SystemClassName` | No       | `current.getTableName()` | Class name (max length: 80)                                                                                                                                                          |

### Field Validation Requirements

- `basic_apply_to` must be a valid ServiceNow table name
- `basic_query_from` must be a valid ServiceNow table name
- `reference_field` must be a valid field name in the apply_to table
- `query_with` if provided, must be valid JavaScript that returns a query
- `insert_callback` if provided, must be valid JavaScript
- Either use basic fields (`basic_apply_to`, `basic_query_from`) or advanced fields (`apply_to`, `query_from`)

## Examples

### Implicit Relationship using Reference Field

```typescript
import { Record } from "@servicenow/sdk/core";

// Assuming we have a custom_task table with a reference field 'department' pointing to department table
// We can create a related list entry using the table.field format:

export const departmentRelatedList = Record({
  $id: Now.ID["department_related_list"],
  table: "sys_ui_related_list",
  data: {
    name: "department", // The table where the related list appears
    view: "Default view"
  }
});

export const departmentRelatedListEntry = Record({
  $id: Now.ID["department_related_list_entry"],
  table: "sys_ui_related_list_entry",
  data: {
    list_id: `\${departmentRelatedList.$id}`,
    position: "0",
    related_list: "custom_task.department" // Using table.field format for implicit relationship
  }
});
```

### Relationship between custom tables

```javascript
// src/fluent/tables/student.now.ts
import {
  DateColumn,
  IntegerColumn,
  StringColumn,
  Table
} from "@servicenow/sdk/core";
export const sn_foo_student = Table({
  name: "sn_foo_student",
  label: "Student",
  schema: {
    id: StringColumn({
      label: "ID",
      mandatory: true,
      maxLength: 10
    }),
    department: StringColumn({
      label: "Department",
      mandatory: true,
      maxLength: 100,
      choices: {
        CS: { label: "Computer Science", sequence: 1 },
        IT: { label: "Information Technology", sequence: 2 },
        ECE: {
          label: "Electronics and Communication Engineering",
          sequence: 3
        },
        EEE: { label: "Electrical and Electronics Engineering", sequence: 4 }
      },
      dropdown: "dropdown_with_none"
    })
  },
  actions: ["read", "update", "delete", "create"],
  accessibleFrom: "public",
  allowWebServiceAccess: true,
  callerAccess: "none"
});

// src/fluent/tables/department.now.ts
import { StringColumn, Table } from "@servicenow/sdk/core";
export const sn_foo_department = Table({
  name: "sn_foo_department",
  label: "Department",
  schema: {
    id: StringColumn({
      label: "ID",
      mandatory: true,
      maxLength: 10
    })
  },
  actions: ["read", "update", "delete", "create"],
  accessibleFrom: "public",
  allowWebServiceAccess: true,
  callerAccess: "none"
});

// src/fluent/relationships/department_allocation.now.ts
import { Record } from "@servicenow/sdk/core";
export const departmentRelRecord = Record({
  $id: Now.ID["department_rel_id"],
  table: "sys_relationship",
  data: {
    advanced: false,
    basic_apply_to: "sn_foo_department",
    basic_query_from: "sn_foo_student",
    name: "Department Allocation Relationship",
    query_with: `(function refineQuery(current, parent) {

	current.addQuery('department', parent.id);

})(current, parent);`,
    simple_reference: false
  }
});

// src/fluent/relationships/related_list_department.now.ts
import { Record } from "@servicenow/sdk/core";
const departmentRelatedListRecord = Record({
  $id: Now.ID["department_related_list_id"],
  table: "sys_ui_related_list",
  data: {
    calculated_name: "Department - Default view",
    name: "sn_foo_department",
    sys_domain: "global",
    sys_domain_path: "/",
    view: "Default view"
  }
});
Record({
  $id: Now.ID["department_related_list_entry_id"],
  table: "sys_ui_related_list_entry",
  data: {
    list_id: `\${departmentRelatedListRecord.$id}`,
    position: "0",
    related_list: `REL:\${departmentRelRecord.$id}`
  }
});
```

### Relationship between out-of-box/store tables

```javascript
import { Record } from "@servicenow/sdk/core";
Record({
  $id: Now.ID["incident_to_problem_relationship_id"],
  table: "sys_relationship",
  data: {
    advanced: false,
    basic_apply_to: "problem",
    basic_query_from: "incident",
    name: "Incidents opened by same user",
    query_with: `(function refineQuery(current, parent) {

	current.addQuery('opened_by', parent.opened_by);
	current.addQuery('active', true);
	current.orderBy('number');

})(current, parent);`,
    simple_reference: false
  }
});
```

### Adding Existing Related list 'Attachments' on custom table

```javascript
import { Record } from "@servicenow/sdk/core";
const schoolRelatedListRecord = Record({
  $id: Now.ID["school_related_list_id"],
  table: "sys_ui_related_list",
  data: {
    calculated_name: "school - Default view",
    name: "sn_foo_school",
    sys_domain: "global",
    sys_domain_path: "/",
    view: "Default view"
  }
});
Record({
  $id: Now.ID["school_related_list_entry_id"],
  table: "sys_ui_related_list_entry",
  data: {
    list_id: `\${schoolRelatedListRecord.$id}`,
    position: "0",
    related_list: "REL:b9edf0ca0a0a0b010035de2d6b579a03"
  }
});
```
