# ATF_SERVER

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
- The `atf` object provides access for ATF Server plugin methods `server`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

This metadata type is defined using the `server` property of the ATF Fluent API.

```ts
atf.server.impersonate({...})
atf.server.createUser({...})
atf.server.log({...})
atf.server.recordQuery({...})
atf.server.recordInsert({...})
atf.server.recordValidation({...})
atf.server.recordUpdate({...})
atf.server.recordDelete({...})
atf.server.searchForCatalogItem({...})
atf.server.checkoutShoppingCart({...})
atf.server.replayRequestItem({...})
```

---

## Properties

### `impersonate`

| Name   | Type                          | Default | Mandatory | Tools needed | Description                               |
| ------ | ----------------------------- | ------- | --------- | ------------ | ----------------------------------------- |
| `user` | `string , Record<'sys_user'>` | —       | Yes       | run_query    | User to impersonate during test execution |

### `createUser`

| Name          | Type                                       | Default | Mandatory | Tools needed     | Description                                                     |
| ------------- | ------------------------------------------ | ------- | --------- | ---------------- | --------------------------------------------------------------- |
| `fieldValues` | `Partial<Data<'sys_user'>>`                | `{}`    | Yes       | get_table_schema | Additional user fields to set (JSON snake_case key/value pairs) |
| `firstName`   | `string`                                   | `""`    | Yes       | None             | First name of user                                              |
| `lastName`    | `string`                                   | `""`    | Yes       | None             | Last name of user                                               |
| `groups`      | `Array<string , Record<'sys_user_group'>>` | `[]`    | Yes       | run_query        | List of user group sys_ids                                      |
| `roles`       | `Array<string , Role>`                     | `[]`    | Yes       | run_query        | List of role sys_ids                                            |
| `impersonate` | `boolean`                                  | `true`  | Yes       | None             | Whether to impersonate user after creation                      |

### `log`

| Name  | Type     | Default | Mandatory | Tools needed | Description                    |
| ----- | -------- | ------- | --------- | ------------ | ------------------------------ |
| `log` | `string` | —       | Yes       | None         | Message to include in test log |

### `recordQuery`

| Name              | Type              | Default                 | Mandatory | Tools needed     | Description                           |
| ----------------- | ----------------- | ----------------------- | --------- | ---------------- | ------------------------------------- |
| `table`           | `TableName`       | —                       | Yes       | None             | Name of the table to query            |
| `fieldValues`     | `string`          | —                       | Yes       | get_table_schema | Encoded query condition               |
| `assert`          | `AssertQueryType` | `'records_match_query'` | No        | None             | Assertion mode (match or not match)   |
| `enforceSecurity` | `boolean`         | `false`                 | No        | None             | Enforce security constraints on query |

### `recordInsert`

| Name              | Type               | Default                          | Mandatory | Tools needed     | Description                                           |
| ----------------- | ------------------ | -------------------------------- | --------- | ---------------- | ----------------------------------------------------- |
| `table`           | `TableName`        | —                                | Yes       | None             | Target table for insert                               |
| `fieldValues`     | `Partial<Data<T>>` | —                                | Yes       | get_table_schema | Field-value map (snake_case keys, double-quoted JSON) |
| `assert`          | `AssertInsert`     | `'record_successfully_inserted'` | No        | None             | Insert success assertion                              |
| `enforceSecurity` | `boolean`          | `true`                           | No        | None             | Enforce security constraints on insert                |

### `recordValidation`

| Name              | Type                         | Default              | Mandatory | Tools needed     | Description                                |
| ----------------- | ---------------------------- | -------------------- | --------- | ---------------- | ------------------------------------------ |
| `table`           | `TableName`                  | —                    | Yes       | None             | Table to validate against                  |
| `recordId`        | `string`                     | —                    | Yes       | run_query        | sys_id of record to validate               |
| `fieldValues`     | `string`                     | —                    | Yes       | get_table_schema | Encoded query condition                    |
| `assert`          | `AssertRecordValidationType` | `'record_validated'` | No        | None             | Assertion type for validation result       |
| `enforceSecurity` | `boolean`                    | `true`               | No        | None             | Enforce security constraints on validation |

### `recordUpdate`

| Name              | Type               | Default                         | Mandatory | Tools needed     | Description                                           |
| ----------------- | ------------------ | ------------------------------- | --------- | ---------------- | ----------------------------------------------------- |
| `table`           | `TableName`        | —                               | Yes       | None             | Table of record                                       |
| `recordId`        | `string`           | —                               | Yes       | run_query        | sys_id of record to update                            |
| `fieldValues`     | `Partial<Data<T>>` | —                               | Yes       | get_table_schema | Field-value map for update (snake_case + JSON format) |
| `assert`          | `AssertUpdateType` | `'record_successfully_updated'` | No        | None             | Assertion result after update                         |
| `enforceSecurity` | `boolean`          | `true`                          | No        | None             | Whether to enforce security during update             |

### `recordDelete`

| Name              | Type                     | Default                         | Mandatory | Tools needed | Description                                 |
| ----------------- | ------------------------ | ------------------------------- | --------- | ------------ | ------------------------------------------- |
| `table`           | `TableName`              | —                               | Yes       | None         | Table of record                             |
| `recordId`        | `string`                 | —                               | Yes       | run_query    | sys_id of record to delete                  |
| `assert`          | `RecordDeleteAssertType` | `'record_successfully_deleted'` | No        | None         | Assertion result after delete               |
| `enforceSecurity` | `boolean`                | `true`                          | No        | None         | Whether to enforce security during deletion |

### `searchForCatalogItem`

| Name             | Type                             | Default                 | Mandatory | Tools needed | Description                             |
| ---------------- | -------------------------------- | ----------------------- | --------- | ------------ | --------------------------------------- |
| `searchTerm`     | `string`                         | —                       | Yes       | None         | Keyword to search for                   |
| `catalog`        | `string , Record<'sc_catalog'>`  | —                       | Yes       | run_query    | Catalog sys_id                          |
| `category`       | `string , Record<'sc_category'>` | —                       | Yes       | run_query    | Category sys_id                         |
| `assertItem`     | `string , Record<'sc_cat_item'>` | —                       | Yes       | run_query    | Catalog item sys_id to verify           |
| `assert`         | `AssertCatalogItem`              | `'assert_item_present'` | No        | None         | Expected search result                  |
| `searchInPortal` | `boolean`                        | `false`                 | No        | None         | Whether to search inside Service Portal |

### `checkoutShoppingCart`

| Name                  | Type                          | Default                  | Mandatory | Tools needed | Description                            |
| --------------------- | ----------------------------- | ------------------------ | --------- | ------------ | -------------------------------------- |
| `requestedFor`        | `string , Record<'sys_user'>` | —                        | Yes       | run_query    | User receiving the request             |
| `deliveryAddress`     | `string`                      | —                        | Yes       | None         | Address where item should be delivered |
| `specialInstructions` | `string`                      | —                        | Yes       | None         | Notes or extra instructions            |
| `assert`              | `AssertCartType`              | `'checkout_successfull'` | No        | None         | Expected checkout result               |

### `replayRequestItem`

| Name           | Type                             | Default | Mandatory | Tools needed | Description                 |
| -------------- | -------------------------------- | ------- | --------- | ------------ | --------------------------- |
| `request_item` | `string , Record<'sc_req_item'>` | —       | Yes       | run_query    | Request item to be replayed |

---

## Valid Values

| Property          | Valid Values                                                                                                                                                                                                                                                                                                                                                                 |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `assert`          | `'record_successfully_inserted'`, `'record_not_inserted'`, `'record_successfully_updated'`, `'record_not_updated'`, `'record_successfully_deleted'`, `'record_not_deleted'`, `'assert_item_present'`, `'assert_item_not_present'`, `'checkout_successfull'`, `'empty_cart'`, `'record_validated'`, `'record_not_found'`, `'records_match_query'`, `'no_records_match_query'` |
| `enforceSecurity` | `true`, `false`                                                                                                                                                                                                                                                                                                                                                              |
| `impersonate`     | `true`, `false`                                                                                                                                                                                                                                                                                                                                                              |
| `searchInPortal`  | `true`, `false`                                                                                                                                                                                                                                                                                                                                                              |

---

## ATF Specs

### ATF Fluent Spec: server

### Context

This chunk provides APIs for user session management and logging in ServiceNow's ATF. It includes operations like impersonating a user, creating a user with specific roles and groups, logging messages for tests, performing record queries, and inserting records into tables. These functionalities are crucial for simulating different user interactions and ensuring databases reflect expected test results.

```typescript
// Impersonates the specified user in the current session for the duration of the test or until another user is impersonated.
atf.server.impersonate({
  $id: Now.ID[''], // string | guid, mandatory
  user: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_user';
}): {
  user: '' // sys_id of the user
};

// Create a user with specified roles and groups. Optionally impersonate the user in the current session for the duration of the test or until another user is impersonated.
atf.server.createUser({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  fieldValues: {}, // a valid JSON object, e.g. { "user_name": "john.doe", "home_phone": "18581234567" }. ALWAYS use the get_table_schema tool to find the field names and types for the `sys_user` table. Do NOT include first_name or last_name
  groups: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_user_group';
  roles: [''], // array of sys_id, need to use runQuery tool to get the EXACT sys_id of the record in table sys_user_role;
  impersonate: false, // boolean
  firstName: '', // string
  lastName: '', // string
}): { user: string; };

// Logs a message that can contain a variable or other information pertaining to the test. This message will be stored as a step result upon test completion.
atf.server.log({
    $id: Now.ID[''], // string | guid, mandatory
    log: ''; // string
}): void;

// Perform a database query to verify if a record matching the conditions set in this step are met
atf.server.recordQuery({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  fieldValues: '', // string, servicenow encoded query. ALWAYS use the get_table_schema tool to find the field names and types for the table.
  enforceSecurity: false, // boolean;
  assert: 'records_match_query', // 'records_match_query' | 'no_records_match_query';
}): { table: string;
    first_record: string; // sys_id of the first record
};

// Inserts a record into a table. Specify the field values to set on the new record, outputs the table and the sys_id of the new record.
atf.server.recordInsert({ // all props are mandatory
    $id: Now.ID[''], // string | guid, mandatory
    table: '',
    fieldValues: {}, // a valid JSON object, e.g. { "field_one": "value1", "field_two": "value2" }. ALWAYS use get_table_schema tool to find the mandatory fields so they can be filled out. Pay attention to the field types.
    assert: '', // 'record_successfully_inserted' | 'record_not_inserted';
    enforceSecurity: false, // boolean;
 }): { table: string;
    record_id: string; // sys_id of the new record
};
```

### ATF Fluent Spec: server.item

### Context

The chunk is part of the ServiceNow Automated Test Framework (ATF) API documentation focusing on catalog and shopping cart functionalities such as searching for catalog items or record producers, adding them to a shopping cart, checking out the cart to create requests, and replaying request items. These APIs are used to automate the testing of catalog-related processes within ServiceNow applications.

```typescript
// Perform search for a Catalog Item or Record Producer in the specified Catalog and Category
atf.server.searchForCatalogItem({ // all props are mandatory
    $id: Now.ID[''], // string | guid, mandatory
    searchInPortal: true, // boolean;
    searchTerm: '', // string;
    catalog: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_catalog';
    category: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_category';
    assertItem: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_cat_item';
    assert: '', // 'assert_item_present' | 'assert_item_not_present';
}): { catalog_item_id: string; };

// Checkout the Shopping Cart and generates a new request.
atf.server.checkoutShoppingCart({ // all props are mandatory
    $id: Now.ID[''], // string | guid, mandatory
    assert: '', // 'empty_cart' | 'checkout_successfull'
    requestedFor: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_user';
    deliveryAddress: '123 main st', // string
    specialInstructions: 'none', // string
}): { request_id: string; };

// Replays a previously created request item with the same values and options.
atf.server.replayRequestItem({
    $id: Now.ID[''], // string | guid, mandatory
    request_item: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sc_req_item';
}): {
    table: string;
    req_item: sys_id | Record 'sc_req_item';
};
```

### ATF Fluent Spec: server.queryvalidate

### Context

This chunk provides APIs within the ServiceNow Automated Test Framework (ATF) for performing server-side record queries and validations. These APIs facilitate automated testing by enabling testers to verify the existence of records and validate their conditions on the server without involving the user interface. These actions are crucial for ensuring that data integrity and business rules are upheld through backend validations and queries during testing.

```typescript
// Perform a database query to verify if a record matching the conditions set in this step are met
atf.server.recordQuery({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  fieldValues: '', // string, servicenow encoded query. ALWAYS use the get_table_schema tool to find the field names and types for the table.
  enforceSecurity: false, // boolean;
  assert: 'records_match_query', // 'records_match_query' | 'no_records_match_query';
}): { table: string;
    first_record: string; // sys_id of the first record
};

// Validates that a given record meets the specified conditions on the server-side.
atf.server.recordValidation({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table:'', // table name
  fieldValues: '', // string, servicenow encoded query. ALWAYS use the get_table_schema tool to find the field names and types for the table.
  recordId: '', // sys_id of the record
  enforceSecurity: false, // boolean
  assert: 'record_validated', // 'record_validated' | 'record_not_found';
}): void;
```

### ATF Fluent Spec: server.record

### Context

The chunk provides ServiceNow's Automated Test Framework (ATF) APIs for performing server-side CRUD (Create, Read, Update, Delete) operations on records. These API steps allow testers to insert, update, and delete records directly from the server without UI interaction. The chunk is relevant for testers needing to programmatically manage database records within automated tests, ensuring controlled and repeatable test environments.

```typescript
// Inserts a record into a table. Specify the field values to set on the new record, outputs the table and the sys_id of the new record.
atf.server.recordInsert({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table: '',
  fieldValues: {}, // a valid JSON object, e.g. { "field_one": "value1", "field_two": "value2" }. ALWAYS use get_table_schema tool to find the mandatory fields so they can be filled out. Pay attention to the field types.
  assert: '', // 'record_successfully_inserted' | 'record_not_inserted';
  enforceSecurity: false, // boolean;
}): { table: string;
  record_id: string; // sys_id of the new record
};

// Changes field values of a record on the server.
// follow this step with atf.server.recordValidation step to ensure that the changes were applied.
atf.server.recordUpdate({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table: '',
  fieldValues: {}, // a valid JSON object, e.g. { "field_one": "value1", "field_two": "value2" }. ALWAYS use get_table_schema tool to find the mandatory fields so they can be filled out. Pay attention to the field types.
  recordId: '', // sys_id of the record
  assert: 'record_successfully_updated', // 'record_successfully_updated' | 'record_not_updated';
  enforceSecurity: false, // boolean;
}): void;

// Deletes a record of a table.
atf.server.recordDelete({ // all props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  table: '', // table name
  recordId: '', // sys_id of the record
  enforceSecurity: false, // boolean
  assert: 'record_successfully_deleted', // 'record_successfully_deleted' | 'record_not_deleted';
}): void;
```

---

### Example

```javascript
import "@servicenow/sdk/global";
import { Test } from "@servicenow/sdk/core";

/**
 * ATF Test: Order iPhone through Standard UI
 *
 * This test:
 * 1. Opens the Apple iPad 3 catalog item in the standard UI
 * 2. Submits the order
 * 3. Asserts that the order was submitted successfully to the server
 * 4. Verifies a request record was created
 */
Test(
  {
    $id: Now.ID["iphone_order_test"],
    name: "Order iPhone - Standard UI",
    description:
      "Open iPhone catalog item, submit the order, and verify submission",
    active: true,
    failOnServerError: true
  },
  atf => {
    // Log that we're starting the test
    atf.server.log({
      $id: Now.ID["start_log_step"],
      log: "Starting iPhone order test through standard UI"
    });

    // Open the iPad 3 catalog item
    atf.server.searchForCatalogItem({
      $id: Now.ID["search_iphone_step"],
      searchTerm: "Apple iPad 3",
      catalog: "e0d08b13c3330100c8b837659bba8fb4", // Service Catalog
      category: "d258b953c611227a0146101fb1be7c31", // Hardware category
      assertItem: "060f3afa3731300054b6a3549dbe5d3e", // iPad 3 (standard, not pro)
      assert: "assert_item_present",
      searchInPortal: false
    });

    // Log that the catalog item was found
    atf.server.log({
      $id: Now.ID["found_item_log_step"],
      log: "iPad 3 catalog item found"
    });

    // Order the catalog item (automatically adds it to cart and checks out)
    const requestResult = atf.server.checkoutShoppingCart({
      $id: Now.ID["checkout_step"],
      requestedFor: "current", // Current user
      deliveryAddress: "123 ServiceNow Way, San Diego, CA",
      specialInstructions: "Test order from ATF",
      assert: "checkout_successfull"
    });

    // Log that the order was submitted successfully
    atf.server.log({
      $id: Now.ID["success_log_step"],
      log: "iPad 3 order submitted successfully"
    });

    // Verify that a request record was created with the expected information
    atf.server.recordValidation({
      $id: Now.ID["verify_request_step"],
      table: "sc_request",
      recordId: "6eed229047801200e0ef563dbb9a71c2",
      fieldValues: "short_descriptionLIKEApple iPad 3", // ALWAYS first use get_table_schema tool to find the field names and types for the table.
      assert: "record_validated",
      enforceSecurity: false
    });
  }
);
```

---

## Notes

- `runQuery` is required for any fields referencing sys_id values from tables like `sys_user`, `sys_user_group`, `sys_user_role`, `sc_catalog`, `sc_category`, `sc_cat_item`, and `sc_req_item`.
- Use snake_case keys and double-quoted values in `fieldValues`. Use the get_table_schema tool to find the field names and types for the table.
- All test steps that output a `record_id` or `user` can be chained with later steps using that ID.
- Literal enums like `assert` must be used as-is and validated.
