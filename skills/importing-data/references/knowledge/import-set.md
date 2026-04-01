# IMPORT_SET

<!-- Related skill: importing-data -->

## Schema for sys_transform_map

- source_table: The source table from which data is being transformed.
- target_table: The target table into which data is being transformed.
- run_business_rules: Flag indicating whether to run business rules during the transformation.
- run_script: Flag indicating whether to execute a script during the transformation.
- enforce_mandatory_fields: Flag to enforce mandatory fields in the target table.
- copy_empty_fields: Flag to copy empty fields from the source to the target.
- script: The script to execute during the transformation.
- active: Indicates whether the transform map is active.
- order: The order in which the transform map should be executed.
- create_new_record_on_empty_coalesce_fields: Flag indicating whether to create a new record when coalesce fields are empty.
- name: The name of the transform map.
- sys_id: Unique identifier for the record.

## Table Description

The `sys_transform_map` table contains records of transform maps used to define how data is transformed from a source table to a target table during import operations.

## Table Relationships

- Each transform map is associated with a source table and a target table, defining the relationship between the data being transformed and its destination.
- The `sys_transform_entry` table contains mappings for source and target fields, allowing for detailed transformation logic.
- The `sys_transform_script` table contains scripts that can be executed during the transformation process, providing additional processing capabilities.

## Schema for sys_transform_script

- map: Reference to the transform map associated with this script.
- sys_id: Unique identifier for the record.
- when: Specifies when the script should run (e.g., onBefore, onAfter).
- script: The script to execute during the transformation process.
- active: Indicates whether the script is active.
- order: The order in which the script should be executed.

## Schema for sys_transform_entry

- source_table: The source table from which data is being transformed.
- source_field: The field in the source table that is being transformed.
- target_table: The target table into which data is being transformed.
- target_field: The field in the target table that will receive the transformed data.
- choice_action: Action to take when there are multiple choices (e.g., create, update).
- coalesce: Flag indicating whether to coalesce on this field.
- coalesce_empty_fields: Flag to determine if empty fields should be coalesced.
- coalesce_case_sensitive: Flag indicating if coalescing should be case-sensitive.
- date_format: Format for date fields.
- reference_value_field: Field to use for reference values.
- source_script: Script to execute for transforming the source data.
- use_source_script: Flag indicating whether to use the source script.
- sys_id: Unique identifier for the record.
- map: Reference to the transform map associated with this entry.

## TransformMap Plugin API

The TransformMap is a plugin-based API exposed through `@servicenow/sdk/core` that provides a declarative way to define data transformation maps for importing external data into ServiceNow tables.

### Plugin API Import

```typescript
import { ImportSet } from "@servicenow/sdk/core";
```

The TransformMap API creates `sys_transform_map` records along with related `sys_transform_entry` records for field mappings and optional `sys_transform_script` records for advanced transformation logic.

## Properties Reference

### Main TransformMap Properties

| Property                 | Type                 | Required | Default | Valid Values                          | Description                                                               |
| ------------------------ | -------------------- | -------- | ------- | ------------------------------------- | ------------------------------------------------------------------------- |
| `$id`                    | `Now.ID[string]`     | Yes      | -       | -                                     | Unique identifier for the metadata object. Format: `$id: Now.ID['value']` |
| `name`                   | `string`             | Yes      | -       | -                                     | Name of the transform map                                                 |
| `targetTable`            | `string`             | Yes      | -       | Valid table name                      | Target ServiceNow table for data import                                   |
| `sourceTable`            | `string`             | Yes      | -       | Valid import table name               | Source import table containing staging data                               |
| `active`                 | `boolean`            | No       | `false` | `true`, `false`                       | Whether the transform map is active                                       |
| `order`                  | `number`             | No       | `100`   | Any integer                           | Execution order for multiple transform maps                               |
| `runBusinessRules`       | `boolean`            | No       | `false` | `true`, `false`                       | Whether to run business rules during transformation                       |
| `enforceMandatoryFields` | `string`             | No       | `no`    | `no`, `onlyMappedFields`, `allFields` | Level of mandatory field enforcement                                      |
| `copyEmptyFields`        | `boolean`            | No       | `false` | `true`, `false`                       | Whether to copy empty fields from source                                  |
| `createOnEmptyCoalesce`  | `boolean`            | No       | `false` | `true`, `false`                       | Create new record when coalesce fields are empty                          |
| `runScript`              | `boolean`            | No       | `false` | `true`, `false`                       | Whether to execute transform map script                                   |
| `script`                 | `string \| function` | No       | -       | Valid JavaScript or imported function | Transform map script (only when `runScript: true`)                        |
| `fields`                 | `object`             | No       | `{}`    | Object with field mappings            | Field mapping configurations as key-value pairs                           |
| `scripts`                | `ImportSetScript[]`  | No       | `[]`    | Array of scripts                      | Additional transformation scripts                                         |

### Field Mapping Structure

The `fields` property uses an object-based structure where keys are target field names and values can be either:

#### Simple String Mapping

```typescript
fields: {
    'target_field': 'source_field'
}
```

#### Complex Object Mapping

```typescript
fields: {
    'target_field': {
        sourceField: 'source_field',
        coalesce: true,
        // ... other properties
    }
}
```

#### Field Mapping Properties

| Property                | Type                 | Required | Default | Valid Values                          | Description                                                   |
| ----------------------- | -------------------- | -------- | ------- | ------------------------------------- | ------------------------------------------------------------- |
| `sourceField`           | `string`             | Yes      | -       | Valid field name                      | Source field name from import table                           |
| `coalesce`              | `boolean`            | No       | `false` | `true`, `false`                       | Whether this field is used for record matching                |
| `coalesceCaseSensitive` | `boolean`            | No       | `false` | `true`, `false`                       | Case-sensitive coalesce matching (only when `coalesce: true`) |
| `coalesceEmptyFields`   | `boolean`            | No       | `false` | `true`, `false`                       | Coalesce on empty fields (only when `coalesce: true`)         |
| `choiceAction`          | `string`             | No       | `""`    | `"reject"`, `"ignore"`, `"create"`    | Action for choice field values                                |
| `sourceScript`          | `string \| function` | No       | -       | Valid JavaScript or imported function | Field transformation script                                   |
| `useSourceScript`       | `boolean`            | No       | `false` | `true`, `false`                       | Whether to use source script for transformation               |
| `dateFormat`            | `string`             | No       | `""`    | Valid date format                     | Date format for date field transformations                    |
| `referenceValueField`   | `string`             | No       | `""`    | Valid field name                      | Reference value field for reference transformations           |

### ImportSetScript Properties

| Property | Type                 | Required | Default     | Valid Values                                                                                                  | Description                          |
| -------- | -------------------- | -------- | ----------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| `$id`    | `Now.ID[string]`     | Yes      | -           | -                                                                                                             | Unique identifier for the script     |
| `active` | `boolean`            | No       | `true`      | `true`, `false`                                                                                               | Whether the script is active         |
| `order`  | `number`             | No       | `100`       | Any integer                                                                                                   | Execution order for multiple scripts |
| `when`   | `string`             | No       | `"onAfter"` | `"onBefore"`, `"onAfter"`, `"onReject"`, `"onStart"`, `"onForeignInsert"`, `"onComplete"`, `"onChoiceCreate"` | When to execute the script           |
| `script` | `string \| function` | No       | -           | Valid JavaScript or imported function                                                                         | Script code to execute               |

## Examples

### Complete Example: Staging Table + Data Source + Import Set

This example demonstrates the complete pattern for creating a functional import setup. ALL THREE components must be created in order:

```typescript
import "@servicenow/sdk/global";
import { Table, Record, ImportSet } from "@servicenow/sdk/core";

// STEP 1: Create Staging Table Definition (REQUIRED - MUST BE FIRST)
// This creates the actual table structure to hold imported data
export const userStagingTable = Table({
  $id: Now.ID["user-staging-table"],
  name: "u_user_import_staging",
  label: "User Import Staging",
  extends: "sys_import_set_row", // All staging tables extend this
  columns: [
    {
      name: "u_email_address",
      type: "email",
      max_length: 100,
      label: "Email Address"
    },
    {
      name: "u_full_name",
      type: "string",
      max_length: 100,
      label: "Full Name"
    },
    {
      name: "u_username",
      type: "string",
      max_length: 40,
      label: "Username"
    }
  ]
});

// STEP 2: Create Data Source (REQUIRED - MUST BE SECOND)
// The data source defines HOW to get data from external systems
export const userDataSource = Record({
  $id: Now.ID["user-csv-datasource"],
  table: "sys_data_source",
  data: {
    name: "User CSV Data Source",
    type: "File",
    format: "CSV",
    file_retrieval_method: "Attachment",
    csv_delimiter: ",",
    header_row: 1,
    // CRITICAL: This must match the table name from STEP 1
    import_set_table_name: "u_user_import_staging",
    import_set_table_label: "User Import Staging",
    batch_size: 500,
    active: true
  }
});

// STEP 3: Create Import Set (Transform Map) (REQUIRED - MUST BE THIRD)
// The import set defines HOW to transform data from staging to target table
export const userImportSet = ImportSet({
  $id: Now.ID["user-import-transform"],
  name: "User Import Transform",
  targetTable: "sys_user",
  // CRITICAL: This must match import_set_table_name in Data Source
  sourceTable: "u_user_import_staging",
  active: true,
  runBusinessRules: true,
  fields: {
    email: {
      sourceField: "u_email_address",
      coalesce: true
    },
    name: "u_full_name",
    user_name: "u_username"
  }
});
```

### Basic Transform Map

```typescript
import "@servicenow/sdk/global";
import { ImportSet } from "@servicenow/sdk/core";

export const userImportSet = ImportSet({
  $id: Now.ID["user-import-transform"],
  name: "User Import Transform",
  targetTable: "sys_user",
  sourceTable: "u_user_import_staging",
  active: true,
  runBusinessRules: true,
  fields: {
    email: "u_email_address",
    name: "u_full_name"
  }
});
```

### Advanced Example: Transform Map with Field Transformations

```typescript
import "@servicenow/sdk/global";
import { ImportSet } from "@servicenow/sdk/core";

export const userImportWithTransforms = ImportSet({
  $id: Now.ID["user-import-advanced"],
  name: "Advanced User Import with Transformations",
  targetTable: "sys_user",
  sourceTable: "u_user_staging",
  active: true,
  runBusinessRules: true,
  enforceMandatoryFields: "allFields",
  copyEmptyFields: false,
  createOnEmptyCoalesce: true,
  fields: {
    email: {
      sourceField: "u_email_address",
      coalesce: true,
      coalesceCaseSensitive: false
    },
    name: {
      sourceField: "u_full_name",
      useSourceScript: true,
      sourceScript: `answer = (function transformEntry(source) {
                // Convert to proper case
                return source.u_full_name ? source.u_full_name.toLowerCase()
                    .split(' ')
                    .map(word => word.charAt(0).toUpperCase() + word.slice(1))
                    .join(' ') : '';
            })(source);`
    },
    u_department: {
      sourceField: "u_dept_code",
      referenceValueField: "u_dept_id",
      choiceAction: "create"
    },
    u_start_date: {
      sourceField: "u_hire_date",
      dateFormat: "MM/dd/yyyy"
    }
  }
});
```

### Advanced Example: Transform Map with Transform Scripts (Using Server Module)

For complex transform logic, use server modules to get TypeScript validation and better organization.

Server Module: `src/server/ci-transforms.ts`

```typescript
/**
 * Main transform function for CI imports
 * Validates and transforms CI data from staging to target table
 */
export function transformCIRow(
  source: any,
  target: any,
  map: any,
  log: any,
  action: string // Standard parameter: 'insert', 'update', or 'ignore'
): void {
  // Compute isUpdate from action parameter
  const isUpdate = action === "update";

  // Set default values and perform validation
  if (!source.u_asset_tag) {
    log.error("Asset tag is required for computer CI");
    return;
  }

  // Set operational status based on source data
  if (source.u_status === "Active") {
    target.operational_status = "1"; // Operational
  } else if (source.u_status === "Retired") {
    target.operational_status = "7"; // Retired
  }

  // Auto-generate display name (only on insert)
  if (!isUpdate) {
    target.name = source.u_hostname || source.u_asset_tag;
  }
}

/**
 * Validation script to run before transformation
 */
export function validateCIData(
  source: any,
  map: any,
  log: any,
  target: any
): boolean {
  // Validate required fields before processing
  if (!source.u_asset_tag || source.u_asset_tag.trim() === "") {
    log.error("Asset tag cannot be empty");
    return false;
  }

  // Normalize hostname
  if (source.u_hostname) {
    source.u_hostname = source.u_hostname.toLowerCase().trim();
  }

  return true;
}

/**
 * Post-processing script to run after transformation
 */
export function postProcessCI(
  source: any,
  map: any,
  log: any,
  target: any
): void {
  // Post-processing cleanup
  log.info("Successfully processed CI: " + target.name);

  // Update related records if needed
  if (target.sys_id && source.u_location_code) {
    // Additional processing logic here
  }
}
```

Import Set Configuration: `src/records/ci-import.ts`

```typescript
import "@servicenow/sdk/global";
import { ImportSet } from "@servicenow/sdk/core";
import {
  transformCIRow,
  validateCIData,
  postProcessCI
} from "../server/ci-transforms";

export const ciImportWithScripts = ImportSet({
  $id: Now.ID["ci-import-scripts"],
  name: "CI Import with Transform Scripts",
  targetTable: "cmdb_ci_computer",
  sourceTable: "u_computer_import",
  active: true,
  runBusinessRules: false,
  runScript: true,
  script: transformCIRow, // Import from module instead of inline string
  fields: {
    asset_tag: {
      sourceField: "u_asset_tag",
      coalesce: true
    },
    name: "u_hostname",
    serial_number: "u_serial_number"
  },
  scripts: [
    {
      $id: Now.ID["ci-validation-script"],
      when: "onBefore",
      order: 50,
      active: true,
      script: validateCIData // Import from module
    },
    {
      $id: Now.ID["ci-cleanup-script"],
      when: "onAfter",
      order: 200,
      active: true,
      script: postProcessCI // Import from module
    }
  ]
});
```

### Example: Transform Map with TypeScript Functions

```typescript
import "@servicenow/sdk/global";
import { ImportSet } from "@servicenow/sdk/core";
import { validateEmail, formatPhoneNumber } from "./server/data-transforms";
import { enrichUserData } from "./server/user-enrichment";

export const advancedUserImport = ImportSet({
  $id: Now.ID["advanced-user-import"],
  name: "Advanced User Import with TypeScript Functions",
  targetTable: "sys_user",
  sourceTable: "u_user_import_staging",
  active: true,
  runBusinessRules: false,
  enforceMandatoryFields: "onlyMappedFields",
  runScript: true,
  script: enrichUserData, // TypeScript function import
  fields: {
    // Simple string mapping
    user_name: "u_username",
    first_name: "u_fname",
    last_name: "u_lname",

    // Complex mapping with coalesce
    email: {
      sourceField: "u_email_address",
      coalesce: true,
      coalesceCaseSensitive: false,
      useSourceScript: true,
      sourceScript: validateEmail // TypeScript function
    },

    // Phone number formatting
    phone: {
      sourceField: "u_phone_number",
      useSourceScript: true,
      sourceScript: formatPhoneNumber // TypeScript function
    },

    // Department reference with choice creation
    u_department: {
      sourceField: "u_dept_code",
      referenceValueField: "u_dept_id",
      choiceAction: "create"
    }
  },
  scripts: [
    {
      $id: Now.ID["user-validation-script"],
      when: "onBefore",
      order: 10,
      active: true,
      script: `(function runTransformScript(source, map, log, target) {
                // Pre-validation logic
                if (!source.u_username || source.u_username.length < 3) {
                    log.error('Username must be at least 3 characters');
                    return false;
                }
                return true;
            })(source, map, log, target);`
    }
  ]
});
```
