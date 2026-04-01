# ATF_FORM_SP

<!-- Related skill: implementing-tests -->

## Test Constructor

Each ATF test starts with the `Test()` function, which accepts test metadata and a configuration function that defines the test steps.

```ts
import { Test } from '@servicenow/sdk/core'
import '@servicenow/sdk-core/global'

Test({
  $id: Now.ID['test_id'],
  name: 'test name',
  description?: 'optional description',
  active?: true,
  failOnServerError?: true
}, (atf) => {
  atf.<plugin>.<method>({
    $id: Now.ID['step_id'],
    ...params
  });

  // More steps...
});
```

- The `$id` must be globally unique for both the test and each step.
- The `atf` object provides access for ATF Form in Service Portal(SP) plugin methods `form_SP`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

This metadata type is defined using the `form_SP` property of the ATF Fluent API.

```ts
atf.form_SP.openServicePortalPage({...})
atf.form_SP.openNewForm({...})
atf.form_SP.setFieldValue({...})
atf.form_SP.fieldValueValidation({...})
atf.form_SP.clickUIAction({...})
atf.form_SP.submitForm({...})
atf.form_SP.uiActionVisibilityValidation({...})
atf.form_SP.fieldStateValidation({...})
```

---

## Properties

### `openServicePortalPage`

| Name              | Type                           | Mandatory | Tools needed | Description                                        |
| ----------------- | ------------------------------ | --------- | ------------ | -------------------------------------------------- |
| `$id`             | `string`                       | Yes       | None         | Unique identifier                                  |
| `portal`          | `string , Record<'sp_portal'>` | Yes       | run_query    | sys_id of the portal (use runQuery on `sp_portal`) |
| `page`            | `string , Record<'sp_page'>`   | Yes       | run_query    | sys_id of the page (use runQuery on `sp_page`)     |
| `queryParameters` | `object`                       | No        | None         | Optional URL parameters                            |

### `openNewForm`

| Name              | Type                           | Mandatory | Tools needed | Description                                      |
| ----------------- | ------------------------------ | --------- | ------------ | ------------------------------------------------ |
| `$id`             | `string`                       | Yes       | None         | Unique identifier                                |
| `table`           | `string`                       | Yes       | None         | Table name                                       |
| `paramID`         | `string , Record<TableName>`   | Yes       | run_query    | sys_id of the record (use runQuery on table T)   |
| `portal`          | `string , Record<'sp_portal'>` | No        | run_query    | Optional portal ID (use runQuery on `sp_portal`) |
| `page`            | `string , Record<'sp_page'>`   | No        | run_query    | Optional page ID (use runQuery on `sp_page`)     |
| `view`            | `string`                       | No        | None         | View name                                        |
| `queryParameters` | `object`                       | No        | None         | Optional query parameters                        |

### `setFieldValue`

| Name          | Type     | Mandatory | Tools needed     | Description                      |
| ------------- | -------- | --------- | ---------------- | -------------------------------- |
| `$id`         | `string` | Yes       | None             | Unique identifier                |
| `table`       | `string` | Yes       | None             | Table name                       |
| `fieldValues` | `object` | Yes       | get_table_schema | JSON with field name-value pairs |

### `fieldValueValidation`

| Name         | Type     | Mandatory | Tools needed     | Description                                  |
| ------------ | -------- | --------- | ---------------- | -------------------------------------------- |
| `$id`        | `string` | Yes       | None             | Unique identifier                            |
| `table`      | `string` | Yes       | None             | Table name                                   |
| `conditions` | `string` | Yes       | get_table_schema | ServiceNow encoded query for field condition |

### `clickUIAction`

| Name       | Type                               | Mandatory | Tools needed | Description                                        |
| ---------- | ---------------------------------- | --------- | ------------ | -------------------------------------------------- |
| `$id`      | `string`                           | Yes       | None         | Unique identifier                                  |
| `table`    | `string`                           | Yes       | None         | Table name                                         |
| `uiAction` | `string , Record<'sys_ui_action'>` | Yes       | run_query    | UI Action sys_id (use runQuery on `sys_ui_action`) |
| `assert`   | `string`                           | Yes       | None         | Assertion type                                     |

### `submitForm`

| Name     | Type     | Mandatory | Tools needed | Description                       |
| -------- | -------- | --------- | ------------ | --------------------------------- |
| `$id`    | `string` | Yes       | None         | Unique identifier                 |
| `assert` | `string` | Yes       | None         | Assertion result after submission |

### `uiActionVisibilityValidation`

| Name         | Type                                   | Mandatory | Tools needed | Description                             |
| ------------ | -------------------------------------- | --------- | ------------ | --------------------------------------- |
| `$id`        | `string`                               | Yes       | None         | Unique identifier                       |
| `table`      | `string`                               | Yes       | None         | Table name                              |
| `visible`    | `string[] , Record<'sys_ui_action'>[]` | No        | run_query    | Optional list of visible UI actions     |
| `notVisible` | `string[] , Record<'sys_ui_action'>[]` | No        | run_query    | Optional list of non-visible UI actions |

### `fieldStateValidation`

| Name           | Type       | Mandatory | Tools needed     | Description                         |
| -------------- | ---------- | --------- | ---------------- | ----------------------------------- |
| `$id`          | `string`   | Yes       | None             | Unique identifier                   |
| `table`        | `string`   | Yes       | None             | Table name                          |
| `visible`      | `string[]` | No        | get_table_schema | Fields that should be visible       |
| `notVisible`   | `string[]` | No        | get_table_schema | Fields that should not be visible   |
| `mandatory`    | `string[]` | No        | get_table_schema | Fields that should be mandatory     |
| `notMandatory` | `string[]` | No        | get_table_schema | Fields that should not be mandatory |
| `readOnly`     | `string[]` | No        | get_table_schema | Fields that should be read-only     |
| `notReadonly`  | `string[]` | No        | get_table_schema | Fields that should not be read-only |

---

## ATF Specs

### Context

The chunk describes APIs for interacting with forms in the ServiceNow Service Portal using the Automated Test Framework (ATF). It includes steps to open a Service Portal page or form, set and validate form field values, click UI Action buttons, and submit forms.

```typescript
// Opens a Service Portal page.
atf.form_SP.openServicePortalPage({
  $id: Now.ID[""],
  portal: "", // need to use runQuery tool
  page: "", // need to use runQuery tool
  queryParameters: {}
});

// Opens a new form for the selected table in the Service Portal.
atf.form_SP.openNewForm({
  $id: Now.ID[""],
  table: "",
  paramID: "", // need to use runQuery tool
  portal: "", // optional
  page: "", // optional
  view: "",
  queryParameters: {}
});

// Sets field values
atf.form_SP.setFieldValue({
  $id: Now.ID[""],
  table: "",
  fieldValues: {} // ALWAYS first use get_table_schema tool to find the field names and types for the table
});

// Validates field values
atf.form_SP.fieldValueValidation({
  $id: Now.ID[""],
  table: "",
  conditions: "" // ALWAYS first use get_table_schema tool to find the field names and types for the table
});

// Clicks a UI Action
atf.form_SP.clickUIAction({
  $id: Now.ID[""],
  table: "",
  uiAction: "", // need to use runQuery tool
  assert: "form_submitted_to_server"
});

// Submit form
atf.form_SP.submitForm({
  $id: Now.ID[""],
  assert: "form_submitted_to_server"
});

// Validate UI Action visibility
atf.form_SP.uiActionVisibilityValidation({
  $id: Now.ID[""],
  table: "",
  visible: [""],
  notVisible: [""]
});

// Validate field state
atf.form_SP.fieldStateValidation({
  $id: Now.ID[""],
  table: "",
  visible: [""],
  notVisible: [""],
  mandatory: [""],
  notMandatory: [""],
  readOnly: [""],
  notReadonly: [""]
});
```

---

### Example

```javascript
// Fluent ATF Test
import { Test } from "@servicenow/sdk/core";
import "@servicenow/sdk-core/global";
Test(
  {
    $id: Now.ID["test_1_abc123"], // fill in a valid ServiceNow sys_id
    name: "Incident Form SP Test",
    description:
      "Test to open an incident form in Service Portal, set short description, submit, and validate declarative action visibility.",
    active: true,
    failOnServerError: true
  },
  atf => {
    atf.form_SP.openNewForm({
      $id: Now.ID["step_1_def456"], // globally unique ID
      table: "incident", // table name hint
      paramID: "incident_form", // assuming paramID for the form
      portal: "26f2fffb77322300454792718a1061e5", // portal sys_id
      page: "", // page sys_id
      view: ""
    });
    atf.form_SP.setFieldValue({
      $id: Now.ID["step_2_ghi789"], // globally unique ID
      table: "incident", // table name hint
      fieldValues: { short_description: "test short description in form SP" } // ALWAYS first use get_table_schema tool to find the field names and types for the table
    });
    const outputOfSubmit = atf.form_SP.submitForm({
      $id: Now.ID["step_3_jkl012"], // globally unique ID
      assert: "form_submitted_to_server"
    });
    atf.form_SP.openNewForm({
      $id: Now.ID["step_4_mno345"], // globally unique ID
      table: "incident", // table name hint
      paramID: outputOfSubmit.record_id, // use the declared variable to fill in value
      portal: "26f2fffb77322300454792718a1061e5", // portal sys_id
      page: "84af292247132100ba13a5554ee4909e", // page sys_id
      view: ""
    });
    atf.form_SP.uiActionVisibilityValidation({
      $id: Now.ID["step_5_pqr678"], // globally unique ID
      table: "incident", // table name hint
      visible: ["0152e2f453922010c5e2ddeeff7b121c"], // declarative action sys_id
      notVisible: []
    });
  }
);
```

## Notes
