# ATF_REPORTING

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
- The `atf` object provides access for ATF Report plugin methods `reporting`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

This metadata type is defined using the `reporting` property of the ATF Fluent API.

```ts
atf.reporting.reportVisibility({...})
```

---

## Properties

### `reportVisibility`

| Name     | Type                                       | Default             | Mandatory | Needs runQuery Tool? | Description                                                  |
| -------- | ------------------------------------------ | ------------------- | --------- | -------------------- | ------------------------------------------------------------ |
| `report` | `string , Record<'sys_report'>`            | —                   | Yes       | Yes                  | sys_id or Record reference to a report in `sys_report` table |
| `assert` | `'can_view_report' , 'cannot_view_report'` | `'can_view_report'` | No        | No                   | Assertion for whether the report should be viewable          |

---

## Valid Values

| Property | Valid Values                                |
| -------- | ------------------------------------------- |
| `assert` | `'can_view_report'`, `'cannot_view_report'` |

---

## ATF Specs

### Context

This chunk is part of the ServiceNow Automated Test Framework (ATF) documentation and focuses on verifying if a user can or cannot view a specific report.

```typescript
atf.reporting.reportVisibility({
  $id: Now.ID[''], // string | guid, mandatory
  report: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_report';
  assert: 'can_view_report', // 'can_view_report' | 'cannot_view_report';
}): void;
```

---

### Example

```javascript
import "@servicenow/sdk/global";
import { Test } from "@servicenow/sdk/core";

Test(
  {
    $id: Now.ID["report_visibility_test"],
    name: "Check Report Visibility",
    description: "Opens a report, asserts its invisibility",
    failOnServerError: true
  },
  atf => {
    atf.reporting.reportVisibility({
      $id: Now.ID["report_visibility_step"],
      report: "00ae51c781f21010f8773175af5ad90e",
      assert: "can_view_report"
    });
  }
);
```

---

## Notes

- Use the `runQuery` tool to retrieve the exact `sys_id` of the report from the `sys_report` table.
- Both `string` and `Record<'sys_report'>` formats are supported for the `report` property; prefer `Record` type when available.
- The `assert` field allows testing both access and denial conditions, but defaults to asserting that the report _can_ be viewed.
- Useful for validating ACLs and roles associated with report visibility.
