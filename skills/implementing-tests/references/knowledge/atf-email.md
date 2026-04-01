# ATF_EMAIL

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
- The `atf` object provides access for ATF Email plugin methods `email`.
- All ATF tests must be wrapped in this structure.

---

## Now SDK Constructor

This metadata type is defined using the `email` property of the ATF Fluent API.

```ts
atf.email.validateOutboundEmail({...})
atf.email.validateOutboundEmailGeneratedByNotification({...})
atf.email.validateOutboundEmailGeneratedByFlow({...})
atf.email.generateInboundEmail({...})
atf.email.generateInboundReplyEmail({...})
atf.email.generateRandomString({...})
```

---

## Example

```ts
import { Now } from "@servicenow/sdk/core";

atf.email.validateOutboundEmail({
  $id: Now.ID["abc123"],
  conditions: "subjectLIKEWelcome^bodyLIKEHello"
});

atf.email.validateOutboundEmailGeneratedByFlow({
  $id: Now.ID["flow001"],
  sourceFlow: "sys_flow_123",
  conditions: "recipientsLIKEadmin"
});

atf.email.generateInboundEmail({
  $id: Now.ID["in001"],
  from: "user@example.com",
  to: "support@example.com",
  subject: "Issue Reported",
  body: "Details of the issue..."
});
```

---

## Properties

### `validateOutboundEmail`

| Name         | Type     | Mandatory | Needs runQuery Tool? | Description                         |
| ------------ | -------- | --------- | -------------------- | ----------------------------------- |
| `$id`        | `string` | Yes       | No                   | Unique identifier for the test step |
| `conditions` | `string` | No        | No                   | Encoded query used to match emails  |

### `validateOutboundEmailGeneratedByNotification`

| Name                 | Type                                       | Mandatory | Needs runQuery Tool? | Description                                                          |
| -------------------- | ------------------------------------------ | --------- | -------------------- | -------------------------------------------------------------------- |
| `$id`                | `string`                                   | Yes       | No                   | Unique identifier                                                    |
| `sourceNotification` | `string , Record<'sysevent_email_action'>` | Yes       | Yes                  | Source notification sys_id (use runQuery on `sysevent_email_action`) |
| `conditions`         | `string`                                   | No        | No                   | Email filter condition                                               |

### `validateOutboundEmailGeneratedByFlow`

| Name         | Type                              | Mandatory | Needs runQuery Tool? | Description                                         |
| ------------ | --------------------------------- | --------- | -------------------- | --------------------------------------------------- |
| `$id`        | `string`                          | Yes       | No                   | Unique identifier                                   |
| `sourceFlow` | `string , Record<'sys_hub_flow'>` | Yes       | Yes                  | Source flow sys_id (use runQuery on `sys_hub_flow`) |
| `conditions` | `string`                          | No        | No                   | Encoded query to find the email                     |

### `generateInboundEmail`

| Name      | Type     | Mandatory | Needs runQuery Tool? | Description          |
| --------- | -------- | --------- | -------------------- | -------------------- |
| `$id`     | `string` | Yes       | No                   | Unique ID            |
| `from`    | `string` | Yes       | No                   | Sender email address |
| `to`      | `string` | Yes       | No                   | Recipient email      |
| `subject` | `string` | Yes       | No                   | Email subject        |
| `body`    | `string` | Yes       | No                   | Email body content   |

### `generateInboundReplyEmail`

| Name           | Type     | Mandatory | Needs runQuery Tool? | Description                                                      |
| -------------- | -------- | --------- | -------------------- | ---------------------------------------------------------------- |
| `$id`          | `string` | Yes       | No                   | Unique ID                                                        |
| `targetTable`  | `string` | Yes       | No                   | Name of the target table                                         |
| `targetRecord` | `string` | Yes       | Yes                  | sys_id of the record to reply to (use runQuery on `targetTable`) |
| `subject`      | `string` | Yes       | No                   | Email subject                                                    |
| `body`         | `string` | Yes       | No                   | Email body                                                       |
| `from`         | `string` | Yes       | No                   | Sender address                                                   |
| `to`           | `string` | Yes       | No                   | Recipient address                                                |

### `generateRandomString`

| Name     | Type     | Mandatory | Needs runQuery Tool? | Description                           |
| -------- | -------- | --------- | -------------------- | ------------------------------------- |
| `$id`    | `string` | Yes       | No                   | Unique ID                             |
| `length` | `number` | No        | No                   | Length of random string (max: 10,000) |

---

## Valid Values

| Property | Valid Values         |
| -------- | -------------------- |
| `length` | Any number \<= 10000 |

---

## ATF Specs

### Context

This chunk describes the Automated Test Framework (ATF) APIs used to test email functionalities within ServiceNow. It details methods for validating and generating email records, including outbound emails sent from notifications or flows, and simulating inbound emails or replies. The APIs allow for searching and filtering emails in testing scenarios, as well as mocking email content with generated strings. This chunk is relevant for automating email-related testing processes using ServiceNow's ATF.

```typescript
// filters the Email [sys_email] table to find an email that was sent from a notification during testing
atf.email.validateOutboundEmail({
  $id: Now.ID[''], // string | guid, mandatory
  conditions: '', //string, Servicenow encoded query
}):void;

// Filters the Email [sys_email] table to find an email that was sent from a flow during testing.
atf.email.validateOutboundEmailGeneratedByFlow({
  $id: Now.ID[''], // string | guid, mandatory
  sourceFlow: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sys_hub_flow';
  conditions: '', //string, Servicenow encoded query
}): void;

// Filters the Email [sys_email] table to find an email that was sent from a notification during testing.
atf.email.validateOutboundEmailGeneratedByNotification({
  $id: Now.ID[''], // string | guid, mandatory
  sourceNotification: '', //need to use runQuery tool to get the EXACT sys_id of the record in table 'sysevent_email_action';
  conditions: '', //string, Servicenow encoded query
}): void;

// Generates a new inbound email. This step also creates an email.read event upon step completion
atf.email.generateInboundEmail({
  $id: Now.ID[''], // string | guid, mandatory
  from: '',
  to: '',
  subject: '',
  body: ''
}): { output_email_record: ''};

// generates an Email [sys_email] record that looks like an email sent in reply to a system notification. This step also creates an email.read event upon step completion.
atf.email.generateInboundReplyEmail({
  $id: Now.ID[''], // string | guid, mandatory
  targetTable:''
  targetRecord: '', // need to use runQuery tool to get the EXACT sys_id of the record in table 'targetTable';
  subject: '',
  body: '',
  from: '',
  to: '',
}): { output_reply_email_record: ''}; // recordId of the inbound reply email

// Generates a string that can be used as test data for another test step. By default, the string is 10 characters long. The maximum length of the string is 10,000 characters.
atf.email.generateRandomString({
  $id: Now.ID[''], // string | guid, mandatory
  length: 1024, // number
}): { random_string: '' };
```

---

### Example

```javascript
import "@servicenow/sdk/global";
import { Test } from "@servicenow/sdk/core";

Test(
  {
    $id: Now.ID["test_email_generation"],
    name: "Generate Test Email",
    description: "Generates a test email with random text content",
    active: true,
    failOnServerError: true
  },
  atf => {
    // Step 1: Generate random text for email body
    atf.email.generateRandomString({
      $id: Now.ID["generate_random_body"],
      length: 512
    });

    // Step 2: Generate the email with the random text
    atf.email.generateInboundEmail({
      $id: Now.ID["send_test_email"],
      from: "t2t@pale.com",
      to: "customer0@pale.com",
      subject: "test email from text2fluent",
      body: "fdadfafa"
    });
  }
);
```

---
