# ATF_FORM

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
- The `atf` object provides access for ATF Form plugin methods `form`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

```ts
atf.form.openNewForm({...})
atf.form.openExistingRecord({...})
atf.form.submitForm({...})
atf.form.setFieldValue({...})
atf.form.fieldValueValidation({...})
atf.form.fieldStateValidation({...})
atf.form.uiActionVisibility({...})
atf.form.clickUIAction({...})
atf.form.clickModalButton({...})
atf.form.declarativeActionVisibility({...})
atf.form.clickDeclarativeAction({...})
```

---

## Properties

### `openNewForm`

| Name     | Type         | Default         | Mandatory | Tools needed | Description                   |
| -------- | ------------ | --------------- | --------- | ------------ | ----------------------------- |
| `table`  | `TableName`  | —               | Yes       | None         | Table to open a new record in |
| `view`   | `string`     | `""`            | No        | None         | Optional form view name       |
| `formUI` | `FormUIType` | `"standard_ui"` | No        | None         | UI flavor to open the form in |

### `openExistingRecord`

| Name               | Type              | Default         | Mandatory | Tools needed | Description                   |
| ------------------ | ----------------- | --------------- | --------- | ------------ | ----------------------------- |
| `table`            | `TableName`       | —               | Yes       | None         | Table to open the record from |
| `recordId`         | `string , Record` | —               | Yes       | run_query    | sys_id or reference to record |
| `view`             | `string`          | `""`            | No        | None         | Optional view name            |
| `formUI`           | `FormUIType`      | `"standard_ui"` | No        | None         | UI flavor                     |
| `selectedTabIndex` | `number`          | `1`             | No        | None         | Tab index to display          |

### `submitForm`

| Name     | Type         | Default         | Mandatory | Tools needed | Description                 |
| -------- | ------------ | --------------- | --------- | ------------ | --------------------------- |
| `assert` | `AssertType` | `""`            | No        | None         | Submission result to assert |
| `formUI` | `FormUIType` | `"standard_ui"` | No        | None         | Form UI flavor              |

### `setFieldValue`

| Name          | Type               | Default         | Mandatory | Tools needed     | Description                               |
| ------------- | ------------------ | --------------- | --------- | ---------------- | ----------------------------------------- |
| `table`       | `TableName`        | —               | Yes       | None             | Table name                                |
| `fieldValues` | `Partial<Data<T>>` | —               | Yes       | get_table_schema | Key-value pairs of field names and values |
| `formUI`      | `FormUIType`       | `"standard_ui"` | No        | None             | UI flavor for form                        |

### `fieldValueValidation`

| Name         | Type         | Default         | Mandatory | Tools needed     | Description                  |
| ------------ | ------------ | --------------- | --------- | ---------------- | ---------------------------- |
| `table`      | `TableName`  | —               | Yes       | None             | Table to validate            |
| `conditions` | `string`     | —               | Yes       | get_table_schema | Encoded query for validation |
| `formUI`     | `FormUIType` | `"standard_ui"` | No        | None             | UI flavor                    |

### `fieldStateValidation`

| Name           | Type                         | Default         | Mandatory | Tools needed     | Description                     |
| -------------- | ---------------------------- | --------------- | --------- | ---------------- | ------------------------------- |
| `table`        | `TableName`                  | —               | Yes       | None             | Table to validate               |
| `visible`      | `(keyof Data<T> , string)[]` | `[]`            | No        | get_table_schema | Fields expected to be visible   |
| `notVisible`   | `(keyof Data<T> , string)[]` | `[]`            | No        | get_table_schema | Fields expected to be hidden    |
| `readOnly`     | `(keyof Data<T> , string)[]` | `[]`            | No        | get_table_schema | Fields expected to be read-only |
| `notReadOnly`  | `(keyof Data<T> , string)[]` | `[]`            | No        | get_table_schema | Fields expected to be editable  |
| `mandatory`    | `(keyof Data<T> , string)[]` | `[]`            | No        | get_table_schema | Fields expected to be mandatory |
| `notMandatory` | `(keyof Data<T> , string)[]` | `[]`            | No        | get_table_schema | Fields expected to be optional  |
| `formUI`       | `FormUIType`                 | `"standard_ui"` | No        | None             | UI flavor                       |

### `uiActionVisibility`

| Name         | Type                                   | Default         | Mandatory | Tools needed | Description                       |
| ------------ | -------------------------------------- | --------------- | --------- | ------------ | --------------------------------- |
| `table`      | `TableName`                            | —               | Yes       | None         | Table on which UI actions exist   |
| `formUI`     | `FormUIType`                           | `"standard_ui"` | No        | None         | UI flavor                         |
| `visible`    | `(string , Record<'sys_ui_action'>)[]` | `[]`            | No        | run_query    | UI Actions expected to be visible |
| `notVisible` | `(string , Record<'sys_ui_action'>)[]` | `[]`            | No        | run_query    | UI Actions expected to be hidden  |

### `clickUIAction`

| Name                | Type                                                   | Default         | Mandatory | Tools needed | Description                       |
| ------------------- | ------------------------------------------------------ | --------------- | --------- | ------------ | --------------------------------- |
| `table`             | `TableName`                                            | —               | Yes       | None         | Table context for the action      |
| `uiAction`          | `string , Record<'sys_ui_action'>`                     | —               | Yes       | run_query    | sys_id or ref of the UI action    |
| `assert`            | `UIActionAssertType`                                   | —               | Yes       | None         | Expected result after the click   |
| `actionType`        | `UIActionType`                                         | `"ui_action"`   | No        | None         | Specifies type of action          |
| `declarativeAction` | `string , Record<'sys_declarative_action_assignment'>` | `""`            | No        | run_query    | Used if actionType is declarative |
| `formUI`            | `FormUIType`                                           | `"standard_ui"` | No        | None         | UI flavor                         |

### `clickModalButton`

| Name            | Type                             | Default         | Mandatory | Tools needed | Description                          |
| --------------- | -------------------------------- | --------------- | --------- | ------------ | ------------------------------------ |
| `uiPage`        | `string , Record<'sys_ui_page'>` | —               | Yes       | run_query    | Page containing modal button         |
| `button`        | `string`                         | —               | Yes       | None         | Button label or identifier           |
| `assert`        | `AssertModalType`                | `""`            | No        | None         | Modal assertion to validate outcome  |
| `assertTimeout` | `number`                         | `5`             | No        | None         | Time to wait before asserting result |
| `formUI`        | `FormUIType`                     | `"standard_ui"` | No        | None         | UI flavor                            |
| `action`        | `ActionType`                     | `"confirm"`     | No        | None         | Modal interaction action             |

### `declarativeActionVisibility`

| Name         | Type                                                       | Default         | Mandatory | Tools needed | Description                                |
| ------------ | ---------------------------------------------------------- | --------------- | --------- | ------------ | ------------------------------------------ |
| `table`      | `TableName`                                                | —               | Yes       | None         | Table containing declarative actions       |
| `visible`    | `(string , Record<'sys_declarative_action_assignment'>)[]` | `[]`            | No        | run_query    | Declarative actions expected to be visible |
| `notVisible` | `(string , Record<'sys_declarative_action_assignment'>)[]` | `[]`            | No        | run_query    | Declarative actions expected to be hidden  |
| `formUI`     | `FormUIType`                                               | `"standard_ui"` | No        | None         | UI flavor                                  |

### `clickDeclarativeAction`

| Name                | Type                                                   | Default         | Mandatory | Tools needed | Description                      |
| ------------------- | ------------------------------------------------------ | --------------- | --------- | ------------ | -------------------------------- |
| `table`             | `TableName`                                            | —               | Yes       | None         | Table on which the action exists |
| `declarativeAction` | `string , Record<'sys_declarative_action_assignment'>` | —               | Yes       | run_query    | Declarative action to click      |
| `assert`            | `AssertDeclarativeActionType`                          | —               | Yes       | None         | Assertion of the action outcome  |
| `formUI`            | `FormUIType`                                           | `"standard_ui"` | No        | None         | UI flavor                        |

---

## Valid Values

| Property         | Valid Values                                                                                   |
| ---------------- | ---------------------------------------------------------------------------------------------- |
| `formUI`         | `"standard_ui"`, `"service_operations_workspace"`, `"asset_workspace"`, `"cmdb_workspace"`     |
| `assert`         | `""`, `"form_submitted_to_server"`, `"form_submission_canceled_in_browser"`, `"page_reloaded"` |
| `action`         | `"confirm"`, `"cancel"`                                                                        |
| `assertTimeout`  | Any positive number                                                                            |
| `actionType`     | `"ui_action"`, `"declarative_action"`                                                          |
| `assert (modal)` | `""`, `"modal_not_closed"`, `"page_not_reloaded"`, `"page_reloaded"`                           |

---

## ATF Specs

### ATF Fluent Spec: form

### Context:

This chunk details APIs used in the ServiceNow Automated Test Framework (ATF) for testing form and modal interactions in a non-Service Portal environment. It includes methods for opening new or existing records, submitting forms, setting and validating field values, clicking UI action buttons, and interacting with modals. Suitable for comprehensive UI testing within the standard ServiceNow user interfaces.

```typescript
// Opens a new form for the selected table and FormUI.
atf.form.openNewForm({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace',
  view: '', // string
}): void;

// Opens an existing record for the selected table and FormUI
// follow this step after a submitForm step to open the record that was just created
atf.form.openExistingRecord({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  recordId: '', // need to use runQuery tool to get the EXACT sys_id of the record in table user input;
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
  view: '', // string
  selectedTabIndex: 0
}): void;

// Submits the current form.
atf.form.submitForm({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  assert: '', // '' | 'form_submitted_to_server' | 'form_submission_canceled_in_browser'
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
}): { table: string; record_id: string };

// Clicks a button within a modal in the specified Form UI
atf.form.clickModalButton({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  uiPage:  '', // need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_ui_page';
  button: '', // button name
  assert: '', // '' | 'page_not_reloaded' | 'modal_not_closed' | 'page_reloaded'
  assertTimeout: 10, // seconds to wait for pass or fail after the button clickable
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
  action: 'confirm', //'confirm' | 'cancel'
}): void;

// Sets field values on the current form after a call to atf.form.openNewForm or atf.form.openExistingRecord
atf.form.setFieldValue({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table: '',
  fieldValues: {}, // ALWAYS use get_table_schema tool to find the mandatory fields so they can be filled out. Pay attention to the field types when setting values. A valid JSON object, e.g. { "field_one": "value1", "field_two": "value2" }.
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
}): void;

// Validates field values on the current form after a call to atf.form.openNewForm or atf.form.openExistingRecord
atf.form.fieldValueValidation({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  conditions: '', // servicenow encoded query. Use the get_table_schema tool to find the field names and types for the table.
  formUI: 'standard_ui' // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
}): void

// Clicks a UI Action button on the current form and asserts the form submission results
atf.form.clickUIAction({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
  actionType: '' // 'ui_action' | 'declarative_action'
  uiAction: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_ui_action';
  declarativeAction: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_declarative_action_assignment';
  assert: 'form_submitted_to_server' // 'form_submitted_to_server' | 'form_submission_canceled_in_browser' | 'page_reloaded_or_redirected'
}): { record_id: string; table: string }
```

### ATF Fluent Spec: form.action

### Context:

This chunk describes APIs used in the Automated Test Framework (ATF) for interacting with UI Actions on forms. These APIs allow testers to validate the visibility of UI Action buttons and to click UI Actions with assertions regarding the submission results. This is applicable within the context of testing forms in various UI environments in ServiceNow, such as 'standard_ui', 'service_operations_workspace', 'asset_workspace', and 'cmdb_workspace'. The chunk is relevant for scenarios where managing UI interactions through ATF is essential.

```typescript
// Validates whether a UI Action button is visible or not on the current form.
atf.form.uiActionVisibility({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  formUI: '', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
  visible: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_ui_action';
  notVisible: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_ui_action';
}): void;

// Clicks a UI Action button on the current form and asserts the form submission results
atf.form.clickUIAction({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  uiAction:'', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_ui_action';
  assert: 'form_submitted_to_server' // 'form_submitted_to_server' | 'form_submission_canceled_in_browser' | 'page_reloaded_or_redirected'
  actionType: '' // 'ui_action' | 'declarative_action'
  declarativeAction:'', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_declarative_action_assignment';
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
}): { record_id: string; table: string }
```

### ATF Fluent Spec: form.actionda

### Context:

This chunk is part of the ServiceNow Automated Test Framework (ATF) API documentation specifically focused on APIs for interacting with declarative actions on forms. These APIs are essential for validating the visibility of declarative actions, performing clicks on these actions, and asserting the results of form submissions. They are used in scenarios where forms within different user interfaces, such as standard UI, service operations workspace, asset workspace, or cmdb workspace, are tested for correct behavior and outcomes associated with declarative actions.

```typescript
// Validates whether a declarative action is visible on the current form
atf.form.declarativeActionVisibility({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  formUI: '', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
  visible: [''], // Array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_declarative_action_assignment';
  notVisible: [''], // Array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_declarative_action_assignment';
}): void;

// Clicks a declarative action on the current form.
atf.form.clickDeclarativeAction({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  declarativeAction:'', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_declarative_action_assignment';
  assert: 'form_submitted_to_server' //'form_submitted_to_server' | 'form_submission_canceled_in_browser' | 'page_reloaded_or_redirected'
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
}): { record_id: string; table: string }
```

### ATF Fluent Spec: form.field

### Context:

The chunk focuses on ServiceNow Automated Test Framework (ATF) APIs designed for manipulating and validating field values on forms specifically within the Service Portal. These APIs allow for setting and validating field values, as well as assessing field states like visibility and mandatory status

```typescript
// Validates field values on the current form after a call to atf.form.openNewForm or atf.form.openExistingRecord
atf.form.fieldValueValidation({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  conditions: '', // servicenow encoded query
  formUI: 'standard_ui' // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
}): void;

// Sets field values on the current form after a call to atf.form.openNewForm or atf.form.openExistingRecord
atf.form.setFieldValue({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table: '',
  fieldValues: {}, // ALWAYS use get_table_schema tool to find the mandatory fields so they can be filled out. A valid JSON object, e.g. { "field_one": "value1", "field_two": "value2" }.
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
}): void;

// Validates states of the desired fields on the current form after a call to atf.form.openNewForm or atf.form.openExistingRecord
atf.form.fieldStateValidation({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  visible: [''], // array of field names
  notVisible: [''], // array of field names
  readOnly: [''], // array of field names
  notReadOnly: [''], // array of field names
  mandatory: [''], // array of field names
  notMandatory: [''], // array of field names
  formUI: 'standard_ui', // 'standard_ui' | 'service_operations_workspace' | 'asset_workspace' | 'cmdb_workspace'
}): void;
```

---

### Example

```javascript
import "@servicenow/sdk/global";
import { Test } from "@servicenow/sdk/core";

Test(
  {
    $id: Now.ID["validate_incident_reference"],
    name: "Validate Incident Reference Field",
    description:
      "Opens a new incident form, sets short description field to Email server is down, submits it, then opens it in the cmdb workspace",
    active: true,
    failOnServerError: true
  },
  atf => {
    // Step 1: Open a new incident form in standard UI
    atf.form.openNewForm({
      $id: Now.ID["open_new_incident"],
      table: "incident",
      formUI: "standard_ui"
    });

    // Step 2: Set caller field to Email server is down
    atf.form.setFieldValue({
      $id: Now.ID["set_caller_field"],
      table: "incident",
      fieldValues: {
        // ALWAYS use get_table_schema tool to find the mandatory fields so they can be filled out. Pay attention to the field types.
        short_description: "Email server is down"
      },
      formUI: "standard_ui"
    });

    // Step 3: Submit the form and store the result with the new record ID
    const submissionResult = atf.form.submitForm({
      $id: Now.ID["submit_incident_form"],
      assert: "form_submitted_to_server",
      formUI: "standard_ui"
    });

    // Step 4: Validate that the field value was set correctly using a server query
    atf.server.recordValidation({
      $id: Now.ID["validate_caller_field"],
      table: "incident",
      recordId: submissionResult.record_id, // Use the record_id from the form submission
      fieldValues: "short_description=Email server is down",
      assert: "record_validated"
    });

    // Step 5: Open the created incident in the cmdb workspace
    atf.form.openExistingRecord({
      $id: Now.ID["open_in_cmdb_workspace"],
      table: "incident",
      recordId: submissionResult.record_id, // Use the record_id from the form submission
      formUI: "cmdb_workspace"
    });

    // Step 6: Verify the caller field is correctly displayed in CMDB workspace
    atf.form.fieldValueValidation({
      $id: Now.ID["validate_caller_in_cmdb"],
      table: "incident",
      conditions: "short_description=Email server is down",
      formUI: "cmdb_workspace"
    });
  }
);
```

## Notes

- All `sys_id` references must be fetched using the `runQuery` tool for tables like:
  - `sys_ui_action`
  - `sys_ui_page`
  - `sys_declarative_action_assignment`
- Fields like `table` accept string names of valid SN tables.
- Form UI types allow testing across legacy and workspace environments.
- `Record` syntax should be used when embedding references to sys_id values.
- For `setFieldValue`, ALWAYS use get_table_schema tool to find the mandatory fields, so they can be filled out. Pay attention to the field types when setting values.
