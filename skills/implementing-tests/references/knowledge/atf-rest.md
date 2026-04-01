# ATF_REST

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
- The `atf` object provides access for ATF Rest plugin methods `rest`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

This metadata type is defined using the `rest` property of the ATF Fluent API.

```ts
atf.rest.sendRestRequest({...})
atf.rest.assertStatusCodeName({...})
atf.rest.assertStatusCode({...})
atf.rest.assertResponseTime({...})
atf.rest.assertResponseHeader({...})
atf.rest.assertResponsePayload({...})
atf.rest.assertResponseJSONPayloadIsValid({...})
atf.rest.assertJsonResponsePayloadElement({...})
atf.rest.assertResponseXMLPayloadIsWellFormed({...})
atf.rest.assertXMLResponsePayloadElement({...})
```

---

## Properties

### `sendRestRequest`

| Name                  | Type                                       | Default   | Mandatory | Needs runQuery Tool? | Description                               |
| --------------------- | ------------------------------------------ | --------- | --------- | -------------------- | ----------------------------------------- |
| `$id`                 | `string`                                   | —         | Yes       | No                   | Unique identifier for this test step      |
| `path`                | `string`                                   | —         | Yes       | No                   | API path (e.g. `/api/now/table/incident`) |
| `body`                | `string`                                   | —         | Yes       | No                   | JSON string representing request body     |
| `auth`                | `'basic' ,'mutual' ,''`                    | `'basic'` | Yes       | No                   | Type of authentication used               |
| `mutualAuth`          | `string ,Record<'sys_certificate'>`        | `''`      | No        | Yes                  | sys_id of the mutual TLS certificate      |
| `basicAuthentication` | `string ,Record<'sys_auth_profile_basic'>` | `''`      | No        | Yes                  | sys_id of the basic auth profile          |
| `method`              | `'get' ,'post' ,'put' ,'delete' ,'patch'`  | `'get'`   | No        | No                   | HTTP method to be used                    |
| `queryParameters`     | `Record<string, string>`                   | `{}`      | No        | No                   | Key-value map of query parameters         |
| `headers`             | `Record<string, string>`                   | `{}`      | No        | No                   | Key-value map of request headers          |

### `assertStatusCodeName`

| Name        | Type                                                     | Default      | Mandatory | Needs runQuery Tool? | Description                            |
| ----------- | -------------------------------------------------------- | ------------ | --------- | -------------------- | -------------------------------------- |
| `$id`       | `string`                                                 | —            | Yes       | No                   | Unique identifier for this test step   |
| `operation` | `'contains' ,'does_not_contain' ,'equals' ,'not_equals'` | `'contains'` | No        | No                   | Comparison operation for status name   |
| `codeName`  | `string`                                                 | —            | Yes       | No                   | Expected status code name (e.g., `OK`) |

### `assertStatusCode`

| Name         | Type                                                                                             | Default    | Mandatory | Needs runQuery Tool? | Description                          |
| ------------ | ------------------------------------------------------------------------------------------------ | ---------- | --------- | -------------------- | ------------------------------------ |
| `$id`        | `string`                                                                                         | —          | Yes       | No                   | Unique identifier for this test step |
| `operation`  | `'equals' ,'not_equals' ,'less_than' ,'greater_than' ,'less_than_equals' ,'greater_than_equals'` | `'equals'` | No        | No                   | Comparison operation for status code |
| `statusCode` | `number`                                                                                         | —          | Yes       | No                   | Expected numeric HTTP status code    |

### `assertResponseTime`

| Name           | Type                          | Default       | Mandatory | Needs runQuery Tool? | Description                            |
| -------------- | ----------------------------- | ------------- | --------- | -------------------- | -------------------------------------- |
| `$id`          | `string`                      | —             | Yes       | No                   | Unique identifier for this test step   |
| `operation`    | `'less_than' ,'greater_than'` | `'less_than'` | No        | No                   | Comparison for expected response time  |
| `responseTime` | `number`                      | —             | Yes       | No                   | Expected response time in milliseconds |

### `assertResponseHeader`

| Name          | Type                                                               | Default    | Mandatory | Needs runQuery Tool? | Description                            |
| ------------- | ------------------------------------------------------------------ | ---------- | --------- | -------------------- | -------------------------------------- |
| `$id`         | `string`                                                           | —          | Yes       | No                   | Unique identifier for this test step   |
| `headerName`  | `string`                                                           | —          | Yes       | No                   | Name of the response header to assert  |
| `operation`   | `'equals' ,'not_equals' ,'exists' ,'contains' ,'does_not_contain'` | `'exists'` | No        | No                   | Operation to apply on the header value |
| `headerValue` | `string`                                                           | —          | Yes       | No                   | Expected header value                  |

### `assertResponsePayload`

| Name           | Type                                                               | Default      | Mandatory | Needs runQuery Tool? | Description                               |
| -------------- | ------------------------------------------------------------------ | ------------ | --------- | -------------------- | ----------------------------------------- |
| `$id`          | `string`                                                           | —            | Yes       | No                   | Unique identifier for this test step      |
| `responseBody` | `string`                                                           | —            | Yes       | No                   | Expected string value of response payload |
| `operation`    | `'equals' ,'not_equals' ,'exists' ,'contains' ,'does_not_contain'` | `'contains'` | No        | No                   | Operation for validating payload content  |

### `assertResponseJSONPayloadIsValid`

| Name  | Type     | Default | Mandatory | Needs runQuery Tool? | Description                          |
| ----- | -------- | ------- | --------- | -------------------- | ------------------------------------ |
| `$id` | `string` | —       | Yes       | No                   | Unique identifier for this test step |

### `assertJsonResponsePayloadElement`

| Name           | Type                                                               | Default    | Mandatory | Needs runQuery Tool? | Description                            |
| -------------- | ------------------------------------------------------------------ | ---------- | --------- | -------------------- | -------------------------------------- |
| `$id`          | `string`                                                           | —          | Yes       | No                   | Unique identifier for this test step   |
| `elementName`  | `string`                                                           | —          | Yes       | No                   | JSON SNC path to target element        |
| `elementValue` | `string`                                                           | —          | Yes       | No                   | Expected value of the JSON element     |
| `operation`    | `'contains' ,'does_not_contain' ,'equals' ,'not_equals' ,'exists'` | `'exists'` | No        | No                   | Comparison type for element validation |

### `assertResponseXMLPayloadIsWellFormed`

| Name  | Type     | Default | Mandatory | Needs runQuery Tool? | Description                          |
| ----- | -------- | ------- | --------- | -------------------- | ------------------------------------ |
| `$id` | `string` | —       | Yes       | No                   | Unique identifier for this test step |

### `assertXMLResponsePayloadElement`

| Name           | Type                                                               | Default    | Mandatory | Needs runQuery Tool? | Description                          |
| -------------- | ------------------------------------------------------------------ | ---------- | --------- | -------------------- | ------------------------------------ |
| `$id`          | `string`                                                           | —          | Yes       | No                   | Unique identifier for this test step |
| `elementPath`  | `string`                                                           | —          | Yes       | No                   | XPath to the target XML element      |
| `elementValue` | `string`                                                           | —          | Yes       | No                   | Expected value of the XML element    |
| `operation`    | `'contains' ,'does_not_contain' ,'equals' ,'not_equals' ,'exists'` | `'exists'` | No        | No                   | Comparison type for XML validation   |

---

## Valid Values

| Property    | Valid Values                                                                                                                                                                                   |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `auth`      | `''`, `'basic'`, `'mutual'`                                                                                                                                                                    |
| `method`    | `'get'`, `'post'`, `'put'`, `'delete'`, `'patch'`                                                                                                                                              |
| `operation` | (varies per function):<br>- `'equals'`, `'not_equals'`, `'contains'`, `'does_not_contain'`, `'exists'`<br>- `'less_than'`, `'greater_than'`<br>- `'less_than_equals'`, `'greater_than_equals'` |

---

## ATF Specs

### ATF Fluent Spec: rest

### Context:

This chunk is part of the ServiceNow Automated Test Framework (ATF) documentation and focuses on REST API testing. It includes APIs for sending HTTP requests using various methods with specific headers and bodies, and making assertions on response status codes, response times, headers, and payloads. These APIs are crucial for validating the behavior and performance of REST APIs during testing within the ATF environment.

```typescript
// Send a REST API request to glide instance with specified HTTP method, path, query parameters, request headers and body.
atf.rest.sendRestRequest({ // each of the following props are mandatory
  $id: Now.ID[''], // string | guid, mandatory
  path: '', // string, ex. '/api/now/table/incident'
  body: '', // string, ex. "{data_one: 'value1', data_field: 'value2', data_three: 'value3'}"
  mutualAuth: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_certificate';
  auth: '',// '' | 'basic' | 'mutual'
  mutualAuth: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_certificate';
  basicAuthentication: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_auth_profile_basic';
  method: 'get', // 'get' | 'post' | 'put' | 'delete' | 'patch'
  queryParameters: {}, // object, snake_case name value pairs, ex.: { field_one: 'value1', field_two: 'value2' }
  headers: {}, // object, snake_case name value pairs, ex.: { header_one: 'value1', header_two: 'value2' }
}): void

// Assert the REST API's HTTP response status code name
atf.rest.assertStatusCodeName({
  $id: Now.ID[''], // string | guid, mandatory
  operation: 'equals', // 'contains' | 'does_not_contain' | 'equals' | 'not_equals'
  codeName: '', // string, ex. 'OK'
}): void

// Assert the HTTP response status code
atf.rest.assertStatusCode({
  $id: Now.ID[''], // string | guid, mandatory
  operation: 'equals',// 'equals' | 'not_equals' | 'less_than' | 'greater_than' | 'less_than_equals' |'greater_than_equals'
  statusCode: 200, // number
}): void

// Assert the HTTP response time
atf.rest.assertResponseTime({
  $id: Now.ID[''], // string | guid, mandatory
  operation: 'less_than', // 'less_than' | 'greater_than'
  responseTime: 3000, // number, in milliseconds
}): void

// Assert an HTTP response header. Select the comparison operation and specify the expected value of the header.
atf.rest.assertResponseHeader({
  $id: Now.ID[''], // string | guid, mandatory
  headerName: '', // string
  operation: 'equals', // 'equals' | 'not_equals' | 'exists' | 'contains' | 'does_not_contain'
  headerValue: '', // string
}): void
```

### ATF Fluent Spec: rest.assertpayload

### Context:

The chunk is part of the ServiceNow Automated Test Framework (ATF) focusing on REST API testing. It details steps for validating response payloads from REST API requests, including assertions for HTTP response body values, JSON payload elements, XML payload structure, and elements. These API steps are crucial for ensuring that REST endpoints function correctly during automated tests by verifying expected data structures and values in responses.

```typescript
// Assert the HTTP response payload is equals to or contains a specified value
atf.rest.assertResponsePayload({
  $id: Now.ID[''], // string | guid, mandatory
  responseBody: '', // string, expected response body
  operation: 'equals' // 'equals' | 'not_equals' | 'exists' | 'contains' | 'does_not_contain'
}): void

// Assert the JSON response payload is valid.
atf.rest.assertResponseJSONPayloadIsValid({
  $id: Now.ID[''], // string | guid, mandatory
}): void // input object with no properties

// Assert the JSON response payload element
atf.rest.assertJsonResponsePayloadElement({
  $id: Now.ID[''], // string | guid, mandatory
  elementName: '', // string, JSON SNC path of the element
  operation: 'contains', // 'contains' | 'does_not_contain' | 'equals' | 'not_equals' | 'exists'
  elementValue: '' // string, value to compare
}): void

// Assert the REST API's XML response payload is well-formed
atf.rest.assertResponseXMLPayloadIsWellFormed({
  $id: Now.ID[''], // string | guid, mandatory
}): void // input object with no properties

// Assert the REST API's XML response payload element
atf.rest.assertXMLResponsePayloadElement({
  $id: Now.ID[''], // string | guid, mandatory
  elementPath: '', // xpath to the element
  operation: 'contains', // 'contains' | 'does_not_contain' | 'equals' | 'not_equals' | 'exists'
  elementValue: '' // string, value to compare
}): void
```

---

### Example

```javascript
import "@servicenow/sdk/global";
import { Test } from "@servicenow/sdk/core";

/**
 * ATF Test: Scaffold API Test
 *
 * This test:
 * 1. Sends a GET request to '/api/now/fluent/scaffold' with query param new=true
 * 2. Verifies the JSON payload is valid
 * 3. Validates that the response contains 'result: success'
 * 4. Validates the status code is OK (200)
 */
Test(
  {
    $id: Now.ID["scaffold_api_test"],
    name: "Scaffold API Test",
    description:
      "Tests the scaffold API endpoint returns success and proper status",
    active: true,
    failOnServerError: true
  },
  atf => {
    // Send the GET request to the scaffold API endpoint with query parameter new=true
    atf.rest.sendRestRequest({
      $id: Now.ID["send_scaffold_request"],
      path: "/api/now/fluent/scaffold",
      body: "",
      auth: "basic",
      method: "get",
      queryParameters: {
        new: "true"
      },
      headers: {}
    });

    // Assert the response status code name is OK
    atf.rest.assertStatusCodeName({
      $id: Now.ID["assert_status_name"],
      operation: "equals",
      codeName: "OK"
    });

    // Assert the response status code is 200
    atf.rest.assertStatusCode({
      $id: Now.ID["assert_status_code"],
      operation: "equals",
      statusCode: 200
    });

    // Validate the response is valid JSON
    atf.rest.assertResponseJSONPayloadIsValid({
      $id: Now.ID["assert_json_valid"]
    });

    // Validate the 'result' element in the JSON response is 'success'
    atf.rest.assertJsonResponsePayloadElement({
      $id: Now.ID["assert_result_success"],
      elementName: "result",
      operation: "equals",
      elementValue: "success"
    });
  }
);
```

---

## Notes

- Always retrieve the correct `sys_id` for:
  - `mutualAuth` from `sys_certificate`
  - `basicAuthentication` from `sys_auth_profile_basic`
- Use literal string values for simple asserts, and `Record<'sys_...'>` when referencing instance records dynamically.
- Payload-related assertions operate strictly on raw values, ensure correct quoting and escaping.
- Operations like `equals`, `contains`, and `exists` affect how expected values are validated—choose them based on context.
