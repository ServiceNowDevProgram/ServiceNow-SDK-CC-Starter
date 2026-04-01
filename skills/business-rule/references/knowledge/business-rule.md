# BUSINESS_RULE

## The BusinessRule Object

The BusinessRule object is the core component for creating business rules with Fluent API.

### Basic Structure

```javascript
import { BusinessRule } from "@servicenow/sdk/core";

const myBusinessRule = BusinessRule({
  $id: Now.ID[0],
  name: "My Business Rule",
  table: "x_snc_table",
  when: "before",
  action: ["insert", "update"],
  script: myFunction
  // other properties as needed
});
```

### Required Properties

- **$id**: A unique identifier for the metadata object. Format: `Now.ID[<value>]`
- **name**: A name for the business rule
- **table**: The name of the table on which the business rule runs

### Common Properties

- **order**: Sequence number for execution order (default: 100)
- **script**: The script to run (function, function expression, or inline script)
- **when**: When the business rule executes (before, after, async, display)
- **action**: Array of record operations to apply the rule to (insert, update, delete, query)
- **active**: Whether the rule is active (true/false)
- **addMessage**: Whether to display a message when the rule runs
- **message**: The message to display
- **abortAction**: Whether to abort the current database transaction
- **roleConditions**: Roles required for the rule to run
- **condition**: JavaScript conditional for when the rule should run
- **filterCondition**: Filter query defining when the rule should run
- **setFieldValue**: Values to set fields in the table
- **description**: Description of the business rule's purpose

## Script Types

The `script` property can use several formats:

1. **Function Export**:

```javascript
import { FunctionExport } from "../server/scripts.js";
script: FunctionExport;
```

2. **Function Expression**:

```javascript
import { FunctionExpression } from "../server/scripts.js";
script: FunctionExpression;
```

3. **Default Export Function**:

```javascript
import DefaultExportFunction from "../server/scripts.js";
script: DefaultExportFunction;
```

4. **Inline Script** (string literal or template literal):

```javascript
script: `gs.info('info')`;
```

## Execution Timing (when)

- **before**: Runs before the record operation (default)
- **after**: Runs after the record operation
- **async**: Runs asynchronously after the operation completes
- **display**: Runs when the record is displayed

## Actions

Business rules can be configured to run on specific record operations:

- **insert**: When records are created
- **update**: When records are modified
- **delete**: When records are deleted
- **query**: When the table is queried

## Conditional Execution

You can control when a business rule runs using:

1. **condition**: JavaScript conditional statement

```javascript
condition: "current.state == 'pending'";
```

2. **filterCondition**: Filter query using ServiceNow's filter syntax

```javascript
filterCondition: `sys_updated_onSTARTSWITHb^sys_updated_bySTARTSWITHm^EQ`;
```

## Role-Based Execution

Restrict execution to specific roles:

```javascript
roleConditions: [admin];
```

## Example Business Rules

### Basic Business Rule with Function Export

```javascript
import { BusinessRule } from "@servicenow/sdk/core";
import { FunctionExport } from "../server/scripts.js";

const BR1 = BusinessRule({
  name: "exportedFunction",
  table: "x_snc_table",
  when: "after",
  action: ["update", "delete", "insert"],
  script: FunctionExport,
  order: 100,
  active: true,
  addMessage: false,
  message: "<p>message</p>",
  abortAction: false,
  $id: Now.ID[0]
});
```

### Business Rule with Function Expression

```javascript
const BR2 = BusinessRule({
  name: "businessrule2",
  table: "x_snc_table",
  script: FunctionExpression,
  when: "after",
  action: ["update"],
  $id: Now.ID[1]
});
```

### Business Rule with Filter Condition

```javascript
const BR3 = BusinessRule({
  name: "businessrule3",
  table: "x_snc_table",
  script: DefaultExportFunction,
  when: "after",
  action: ["update"],
  filterCondition: `sys_updated_onSTARTSWITHb^sys_updated_bySTARTSWITHm^EQ`,
  $id: Now.ID[2]
});
```

### Business Rule with Inline Script and Role Condition

```javascript
const BR4 = BusinessRule({
  name: "templateBR",
  action: ["insert"],
  when: "after",
  table: "x_snc_table",
  roleConditions: [admin],
  order: 100,
  active: true,
  addMessage: true,
  message: "<p>message</p>",
  script: `gs.info('info')`,
  abortAction: false,
  $id: Now.ID[3]
});
```

## Metadata for Specific Installation Scenarios

You can control when business rules are installed using the `$meta` property:

```javascript
$meta: {
  installMethod: "demo"; // or 'first install'
}
```

- **demo**: Installed when "Load demo data" is selected
- **first install**: Installed only the first time an application is installed
