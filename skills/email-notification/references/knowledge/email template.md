# Email Template

# ServiceNow Email Templates - Record API Reference

Create email templates using the Record API to define reusable email content for notifications.

### Data Properties

| Field           | Type          | Required | Default    | Description                                                                                                               |
| --------------- | ------------- | -------- | ---------- | ------------------------------------------------------------------------------------------------------------------------- |
| `name`          | `String`      | Yes      | -          | Name of the template (max length: 100, must be unique)                                                                    |
| `message_text`  | `EmailScript` | No       | -          | Plain text version of the message (max length: 4000)                                                                      |
| `message_list`  | `List`        | No       | -          | Push Messages (reference to sys_push_notif_msg)                                                                           |
| `sys_version`   | `String`      | No       | '2'        | Notification Version (V1 or V2)                                                                                           |
| `message_html`  | `HTMLScript`  | No       | -          | HTML version of the message (max length: 4000)                                                                            |
| `sms_alternate` | `EmailScript` | No       | -          | SMS alternate message (max length: 8000)                                                                                  |
| `message`       | `EmailScript` | No       | -          | Message content (max length: 4000)                                                                                        |
| `collection`    | `TableName`   | No       | 'incident' | Target table name                                                                                                         |
| `email_layout`  | `Reference`   | No       | -          | Reference to sys_email_layout. Can reference an existing record using its sys_id or create a new one using the Record API |
| `subject`       | `String`      | No       | -          | Email subject (max length: 100)                                                                                           |

### Email Layout [sys_email_layout]

#### Data Properties

| Field             | Type      | Required | Default | Description                                  |
| ----------------- | --------- | -------- | ------- | -------------------------------------------- |
| `name`            | `String`  | Yes      | -       | Name of the layout (max length: 100)         |
| `layout`          | `HTML`    | No       | -       | HTML layout content (max length: 65536)      |
| `advanced`        | `Boolean` | No       | false   | Whether to use advanced XML layout           |
| `advanced_layout` | `XML`     | No       | -       | XML-based layout content (max length: 65000) |
| `description`     | `String`  | No       | -       | Description of the layout (max length: 1000) |

**Finding and Using Existing Layouts:**

#### Step 1: Find Layout by Name or Description

Call `run_query` tool:

- Table: `sys_email_layout`

#### Step 2: View Layout Contents

Once you have the sys_id, you can query the specific layout:

Call `run_query` tool:

- Table: `sys_email_layout`
- Encoded Query: `sys_id=b9edf0ca0a0a0b010035de2d6b579a03`

This will give you the complete layout configuration including HTML/XML content.

**Using the Layout:**

1. Reference by sys_id in email templates:

```javascript
// Use existing layout's sys_id
data: {
  email_layout: "b9edf0ca0a0a0b010035de2d6b579a03";
}
```

### Examples

```typescript
import { Record } from "@servicenow/sdk/core";

// Create only if doesn't exist
const incidentTemplate = Record({
  $id: Now.ID["incident_notification_template"],
  table: "sysevent_email_template",
  data: {
    name: "Incident Update Template",
    subject: "Incident \${number} has been updated",
    message_html: `
            <h1>Incident Update</h1>
            <p>Incident \${number} has been updated:</p>
            <ul>
                <li>State: \${state}</li>
                <li>Priority: \${priority}</li>
                <li>Assigned To: \${assigned_to}</li>
            </ul>
        `,
    collection: "incident",
    sys_version: "2"
  }
});
```

## Implementation Examples For Layout

1. Create new layout using Record API:

```javascript
const standardLayout = Record({
  $id: Now.ID["standard_email_layout"],
  table: "sys_email_layout",
  data: {
    name: "Standard Layout",
    layout: '<div class="email-container">\${email_body}</div>',
    description: "Standard email layout with container"
  }
});
```

1. Advanced XML layout example:

```javascript
const advancedLayout = Record({
  $id: Now.ID["advanced_email_layout"],
  table: "sys_email_layout",
  data: {
    name: "Advanced Layout",
    advanced: "true",
    advanced_layout: `
            <layout>
                <header>
                    <logo src="\${company_logo}"/>
                </header>
                <body>
                    \${email_body}
                </body>
                <footer>
                    <text>Powered by ServiceNow</text>
                </footer>
            </layout>
        `,
    description: "Advanced XML-based email layout"
  }
});
```

**Email Template with message_html:**

```javascript
import { Record } from "@servicenow/sdk/core";

// Create an email template
const incidentTemplate = Record({
  $id: Now.ID["incident_notification_template"],
  table: "sysevent_email_template",
  data: {
    name: "Incident Update",
    subject: "Incident \${number} has been updated",
    message_html: `
            <h1>Incident Update</h1>
            <p>Incident \${number} has been updated:</p>
            <ul>
                <li>State: \${state}</li>
                <li>Priority: \${priority}</li>
                <li>Assigned To: \${assigned_to}</li>
            </ul>
        `,
    collection: "incident",
    sys_version: "2"
  }
});
```

**Email Template with message_text:**

```javascript
const incidentMsgTemplate = Record({
  $id: Now.ID["incident_msg_template"],
  table: "sysevent_email_template",
  data: {
    name: "Incident Active",
    collection: "incident",
    message_text: `Hi, Active: \${active} `
  }
});
```

**Complete Example: Email Template with Layout;**

```javascript
// First create a layout (or use an existing one)
const incidentLayout = Record({
  $id: Now.ID["incident_email_layout"],
  table: "sys_email_layout",
  data: {
    name: "Incident Notification Layout",
    layout: `
            <div class="incident-notification">
                <header style="background-color: #f8f9fa; padding: 20px;">
                    <img src="\${company_logo}" alt="Company Logo" />
                    <h1 style="color: #1f70ff;">Incident Update</h1>
                </header>
                <main style="padding: 20px;">
                    \${notification:body}
                </main>
                <footer style="background-color: #f8f9fa; padding: 20px; text-align: center;">
                    <p>For assistance, contact IT Support</p>
                    <small>© \${current_year} \${company_name}</small>
                </footer>
            </div>
        `,
    description: "Standard layout for incident notifications"
  }
});

// Create email template using the layout
const incidentTemplate = Record({
  $id: Now.ID["incident_notification_template"],
  table: "sysevent_email_template",
  data: {
    name: "Incident Status Update",
    collection: "incident",
    message_html: `
            <h2>Incident \${number} - \${short_description}</h2>
            <table>
                <tr>
                    <th>Status:</th>
                    <td>\${state}</td>
                </tr>
                <tr>
                    <th>Priority:</th>
                    <td>\${priority}</td>
                </tr>
                <tr>
                    <th>Assigned To:</th>
                    <td>\${assigned_to}</td>
                </tr>
            </table>
            <h3>Updates</h3>
            <p>\${work_notes}</p>
        `,
    message_text: `
            Incident \${number} - \${short_description}
            
Status: \${state}
Priority: \${priority}
Assigned To: \${assigned_to}

Updates:
\${work_notes}
        `,
    subject: "Incident \${number} Update: \${short_description}",
    email_layout: incidentLayout.$id // Reference to the layout
  }
});
```

This example demonstrates:

1. Creating a custom layout with styled HTML
2. Using the layout in an email template
3. Providing both HTML and plain text versions
4. Using dynamic variables from the incident record
5. Proper styling and formatting for better readability
