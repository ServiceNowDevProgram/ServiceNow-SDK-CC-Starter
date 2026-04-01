# ATF_RESPONSIVEDASHBOARD

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
- The `atf` object provides access for ATF Responsive Dashboard plugin methods `responsiveDashboard`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

```ts
atf.responsiveDashboard.responsiveDashboardVisibility({...})
atf.responsiveDashboard.responsiveDashboardSharing({...})
```

---

## Properties

### `responsiveDashboardVisibility`

| Name        | Type                                                  | Default                  | Mandatory | Needs runQuery Tool? | Description                                          |
| ----------- | ----------------------------------------------------- | ------------------------ | --------- | -------------------- | ---------------------------------------------------- |
| `$id`       | `string`                                              | —                        | Yes       | No                   | Unique identifier for this test step                 |
| `dashboard` | `string , Record<'pa_dashboards'>`                    | —                        | Yes       | Yes                  | The sys_id or record of the dashboard being verified |
| `assert`    | `'dashboard_is_visible' , 'dashboard_is_not_visible'` | `'dashboard_is_visible'` | No        | No                   | Type of visibility assertion being performed         |

### `responsiveDashboardSharing`

| Name        | Type                                               | Default                 | Mandatory | Needs runQuery Tool? | Description                                                        |
| ----------- | -------------------------------------------------- | ----------------------- | --------- | -------------------- | ------------------------------------------------------------------ |
| `$id`       | `string`                                           | —                       | Yes       | No                   | Unique identifier for this test step                               |
| `dashboard` | `string , Record<'pa_dashboards'>`                 | —                       | Yes       | Yes                  | The sys_id or record of the dashboard whose shareability is tested |
| `assert`    | `'can_share_dashboard' , 'cannot_share_dashboard'` | `'can_share_dashboard'` | No        | No                   | Assertion for whether the user can share the dashboard             |

---

## Valid Values

| Property                                 | Valid Values                                           |
| ---------------------------------------- | ------------------------------------------------------ |
| `assert` (responsiveDashboardVisibility) | `'dashboard_is_visible'`, `'dashboard_is_not_visible'` |
| `assert` (responsiveDashboardSharing)    | `'can_share_dashboard'`, `'cannot_share_dashboard'`    |

---

## ATF Specs

### Context

This chunk is part of the ServiceNow Automated Test Framework (ATF) documentation and focuses on validating user-level access and permissions for responsive dashboards. The tests allow confirmation of both visibility and shareability using sys_id lookups from the `pa_dashboards` table.

```typescript
atf.responsiveDashboard.responsiveDashboardSharing({
  $id: Now.ID[''], // string | guid, mandatory
  dashboard: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'pa_dashboards';
  assert: 'can_share_dashboard',// 'can_share_dashboard' | 'cannot_share_dashboard';
}): void;

atf.responsiveDashboard.responsiveDashboardVisibility({
  $id: Now.ID[''], // string | guid, mandatory
  dashboard:'', // need to use runQuery tool to get the EXACT sys_id of the record in table 'pa_dashboards';
  assert: 'dashboard_is_visible', // 'dashboard_is_visible' | 'dashboard_is_not_visible';
}): void;
```

---

### Example

```javascript
import "@servicenow/sdk/global";
import { Test } from "@servicenow/sdk/core";

Test(
  {
    $id: Now.ID["dashboard_share_test"],
    name: "Dashboard Share Test",
    description:
      "Test that shares the Subscription Overview dashboard and verifies it was shared",
    active: true,
    failOnServerError: true
  },
  atf => {
    // Step 1: Share the dashboard
    atf.responsiveDashboard.responsiveDashboardSharing({
      $id: Now.ID["share_dashboard_step"],
      dashboard: "5e1b81e053031010b643ddeeff7b1266", // Subscription Overview dashboard sys_id
      assert: "can_share_dashboard"
    });

    // Step 2: Verify the dashboard is visible (confirming it's shared)
    atf.responsiveDashboard.responsiveDashboardVisibility({
      $id: Now.ID["verify_dashboard_visible"],
      dashboard: "5e1b81e053031010b643ddeeff7b1266", // Subscription Overview dashboard sys_id
      assert: "dashboard_is_visible"
    });
  }
);
```

---

## Notes

- Use the `runQuery` tool to look up valid sys_id values from the `pa_dashboards` table.
- `dashboard` can be passed as a raw sys_id `string` or as a `Record<'pa_dashboards'>`.
- Default values for `assert` are applied automatically if omitted.
- These tests are useful for confirming personalized or role-based dashboard access and sharing capabilities in multi-user test flows.
