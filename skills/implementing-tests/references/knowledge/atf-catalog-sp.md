# ATF_CATALOG_SP

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
- The `atf` object provides access for ATF Catalog in Service Portal(SP) plugin methods `catalog_SP`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

```ts
atf.catalog_SP.openRecordProducer({...})
atf.catalog_SP.submitRecordProducer({...})
atf.catalog_SP.setCatalogItemQuantity({...})
atf.catalog_SP.openCatalogItem({...})
atf.catalog_SP.validatePriceAndRecurringPrice({...})
atf.catalog_SP.addItemtoShoppingCart({...})
atf.catalog_SP.orderCatalogItem({...})
atf.catalog_SP.openOrderGuide({...})
atf.catalog_SP.navigatewithinOrderGuide({...})
atf.catalog_SP.validateOrderGuideItem({...})
atf.catalog_SP.reviewOrderGuideSummary({...})
atf.catalog_SP.saveCurrentRowOfMultiRowVariableSet({...})
atf.catalog_SP.addRowToMultiRowVariableSet({...})
atf.catalog_SP.setVariableValue({...})
atf.catalog_SP.validateVariableValue({...})
atf.catalog_SP.variableStateValidation({...})
```

---

## Properties

### `openRecordProducer`

| Name              | Type                                     | Default                              | Mandatory | Needs runQuery Tool? | Description                   |
| ----------------- | ---------------------------------------- | ------------------------------------ | --------- | -------------------- | ----------------------------- |
| `recordProducer`  | `string ,Record<'sc_cat_item_producer'>` | —                                    | Yes       | Yes                  | sys_id of the Record Producer |
| `portal`          | `string ,Record<'sp_portal'>`            | `"81b75d3147032100ba13a5554ee4902b"` | No        | Yes                  | Portal sys_id                 |
| `page`            | `string ,Record<'sp_page'>`              | `"9f12251147132100ba13a5554ee490f4"` | No        | Yes                  | Page sys_id                   |
| `queryParameters` | `QueryParam`                             | `{}`                                 | No        | No                   | Additional query parameters   |

### `submitRecordProducer`

| Name     | Type                                                                 | Default                      | Mandatory | Needs runQuery Tool? | Description                     |
| -------- | -------------------------------------------------------------------- | ---------------------------- | --------- | -------------------- | ------------------------------- |
| `assert` | `'form_submitted_to_server' ,'form_submission_cancelled_in_browser'` | `'form_submitted_to_server'` | No        | No                   | Assertion after form submission |

### `setCatalogItemQuantity`

| Name       | Type             | Default | Mandatory | Needs runQuery Tool? | Description           |
| ---------- | ---------------- | ------- | --------- | -------------------- | --------------------- |
| `quantity` | `string ,number` | —       | Yes       | No                   | Quantity value to set |

### `openCatalogItem`

| Name              | Type                            | Default                              | Mandatory | Needs runQuery Tool? | Description                 |
| ----------------- | ------------------------------- | ------------------------------------ | --------- | -------------------- | --------------------------- |
| `catalogItem`     | `string ,Record<'sc_cat_item'>` | —                                    | Yes       | Yes                  | sys_id of catalog item      |
| `portal`          | `string ,Record<'sp_portal'>`   | `"81b75d3147032100ba13a5554ee4902b"` | No        | Yes                  | Portal sys_id               |
| `page`            | `string ,Record<'sp_page'>`     | `"sc_cat_item"`                      | No        | Yes                  | Page sys_id                 |
| `queryParameters` | `QueryParam`                    | `{}`                                 | No        | No                   | Additional query parameters |

### `validatePriceAndRecurringPrice`

| Name             | Type             | Default | Mandatory | Needs runQuery Tool? | Description                    |
| ---------------- | ---------------- | ------- | --------- | -------------------- | ------------------------------ |
| `price`          | `string ,number` | —       | Yes       | No                   | Expected price value           |
| `recurringPrice` | `string ,number` | —       | Yes       | No                   | Expected recurring price value |
| `frequency`      | `FrequencyType`  | —       | Yes       | No                   | Frequency for recurring price  |

### `addItemtoShoppingCart`

| Name     | Type                                                                 | Default                      | Mandatory | Needs runQuery Tool? | Description                    |
| -------- | -------------------------------------------------------------------- | ---------------------------- | --------- | -------------------- | ------------------------------ |
| `assert` | `'form_submitted_to_server' ,'form_submission_cancelled_in_browser'` | `'form_submitted_to_server'` | No        | No                   | Assertion after adding to cart |

### `orderCatalogItem`

| Name     | Type                                                                 | Default                      | Mandatory | Needs runQuery Tool? | Description                   |
| -------- | -------------------------------------------------------------------- | ---------------------------- | --------- | -------------------- | ----------------------------- |
| `assert` | `'form_submitted_to_server' ,'form_submission_cancelled_in_browser'` | `'form_submitted_to_server'` | No        | No                   | Assertion after ordering item |

### `openOrderGuide`

| Name              | Type                                  | Default                              | Mandatory | Needs runQuery Tool? | Description                 |
| ----------------- | ------------------------------------- | ------------------------------------ | --------- | -------------------- | --------------------------- |
| `orderGuide`      | `string ,Record<'sc_cat_item_guide'>` | —                                    | Yes       | Yes                  | sys_id of Order Guide       |
| `portal`          | `string ,Record<'sp_portal'>`         | `"81b75d3147032100ba13a5554ee4902b"` | No        | Yes                  | Portal sys_id               |
| `page`            | `string ,Record<'sp_page'>`           | `"sc_cat_item_guide"`                | No        | Yes                  | Page sys_id                 |
| `queryParameters` | `QueryParam`                          | `{}`                                 | No        | No                   | Additional query parameters |

### `navigatewithinOrderGuide`

| Name        | Type                                     | Default              | Mandatory | Needs runQuery Tool? | Description                      |
| ----------- | ---------------------------------------- | -------------------- | --------- | -------------------- | -------------------------------- |
| `guideStep` | `GuideStepType`                          | `1`                  | No        | No                   | Step number to navigate to       |
| `assert`    | `'navigate_success' ,'navigate_failure'` | `'navigate_success'` | No        | No                   | Assertion for navigation outcome |

### `validateOrderGuideItem`

| Name    | Type                                    | Default | Mandatory | Needs runQuery Tool? | Description                 |
| ------- | --------------------------------------- | ------- | --------- | -------------------- | --------------------------- |
| `items` | `Array<string ,Record<'sc_cat_item'> >` | —       | Yes       | Yes                  | Items included in the guide |

### `reviewOrderGuideSummary`

| Name    | Type                                    | Default | Mandatory | Needs runQuery Tool? | Description                |
| ------- | --------------------------------------- | ------- | --------- | -------------------- | -------------------------- |
| `items` | `Array<string ,Record<'sc_cat_item'> >` | —       | Yes       | Yes                  | Items in the summary stage |
| `price` | `string ,number`                        | —       | Yes       | No                   | Price of summary items     |

### `saveCurrentRowOfMultiRowVariableSet`

| Name     | Type                                                                 | Default                      | Mandatory | Needs runQuery Tool? | Description                |
| -------- | -------------------------------------------------------------------- | ---------------------------- | --------- | -------------------- | -------------------------- |
| `assert` | `'form_submitted_to_server' ,'form_submission_cancelled_in_browser'` | `'form_submitted_to_server'` | No        | No                   | Assertion after saving row |

### `addRowToMultiRowVariableSet`

| Name          | Type                                    | Default | Mandatory | Needs runQuery Tool? | Description                   |
| ------------- | --------------------------------------- | ------- | --------- | -------------------- | ----------------------------- |
| `catalogItem` | `string ,Record<'sc_cat_item'>`         | —       | Yes       | Yes                  | Catalog item sys_id           |
| `variableSet` | `string ,Record<'item_option_new_set'>` | —       | Yes       | Yes                  | Multi-row variable set sys_id |

### `setVariableValue`

| Name             | Type                                                    | Default | Mandatory | Needs runQuery Tool? | Description                                                                                                                                                                                                                                  |
| ---------------- | ------------------------------------------------------- | ------- | --------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `catalogItem`    | `string ,Record<'sc_cat_item' ,'sc_cat_item_producer'>` | —       | Yes       | Yes                  | Catalog item or Record Producer                                                                                                                                                                                                              |
| `variableSet`    | `string ,Record<'item_option_new_set'>`                 | —       | Yes       | Yes                  | Variable set sys_id                                                                                                                                                                                                                          |
| `variableValues` | `string`                                                | —       | Yes       | Yes                  | must be a string in the format: IO:<sys_id>=<value>, joined with ^ and ending in ^EQ. Use the runQuery agent tool to retrieve each sys_id from item_option_new by querying on name=<label>. Example: IO:<sys_id1>=true^IO:<sys_id2>=500MB^EQ |

### `validateVariableValue`

| Name             | Type                                    | Default | Mandatory | Needs runQuery Tool? | Description                                                                                                                                                                                                                                  |
| ---------------- | --------------------------------------- | ------- | --------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `catalogItem`    | `string ,Record<'sc_cat_item'>`         | —       | Yes       | Yes                  | Catalog item sys_id                                                                                                                                                                                                                          |
| `variableSet`    | `string ,Record<'item_option_new_set'>` | —       | Yes       | Yes                  | Variable set sys_id                                                                                                                                                                                                                          |
| `variableValues` | `string`                                | —       | Yes       | Yes                  | must be a string in the format: IO:<sys_id>=<value>, joined with ^ and ending in ^EQ. Use the runQuery agent tool to retrieve each sys_id from item_option_new by querying on name=<label>. Example: IO:<sys_id1>=true^IO:<sys_id2>=500MB^EQ |

### `variableStateValidation`

| Name           | Type                                    | Default | Mandatory | Needs runQuery Tool? | Description                            |
| -------------- | --------------------------------------- | ------- | --------- | -------------------- | -------------------------------------- |
| `catalogItem`  | `string ,Record<'sc_cat_item'>`         | —       | Yes       | Yes                  | Catalog item sys_id                    |
| `variableSet`  | `string ,Record<'item_option_new_set'>` | —       | Yes       | Yes                  | Variable set sys_id                    |
| `visible`      | `string[]`                              | `[]`    | No        | No                   | Variable names expected to be visible  |
| `notVisible`   | `string[]`                              | `[]`    | No        | No                   | Variables expected to be hidden        |
| `readonly`     | `string[]`                              | `[]`    | No        | No                   | Variables expected to be read-only     |
| `notReadonly`  | `string[]`                              | `[]`    | No        | No                   | Variables expected to be editable      |
| `mandatory`    | `string[]`                              | `[]`    | No        | No                   | Variables expected to be mandatory     |
| `notMandatory` | `string[]`                              | `[]`    | No        | No                   | Variables not expected to be mandatory |

---

## Valid Values

| Property                            | Valid Values                                                           |
| ----------------------------------- | ---------------------------------------------------------------------- |
| `assert`                            | `'form_submitted_to_server'`, `'form_submission_cancelled_in_browser'` |
| `frequency`                         | `'weekly'`, `'monthly'`, `'quarterly'`, `'yearly'`                     |
| `guideStep`                         | `'1'`, `'2'`, `'3'`                                                    |
| `assert (navigatewithinOrderGuide)` | `'navigate_success'`, `'navigate_failure'`                             |

---

## ATF Specs

### Context:

This chunk describes APIs used in ServiceNow's Application Testing Framework (ATF) to interact with Record Producers within the Service Portal. It allows opening a Record Producer by sys_id and submitting it, optionally asserting the result of the submission. These steps are crucial for automating tests involving Record Producers, enabling comprehensive validation of their behavior and outcomes.

```typescript
// Opens a Record Producer in the Service Portal.
atf.catalog_SP.openRecordProducer({ // recordProducer is mandatory; other props are optional
  $id: Now.ID[''], // string | guid, mandatory
  recordProducer: '', // need to use runQuery tool to get the EXACT sys_id of the record in table Record Producer
  portal: '' // Optional. need to use runQuery tool to get the EXACT sys_id of the record in table 'sp_portal';.
  page: '' // Optional. need to use runQuery tool to get the EXACT sys_id of the record in table 'sp_page';.
  queryParameters: {} // Optional. { [name: string]: string | number | boolean | Record | TableName }.
}): void;

// Submit currently opened Record Producer
atf.catalog_SP.submitRecordProducer({ // all props are optional
  $id: Now.ID[''], // string | guid, mandatory
  assert: 'form_submitted_to_server' // Optional. 'form_submitted_to_server' | 'form_submission_cancelled_in_browser'
}): {
  record_id: string, // This is the record_id created after submit form
  table: '' //TableName
};

// Sets the quantity value for the current Catalog Item.
atf.catalog_SP.setCatalogItemQuantity({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  quantity: '' // Quantity as a number or string.
}): void;
```

### Context:

This chunk details steps for interacting with catalog items in the ServiceNow Service Portal using Automated Test Framework (ATF) APIs, including opening a catalog item, validating its price, adding it to a shopping cart, and ordering it. These actions enable automated testing of catalog item transactions within a portal environment.

```typescript
// Opens a Catalog Item in the Service Portal.
atf.catalog_SP.openCatalogItem({ // catalogItem is mandatory; other props are optional
  $id: Now.ID[''], // string | guid, mandatory
  catalogItem: '', // need to use runQuery tool to get the EXACT sys_id of the record in table sc_cat_item
  portal: '' // Optional. need to use runQuery tool to get the EXACT sys_id of the record in table 'sp_portal';.
  page: '' // Optional. need to use runQuery tool to get the EXACT sys_id of the record in table 'sp_page';.
  queryParameters: {} // Optional. { [name: string]: string | number | boolean | Record | TableName }.
}): void;

// Validates the price and recurring price of the current Catalog Item.
atf.catalog_SP.validatePriceAndRecurringPrice({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  price: '', // Price as a number or string.
  recurringPrice: '', // Recurring price as a number or string.
  frequency: '' // 'weekly' | 'quarterly' | 'monthly' | 'yearly' |
}): void;

// Adds an item to the Shopping Cart.
atf.catalog_SP.addItemtoShoppingCart({ // all props are optional
  $id: Now.ID[''], // string | guid, mandatory
  assert: '' // Optional. 'form_submitted_to_server' | 'form_submission_cancelled_in_browser'
}): { cart_item_id: string };

// Orders the currently opened Catalog Item.
atf.catalog_SP.orderCatalogItem({ // all props are optional
  $id: Now.ID[''], // string | guid, mandatory
  assert: 'form_submitted_to_server' // Optional. 'form_submitted_to_server' | 'form_submission_cancelled_in_browser'
}): {
  record_id: string, // This is the record_id created after submit form
  table: '' //TableName
};
```

### Context:

This chunk focuses on the ServiceNow Automated Test Framework (ATF) steps related to Order Guides in the Service Portal. It provides APIs to open an Order Guide, navigate through its steps, validate included items, and review the summary, aiding in comprehensive testing of order guide-related functionality within the ServiceNow platform.

```typescript
// Opens an Order Guide in the Service Portal.
atf.catalog_SP.openOrderGuide({ // orderGuide is mandatory; other props are optional
  $id: Now.ID[''], // string | guid, mandatory
  orderGuide: '', // need to use runQuery tool to get the EXACT sys_id of the record in table sc_cat_item_guide
  portal: '' // Optional. need to use runQuery tool to get the EXACT sys_id of the record in table 'sp_portal';.
  page: '' // Optional. need to use runQuery tool to get the EXACT sys_id of the record in table 'sp_page';.
  queryParameters: {} // Optional. { [name: string]: string | number | boolean | Record | TableName }.
}): { table: TableName; record_id: string };

// Navigates within an Order Guide.
atf.catalog_SP.navigatewithinOrderGuide({ // all props are optional
  $id: Now.ID[''], // string | guid, mandatory
  guideStep: '', // Optional. '1' | '2' | '3'
  assert: '' // Optional. 'navigate_success' | 'navigate_failure'
}): void;

//Validate items included in the Order Guide
atf.catalog_SP.validateOrderGuideItem({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  items: '', //(need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_cat_item';
}): void;

//Review Order Guide Summary in Service Portal
atf.catalog_SP.reviewOrderGuideSummary({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  items: '', // (need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_cat_item';
  price: '', // string | number
}): void;
```

### Context:

This chunk focuses on Automated Test Framework (ATF) steps for managing and validating multi-row variable sets and catalog item variables within the Service Portal.

```typescript
// Saves the current row of a multi-row variable set in the Service Portal.
  atf.catalog_SP.saveCurrentRowOfMultiRowVariableSet({ // all props are optional
  $id: Now.ID[''], // string | guid, mandatory
  assert: '' // Optional. 'form_submitted_to_server' | 'form_submission_cancelled_in_browser'
}): void;

// Adds a row to a multi-row variable set in the Service Portal.
atf.catalog_SP.addRowToMultiRowVariableSet({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  catalogItem: '', // need to use runQuery tool to get the EXACT sys_id of the record in table sc_cat_item
  variableSet: '' // need to use runQuery tool to get the EXACT sys_id of the record in table item_option_new_set
}): void;

// Sets variable values on the current Catalog Item or Record Producer page.
atf.catalog_SP.setVariableValue({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  catalogItem: '', // need to use runQuery tool to get the EXACT sys_id of the record in table sc_cat_item
  variableSet: '', // need to use runQuery tool to get the EXACT sys_id of the record in table item_option_new_set
  variableValues: '' // variableValues must be a string in the format: IO:<sys_id>=<value>, joined with ^ and ending in ^EQ. Use the runQuery agent tool to retrieve each sys_id from 'item_option_new' by querying on 'name=<label>'. Example: "IO:<sys_id1>=true^IO:<sys_id2>=500MB^EQ"
}): void;

// Validates variable values on the current Catalog Item or Record Producer page.
atf.catalog_SP.validateVariableValue({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  catalogItem: '', // need to use runQuery tool to get the EXACT sys_id of the record in table sc_cat_item
  variableSet: '', // need to use runQuery tool to get the EXACT sys_id of the record in table item_option_new_set
  variableValues: '' // variableValues must be a string in the format: IO:<sys_id>=<value>, joined with ^ and ending in ^EQ. Use the runQuery agent tool to retrieve each sys_id from 'item_option_new' by querying on 'name=<label>'. Example: "IO:<sys_id1>=true^IO:<sys_id2>=500MB^EQ"
}): void;

// Validates the states of variables on the current Catalog Item or Record Producer page.
atf.catalog_SP.variableStateValidation({ // catalogItem and variableSet are mandatory; other props are optional
  $id: Now.ID[''], // string | guid, mandatory
  catalogItem: '', // need to use runQuery tool to get the EXACT sys_id of the record in table sc_cat_item
  variableSet: '', // need to use runQuery tool to get the EXACT sys_id of the record in table item_option_new_set
  visible: [''] // Optional. Array of variable names.
  notVisible: [''] // Optional. Array of variable names.
  readOnly: [''] // Optional. Array of variable names.
  notReadOnly: [''] // Optional. Array of variable names.
  mandatory: [''] // Optional. Array of variable names.
  notMandatory: [''] // Optional. Array of variable names.
}): void;
```

---

### Example

```javascript
import "@servicenow/sdk/global";
import { Test } from "@servicenow/sdk/core";

// ATF Test that opens a Camtasia catalog item in the Software category in Customer Service Portal,
// asserts the item is visible, then opens the item in service portal form,
// sets the field 'state' to 'submitted' in service portal, logs a message, and validates
// that the report 'software_catalog_report' is not visible

Test(
  {
    $id: Now.ID["open_camtasia_catalog_item"],
    name: "Open Camtasia Catalog Item Test",
    description:
      "Tests opening and interacting with the Camtasia catalog item in the Service Portal",
    active: true,
    failOnServerError: true
  },
  atf => {
    // Step 1: Open the catalog item Camtasia in the Software category in Service Portal
    atf.catalog_SP.openCatalogItem({
      $id: Now.ID["open_catalog_item_step"],
      catalogItem: "6d53a69147dec110f53d37d2846d4314", // Camtasia catalog item sys_id
      portal: "81b75d3147032100ba13a5554ee4902b" // Service Portal sys_id
    });
    // Step 2: Set the state field to 'submitted' in the service portal form
    atf.form_SP.setFieldValue({
      $id: Now.ID["set_state_field"],
      table: "sc_cat_item",
      fieldValues: {
        state: "submitted"
      }
    });
    // Step 3: Log a message that the catalog item has been opened
    atf.server.log({
      $id: Now.ID["log_catalog_item_opened"],
      log: "Camtasia catalog item in the Service Portal opened"
    });
  }
);
```

---

## Notes

- All `sys_id` values must be fetched using the `runQuery` tool for the relevant tables:
  - `sc_cat_item`, `sc_cat_item_producer`, `sc_cat_item_guide`, `sp_page`, `sp_portal`, `item_option_new_set`.
- Any field allowing both `string` and `Record` types supports literal `sys_id` or `{ table: string, sys_id: string }` style.
- Optional fields with default values (e.g., `portal`, `page`, `assert`) can be omitted when defaults are sufficient.
- Variables set or validated must already exist on the open item/producer; incorrect context leads to failure.
