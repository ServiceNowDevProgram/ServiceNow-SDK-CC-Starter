# ATF_CATALOG

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
- The `atf` object provides access for ATF Catalog plugin methods `catalog`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

This metadata type is defined using the `catalog` property of the ATF Fluent API.

```ts
atf.catalog.openCatalogItem({...})
atf.catalog.addItemToShoppingCart({...})
atf.catalog.setCatalogItemQuantity({...})
atf.catalog.orderCatalogItem({...})
atf.catalog.validatePriceAndRecurringPrice({...})
atf.catalog.validateVariableValue({...})
atf.catalog.variableStateValidation({...})
atf.catalog.setVariableValue({...})
atf.catalog.openRecordProducer({...})
atf.catalog.submitRecordProducer({...})
```

---

## Properties

### `openCatalogItem`

| Name          | Type                             | Default | Mandatory | Needs runQuery Tool? | Description                          |
| ------------- | -------------------------------- | ------- | --------- | -------------------- | ------------------------------------ |
| `$id`         | `string`                         | —       | Yes       | No                   | Unique identifier for this test step |
| `catalogItem` | `string , Record<'sc_cat_item'>` | —       | Yes       | Yes                  | sys_id of the catalog item to open   |

### `addItemToShoppingCart`

| Name     | Type                                                                       | Default | Mandatory | Needs runQuery Tool? | Description                          |
| -------- | -------------------------------------------------------------------------- | ------- | --------- | -------------------- | ------------------------------------ |
| `$id`    | `string`                                                                   | —       | Yes       | No                   | Unique identifier for this test step |
| `assert` | `'' , 'form_submission_cancelled_in_browser' , 'form_submitted_to_server'` | `''`    | Yes       | No                   | Assertion on form submission result  |

### `setCatalogItemQuantity`

| Name       | Type              | Default | Mandatory | Needs runQuery Tool? | Description                          |
| ---------- | ----------------- | ------- | --------- | -------------------- | ------------------------------------ |
| `$id`      | `string`          | —       | Yes       | No                   | Unique identifier for this test step |
| `quantity` | `number , string` | —       | Yes       | No                   | Quantity of item to be set           |

### `orderCatalogItem`

| Name     | Type                                                                  | Default | Mandatory | Needs runQuery Tool? | Description                                 |
| -------- | --------------------------------------------------------------------- | ------- | --------- | -------------------- | ------------------------------------------- |
| `$id`    | `string`                                                              | —       | Yes       | No                   | Unique identifier for this test step        |
| `assert` | `'form_submitted_to_server' , 'form_submission_cancelled_in_browser'` | —       | Yes       | No                   | Assertion on catalog item ordering behavior |

### `validatePriceAndRecurringPrice`

| Name             | Type                                                                                                   | Default | Mandatory | Needs runQuery Tool? | Description                          |
| ---------------- | ------------------------------------------------------------------------------------------------------ | ------- | --------- | -------------------- | ------------------------------------ |
| `$id`            | `string`                                                                                               | —       | Yes       | No                   | Unique identifier for this test step |
| `price`          | `number , string`                                                                                      | —       | Yes       | No                   | Expected price value                 |
| `recurringPrice` | `number , string`                                                                                      | —       | Yes       | No                   | Expected recurring price value       |
| `frequency`      | `'' , 'weekly' , 'quarterly' , 'weekly2' , 'semiannual' , 'monthly' , 'yearly' , 'daily' , 'monthly2'` | `''`    | Yes       | No                   | Recurrence frequency to validate     |

### `validateVariableValue`

| Name                | Type                             | Default | Mandatory | Needs runQuery Tool? | Description                                                                                                                                                                                                                                  |
| ------------------- | -------------------------------- | ------- | --------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$id`               | `string`                         | —       | Yes       | No                   | Unique identifier for this test step                                                                                                                                                                                                         |
| `catalogItem`       | `string , Record<'sc_cat_item'>` | —       | Yes       | Yes                  | sys_id of catalog item to validate variables on                                                                                                                                                                                              |
| `catalogConditions` | `string`                         | —       | Yes       | Yes                  | must be a string in the format: IO:<sys_id>=<value>, joined with ^ and ending in ^EQ. Use the runQuery agent tool to retrieve each sys_id from item_option_new by querying on name=<label>. Example: IO:<sys_id1>=true^IO:<sys_id2>=500MB^EQ |

### `variableStateValidation`

| Name           | Type                                        | Default | Mandatory | Needs runQuery Tool? | Description                                    |
| -------------- | ------------------------------------------- | ------- | --------- | -------------------- | ---------------------------------------------- |
| `$id`          | `string`                                    | —       | Yes       | No                   | Unique identifier for this test step           |
| `catalogItem`  | `string , Record<'sc_cat_item'>`            | —       | Yes       | Yes                  | Catalog item whose variables are being checked |
| `visible`      | `Array<string , Record<'item_option_new'>>` | `[]`    | Yes       | Yes                  | Variable sys_ids expected to be visible        |
| `notVisible`   | `Array<string , Record<'item_option_new'>>` | `[]`    | Yes       | Yes                  | Variable sys_ids expected to be not visible    |
| `readOnly`     | `Array<string , Record<'item_option_new'>>` | `[]`    | Yes       | Yes                  | Variable sys_ids expected to be read-only      |
| `notReadOnly`  | `Array<string , Record<'item_option_new'>>` | `[]`    | Yes       | Yes                  | Variable sys_ids expected to be editable       |
| `mandatory`    | `Array<string , Record<'item_option_new'>>` | `[]`    | Yes       | Yes                  | Variable sys_ids expected to be mandatory      |
| `notMandatory` | `Array<string , Record<'item_option_new'>>` | `[]`    | Yes       | Yes                  | Variable sys_ids expected to not be mandatory  |

### `setVariableValue`

| Name             | Type                             | Default | Mandatory | Needs runQuery Tool? | Description                                                                                                                                                                                                                                  |
| ---------------- | -------------------------------- | ------- | --------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `$id`            | `string`                         | —       | Yes       | No                   | Unique identifier for this test step                                                                                                                                                                                                         |
| `catalogItem`    | `string , Record<'sc_cat_item'>` | —       | Yes       | Yes                  | Catalog item whose variables are being set                                                                                                                                                                                                   |
| `variableValues` | `string`                         | —       | Yes       | Yes                  | must be a string in the format: IO:<sys_id>=<value>, joined with ^ and ending in ^EQ. Use the runQuery agent tool to retrieve each sys_id from item_option_new by querying on name=<label>. Example: IO:<sys_id1>=true^IO:<sys_id2>=500MB^EQ |

### `openRecordProducer`

| Name          | Type                                      | Default | Mandatory | Needs runQuery Tool? | Description                           |
| ------------- | ----------------------------------------- | ------- | --------- | -------------------- | ------------------------------------- |
| `$id`         | `string`                                  | —       | Yes       | No                   | Unique identifier for this test step  |
| `catalogItem` | `string , Record<'sc_cat_item_producer'>` | —       | Yes       | Yes                  | sys_id of the record producer to open |

### `submitRecordProducer`

| Name     | Type                                                                       | Default | Mandatory | Needs runQuery Tool? | Description                                           |
| -------- | -------------------------------------------------------------------------- | ------- | --------- | -------------------- | ----------------------------------------------------- |
| `$id`    | `string`                                                                   | —       | Yes       | No                   | Unique identifier for this test step                  |
| `assert` | `'' , 'form_submitted_to_server' , 'form_submission_cancelled_in_browser'` | `''`    | Yes       | No                   | Assertion on submission result of the record producer |

---

## Valid Values

| Property    | Valid Values                                                                                   |
| ----------- | ---------------------------------------------------------------------------------------------- |
| `assert`    | `''`, `form_submitted_to_server`, `form_submission_cancelled_in_browser`                       |
| `frequency` | `''`, `weekly`, `quarterly`, `weekly2`, `semiannual`, `monthly`, `yearly`, `daily`, `monthly2` |

---

## ATF Specs

### ATF Fluent Spec: catalog.actions

### Context

This chunk focuses on catalog item management within ServiceNow using the Automated Test Framework (ATF). It provides APIs for interacting with catalog items, including opening catalog items by their system ID, adding them to the shopping cart, setting their quantity, and ordering or closing them. These steps are designed to facilitate automated testing of catalog-related transactions and workflows in ServiceNow, enhancing the efficiency and accuracy of test scenarios involving catalog items.

```typescript
// open a catalog item by sys_id
atf.catalog.openCatalogItem({
    $id: Now.ID[''], // string | guid, mandatory
    catalogItem: '' //need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_cat_item';
}): void;

// Add item to Shopping Cart after a call to atf.catalog.openCatalogItem
atf.catalog.addItemToShoppingCart({
    $id: Now.ID[''], // string | guid, mandatory
    assert: '' // '' | 'form_submission_cancelled_in_browser' | 'form_submitted_to_server';
}): {
    cart_item_id: '' // sys_id of the item added to the cart
};

// Sets quantity value on the current catalog item after a call to atf.catalog.openCatalogItem
atf.catalog.setCatalogItemQuantity({
    $id: Now.ID[''], // string | guid, mandatory
    quantity: '' // number | string
}): void;

// Order and close the currently opened catalog item after a call to atf.catalog.openCatalogItem, no other API calls are allowed after this
atf.catalog.orderCatalogItem({
    $id: Now.ID[''], // string | guid, mandatory
    assert: '', // 'form_submitted_to_server' | 'form_submission_cancelled_in_browser'
}): {
    request_id: '', // sys_id
    cart: ''
};

```

### ATF Fluent Spec: catalog.validation

### Context

This chunk provides ATF APIs for validation tasks related to catalog items and their variables within ServiceNow. The APIs support testing functionalities like validating the price and recurring prices of catalog items, ensuring that variable values comply with expected conditions, and checking the visible, read-only, and mandatory states of variables. These steps are crucial for comprehensive validation when catalog items or related forms are used within ServiceNow's application testing framework.

```typescript
// Step to validate price and recurring price of a Catalog Item after a call to atf.catalog.openCatalogItem. Can not be used with Record Producers
atf.catalog.validatePriceAndRecurringPrice({
    $id: Now.ID[''], // string | guid, mandatory
    price: '', // number | string
    recurringPrice: '', // number | string
    frequency: '', // '' | 'weekly' | 'quarterly' | 'weekly2' | 'semiannual' | 'monthly' | 'yearly' | 'daily' | 'monthly2';
}): void;

// Validates variable values on the Catalog Item, Record Producer pages or a page containing a variable editor. can be used only in-between atf.catalog.openCatalogItem step and atf.catalog.orderCatalogItem step
atf.catalog.validateVariableValue({
    $id: Now.ID[''], // string | guid, mandatory
    catalogItem: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_cat_item';
    catalogConditions: '' //  must be a string in the format: IO:<sys_id>=<value>, joined with ^ and ending in ^EQ. Use the runQuery agent tool to retrieve each sys_id from item_option_new by querying on name=<label>. Example: IO:<sys_id1>=true^IO:<sys_id2>=500MB^EQ
}): void;

// Validates states of the desired variables.
atf.catalog.variableStateValidation({ // each of the following props are mandatory
    $id: Now.ID[''], // string | guid, mandatory
    catalogItem: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_cat_item';
    visible: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'item_option_new';
    notVisible: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'item_option_new';
    readOnly: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'item_option_new';
    notReadOnly: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'item_option_new';
    mandatory: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'item_option_new';
    notMandatory: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'item_option_new';
}): void;
```

### ATF Fluent Spec: catalog.vars

### Context

This chunk focuses on APIs in the Automated Test Framework (ATF) for interacting with Catalog Items and Record Producers within ServiceNow. These APIs facilitate setting variable values on Catalog Items or Record Producer pages, opening Record Producers, and submitting them as part of automated tests. They are intended for use when automating ServiceNow's catalog-related functionalities.

```typescript
// Sets variable values on the current Catalog Item or Record Producer page or a form containing variable editor
atf.catalog.setVariableValue({
    $id: Now.ID[''], // string | guid, mandatory
    catalogItem: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_cat_item';
    variableValues: '' //  must be a string in the format: IO:<sys_id>=<value>, joined with ^ and ending in ^EQ. Use the runQuery agent tool to retrieve each sys_id from item_option_new by querying on name=<label>. Example: IO:<sys_id1>=true^IO:<sys_id2>=500MB^EQ
}): void;

// Opens a Record Producer for a catalog item
atf.catalog.openRecordProducer({
    $id: Now.ID[''], // string | guid, mandatory
    catalogItem: '' //need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_cat_item_producer';
}): void;

// Submit currently opened Record Producer after a call to atf.catalog.openRecordProducer, no other API calls are allowed after this
atf.catalog.submitRecordProducer({
    $id: Now.ID[''], // string | guid, mandatory
    assert: '' // '' | 'form_submitted_to_server' | 'form_submission_cancelled_in_browser'
}): {
    record_id: '' // sys_id
};
```

## Notes

- Use the `runQuery` tool to retrieve exact sys_ids from these tables:
  - `sc_cat_item`
  - `sc_cat_item_producer`
  - `item_option_new`
- Many inputs accept either a `string` sys_id or a `Record<...>` object — prefer string format for consistency in ATF steps.
- For variables and conditions, use the `IO:sys_id=value` convention joined by `^` for multiple entries.
- All Fluent steps are designed to be used in specific sequences. For example:
  - Do not call `orderCatalogItem` before `openCatalogItem`.
  - `submitRecordProducer` must only follow `openRecordProducer`.
