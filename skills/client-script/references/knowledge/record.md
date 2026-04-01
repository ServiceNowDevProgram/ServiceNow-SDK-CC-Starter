# RECORD

<!-- Related skill: record -->

## Record Object

Add data to any table with a record.

## Properties

| Name    | Type             | Description                                                                                                                                                                                                            |
| ------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$id`   | String or Number | Required. A unique ID for the metadata object provided in the following format, where `<value>` is a string or number: `$id: Now.ID[<value>]`. When you build the application, this ID is hashed into a unique sys_ID. |
| `table` | String           | Required. The name of the table to which the record belongs.                                                                                                                                                           |
| `data`  | Object           | Fields and their values in the table. To use text content from another file with a property, you can refer to a file in the application using the syntax `Now.include('path/to/file')`.                                |
| `$meta` | Object           | Metadata for the application metadata. With the `installMethod` property, you can map the application metadata to an output directory that loads only in specific circumstances.                                       |

## Examples

### Example 1: Adding records to a custom table

```typescript
import {
  Table,
  BooleanColumn,
  StringColumn,
  DateColumn,
  DateTimeColumn,
  IntegerColumn,
  Record
} from "@servicenow/sdk/core";

// Create a custom table
export const x_snc_example_to_do = Table({
  name: "x_snc_example_to_do",
  label: "My To Do Table",
  schema: {
    task: StringColumn({ label: "Task", maxLength: 120 }),
    deadline: DateColumn({ label: "Deadline" }),
    state: StringColumn({
      label: "State",
      choices: {
        ready: { label: "Ready", sequence: 0 },
        completed: { label: "Completed", sequence: 1 },
        in_progress: { label: "In Progress", sequence: 2 }
      }
    }),
    completed_at: DateTimeColumn({ label: "Completed" }),
    priority: IntegerColumn({ label: "Priority", maxLength: 40 }),
    active: BooleanColumn({ label: "Active" })
  }
});

// Add a record to the custom table
export const todo1 = Record({
  table: "x_snc_example_to_do",
  $id: Now.ID["todo-1"],
  data: {
    task: "Complete the documentation",
    deadline: "2025-07-30", // Default format is 'yyyy-mm-dd' for DateColumn
    state: "completed",
    completed_at: "2025-07-18 20:56:22", // Default format is 'yyyy-mm-dd HH:mm:ss' for DateTimeColumn
    priority: 1,
    active: true
  }
});
```

### Example 2: Adding a Menu Category

```typescript
import { Record } from "@servicenow/sdk/core";
export const appCategory = Record({
  table: "sys_app_category",
  $id: Now.ID[9],
  data: {
    name: "example",
    style: Now.include("./css-file.css")
  }
});
```

### Example 3: Adding a Script Include

```typescript
import { Record } from "@servicenow/sdk/core";
export const SampleClass = Record({
  $id: Now.ID["SampleClass"],
  table: "sys_script_include",
  data: {
    name: "SampleClass",
    apiName: "x_sample.SampleClass",
    access: "public",
    caller_access: "caller_tracking",
    protection_policy: "read-only",
    script: Now.include("./SampleClass.server.js")
  }
});
```

### Example 4: Adding an Incident

```typescript
import { Record } from "@servicenow/sdk/core";
export const incident1 = Record({
  $id: Now.ID["incident-1"],
  table: "incident",
  data: {
    active: true,
    approval: "not requested",
    description: "Unable to send or receive emails.",
    incident_state: 1,
    short_description: "Email server is down.",
    subcategory: "email",
    caller_id: "77ad8176731313005754660c4cf6a7de"
  }
});
```

### Example 5: Adding a Server

```typescript
import { Record } from "@servicenow/sdk/core";
export const ciserver1 = Record({
  $id: Now.ID["cmdb-ci-server-1"],
  table: "cmdb_ci_server",
  data: {
    asset_tag: "P1000199",
    attested: false,
    can_print: false,
    company: "e7c1f3d53790200044e0bfc8bcbe5deb",
    cost: 2160,
    cost_cc: "USD",
    cpu_speed: 633,
    cpu_type: "GenuineIntel",
    disk_space: 100,
    manufacturer: "b7e7d7d8c0a8016900a5d7f291acce5c",
    name: "DatabaseServer1",
    os: "Linux Red Hat",
    short_description: "DB Server",
    subcategory: "Computer"
  }
});
```
