# ATF_APPNAV

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
- The `atf` object provides access for ATF Application Navigator plugin methods `applicationNavigator`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

This metadata type is defined using the `applicationNavigator` property of the ATF Fluent API.

```ts
atf.applicationNavigator.moduleVisibility({...})
atf.applicationNavigator.navigateToModule({...})
atf.applicationNavigator.applicationMenuVisibility({...})
```

---

## Properties

### `moduleVisibility`

| Name                | Type                                                         | Default                          | Mandatory | Needs runQuery Tool? | Description                            |
| ------------------- | ------------------------------------------------------------ | -------------------------------- | --------- | -------------------- | -------------------------------------- |
| `$id`               | `string`                                                     | —                                | Yes       | No                   | Unique identifier for this test step   |
| `navigator`         | `'ui15' ,'ui16' ,'polaris'`                                  | `'ui15'`                         | Yes       | No                   | Navigation UI flavor                   |
| `assertNotVisible`  | `'at_least_modules_not_visible' ,'only_modules_not_visible'` | `'at_least_modules_not_visible'` | Yes       | No                   | Assertion mode for modules not visible |
| `notVisibleModules` | `Array<string ,Record<'sys_app_module'>>`                    | `[]`                             | Yes       | Yes                  | List of sys_id of modules not visible  |
| `assertVisible`     | `'at_least_modules_visible' ,'only_modules_visible'`         | `'at_least_modules_visible'`     | Yes       | No                   | Assertion mode for visible modules     |
| `visibleModules`    | `Array<string ,Record<'sys_app_module'>>`                    | `[]`                             | Yes       | Yes                  | List of sys_id of modules visible      |

### `navigateToModule`

| Name     | Type                               | Mandatory | Needs runQuery Tool? | Description                                         |
| -------- | ---------------------------------- | --------- | -------------------- | --------------------------------------------------- |
| `$id`    | `string`                           | Yes       | No                   | Unique identifier for this test step                |
| `module` | `string ,Record<'sys_app_module'>` | Yes       | Yes                  | sys_id of the module to simulate user navigation to |

### `applicationMenuVisibility`

| Name               | Type                                                                   | Default                               | Mandatory | Needs runQuery Tool? | Description                                        |
| ------------------ | ---------------------------------------------------------------------- | ------------------------------------- | --------- | -------------------- | -------------------------------------------------- |
| `$id`              | `string`                                                               | —                                     | Yes       | No                   | Unique identifier for this test step               |
| `navigator`        | `'ui15' ,'ui16' ,'polaris'`                                            | `'ui15'`                              | Yes       | No                   | Navigation UI flavor                               |
| `visible`          | `Array<string ,Record<'sys_app_application'>>`                         | `[]`                                  | Yes       | Yes                  | sys_id of application menus expected to be visible |
| `assertVisible`    | `'at_least_applications_visible' ,'only_applications_visible'`         | `'at_least_applications_visible'`     | Yes       | No                   | Assertion type for visible apps                    |
| `notVisible`       | `Array<string ,Record<'sys_app_application'>>`                         | `[]`                                  | Yes       | Yes                  | sys_id of application menus not visible            |
| `assertNotVisible` | `'at_least_applications_not_visible' ,'only_applications_not_visible'` | `'at_least_applications_not_visible'` | Yes       | No                   | Assertion type for apps not visible                |

---

## Valid Values

| Property           | Valid Values                                                                                                                             |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `navigator`        | `'ui15'`, `'ui16'`, `'polaris'`                                                                                                          |
| `assertVisible`    | `'at_least_modules_visible'`, `'only_modules_visible'`, `'at_least_applications_visible'`, `'only_applications_visible'`                 |
| `assertNotVisible` | `'at_least_modules_not_visible'`, `'only_modules_not_visible'`, `'at_least_applications_not_visible'`, `'only_applications_not_visible'` |

---

## ATF Specs

### Context

This chunk is part of the ServiceNow Automated Test Framework (ATF) documentation and focuses on testing and verifying application navigation. It includes APIs to test the visibility of modules and application menus in the ServiceNow platform's left navigation bar and provides steps to navigate specific modules programmatically, enhancing automated UI testing capabilities.

```ts
// Verifies visibility of modules in the left navigation bar.
atf.applicationNavigator.moduleVisibility({
  $id: Now.ID[''],
  navigator: 'polaris',
  assertNotVisible: 'at_least_modules_not_visible',
  notVisibleModules: [''],
  assertVisible: 'at_least_modules_visible',
  visibleModules: [''],
}): void;

// Navigates to a module, as if a user had clicked on it
atf.applicationNavigator.navigateToModule({
  $id: Now.ID[''],
  module: ''
}): void;

// Verifies visibility of application menus in the left navigation bar
atf.applicationNavigator.applicationMenuVisibility({
  $id: Now.ID[''],
  navigator: 'polaris',
  visible: [''],
  assertVisible: 'at_least_applications_visible',
  notVisible: [''],
  assertNotVisible: 'at_least_applications_not_visible'
}): void;
```

---

### Example

```ts
import { Test } from "@servicenow/sdk/core";
import "@servicenow/sdk-core/global";

Test(
  {
    $id: Now.ID["test_1_abc123"],
    name: "Incident Management Test Suite",
    description:
      "Test suite to impersonate user, query and update incident record",
    active: true,
    failOnServerError: true
  },
  atf => {
    atf.applicationNavigator.moduleVisibility({
      $id: Now.ID["abc123"],
      navigator: "polaris",
      assertNotVisible: "at_least_modules_not_visible",
      notVisibleModules: [""],
      assertVisible: "at_least_modules_visible",
      visibleModules: [
        "08709793534233004cb4ddeeff7b12c7",
        "5d7279f0531200101829ddeeff7b1226"
      ]
    });

    atf.applicationNavigator.navigateToModule({
      $id: Now.ID["xyz789"],
      module: "incident_create_sys_id"
    });

    atf.applicationNavigator.applicationMenuVisibility({
      $id: Now.ID["menu001"],
      navigator: "polaris",
      visible: ["18f23e670a0a0b2400c55b122be52373"],
      assertVisible: "at_least_applications_visible",
      notVisible: [""],
      assertNotVisible: "at_least_applications_not_visible"
    });
  }
);
```

---

## Notes

- All fields shown as mandatory are based strictly on your JSDoc definitions.
- `sys_id` values must be retrieved via `runQuery` tool from the respective tables: `sys_app_module` and `sys_app_application`.
- Avoid using external links or client-side modules with `navigateToModule()`.

```

```
