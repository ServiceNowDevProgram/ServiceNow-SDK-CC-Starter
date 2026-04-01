# Email Notification

The Email Notification API allows you to create email notifications [sysevent_email_action] that send automated emails based on database operations, custom events, or manual triggers.

## EmailNotification object

Configure an email notification to send automated email messages.

Properties

| Name                     | Type                | Description                                                                                                                                            |
| ------------------------ | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| $id                      | String or Number    | Optional. A unique ID for the metadata object. Format: `$id: Now.ID[<value>]` When you build the application, this ID is hashed into a unique sys_ID.  |
| table                    | String              | Required. The table to which the notification applies. Must be a valid ServiceNow table name.                                                          |
| name                     | String              | Optional. A descriptive name for the notification.                                                                                                     |
| description              | String              | Optional. A detailed description of the notification's purpose.                                                                                        |
| category                 | String or Reference | Optional. The sys_id of the notification category [sys_notification_category]. Default: `'c97d83137f4432005f58108c3ffa917a'` (default email category). |
| notificationType         | String              | Optional. The type of notification. Valid values: `'email'` (default), `'vcalendar'`. Note: VCalendar notifications cannot have digest configuration.  |
| active                   | Boolean             | Optional. Whether the notification is active. Default: `true`.                                                                                         |
| mandatory                | Boolean             | Optional. Whether users can opt out. Default: `false`.                                                                                                 |
| enableDynamicTranslation | Boolean             | Optional. Enable dynamic translation of content. Default: `false`.                                                                                     |
| triggerConditions        | Object              | Required. Defines when the notification is triggered. See Trigger Conditions.                                                                          |
| recipientDetails         | Object              | Optional. Defines who receives the notification. See Recipient Details.                                                                                |
| emailContent             | Object              | Optional. Defines the email content and formatting. See Email Content.                                                                                 |
| digest                   | Object              | Optional. Configures digest settings. Not available for VCalendar. See Digest Configuration.                                                           |

## Trigger Conditions

Defines when the notification is triggered. Structure varies based on `generationType`.

### Engine-Based (`generationType: 'engine'`)

Triggers on database operations (insert/update).

| Name              | Type    | Description                                                                                                                      |
| ----------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| generationType    | String  | Required. Must be `'engine'`.                                                                                                    |
| onRecordInsert    | Boolean | Optional. Trigger on record insert. Default: `false`. Note: At least one of `onRecordInsert` or `onRecordUpdate` must be `true`. |
| onRecordUpdate    | Boolean | Optional. Trigger on record update. Default: `false`. Note: At least one of `onRecordInsert` or `onRecordUpdate` must be `true`. |
| weight            | Number  | Optional. Priority weight (0-1000). Default: `0`.                                                                                |
| condition         | String  | Optional. Encoded query condition.                                                                                               |
| advancedCondition | String  | Optional. Server-side JavaScript for complex conditions.                                                                         |
| itemTable         | String  | Optional. Reference to item table.                                                                                               |
| item              | String  | Optional. Specific item field value.                                                                                             |
| order             | Number  | Optional. Execution order (0-9999). Default: `100`.                                                                              |

### Event-Based (`generationType: 'event'`)

Triggers on custom ServiceNow events.

| Name                 | Type   | Description                                                                                  |
| -------------------- | ------ | -------------------------------------------------------------------------------------------- |
| generationType       | String | Required. Must be `'event'`.                                                                 |
| eventName            | String | Required. The name of the custom event.                                                      |
| weight               | Number | Optional. Priority weight (0-1000). Default: `0`.                                            |
| condition            | String | Optional. Encoded query condition.                                                           |
| advancedCondition    | String | Optional. Server-side JavaScript for complex conditions.                                     |
| itemTable            | String | Optional. Reference to item table.                                                           |
| affectedFieldOnEvent | String | Optional. Which event parameter contains affected field. Valid values: `'parm1'`, `'parm2'`. |
| item                 | String | Optional. Specific item field value.                                                         |
| order                | Number | Optional. Execution order (0-9999). Default: `100`.                                          |

### Triggered (`generationType: 'triggered'`)

Manually triggered notifications.

| Name           | Type   | Description                      |
| -------------- | ------ | -------------------------------- |
| generationType | String | Required. Must be `'triggered'`. |

## Recipient Details

Defines who receives the notification.

### Common Properties (All Types)

| Name                     | Type             | Description                                                                      |
| ------------------------ | ---------------- | -------------------------------------------------------------------------------- |
| recipientUsers           | Array of Strings | Optional. Array of user sys_ids or email addresses.                              |
| recipientFields          | Array of Strings | Optional. Array of field names containing recipients (must be reference fields). |
| recipientGroups          | Array of Strings | Optional. Array of group sys_ids.                                                |
| excludeDelegates         | Boolean          | Optional. Exclude delegate users. Default: `false`.                              |
| isSubscribableByAllUsers | Boolean          | Optional. Allow all users to subscribe. Default: `false`.                        |
| sendToCreator            | Boolean          | Optional. Send to record creator. Default: `false`.                              |

### Event-Based Only

| Name                    | Type    | Description                                                                         |
| ----------------------- | ------- | ----------------------------------------------------------------------------------- |
| eventParm1WithRecipient | Boolean | Optional. Event parameter 1 contains recipient. Only for event-based notifications. |
| eventParm2WithRecipient | Boolean | Optional. Event parameter 2 contains recipient. Only for event-based notifications. |

## Email Content

Defines the email content and formatting.

### Common Properties

| Name               | Type             | Description                                                                                         |
| ------------------ | ---------------- | --------------------------------------------------------------------------------------------------- |
| contentType        | String           | Optional. Content type. Valid values: `'text/html'` (default), `'text/plain'`, `'multipart/mixed'`. |
| template           | String           | Optional. Email template sys_id [sysevent_email_template].                                          |
| style              | String           | Optional. Email style sys_id [sys_email_style].                                                     |
| subject            | String           | Optional. Email subject line.                                                                       |
| message            | String           | Optional. Message content (legacy field).                                                           |
| smsAlternate       | String           | Optional. SMS alternate message.                                                                    |
| importance         | String           | Optional. Importance level. Valid values: `'low'`, `'high'`.                                        |
| includeAttachments | Boolean          | Optional. Include attachments. Default: `false`.                                                    |
| omitWatermark      | Boolean          | Optional. Omit watermark. Default: `false`.                                                         |
| from               | String           | Optional. From email address.                                                                       |
| replyTo            | String           | Optional. Reply-to address.                                                                         |
| pushMessageOnly    | Boolean          | Optional. Push message only. Default: `false`.                                                      |
| pushMessageList    | Array of Strings | Optional. Push message sys_ids.                                                                     |
| accessRestriction  | String           | Optional. Access restriction sys_id [sys_email_access_restriction].                                 |
| emailScript        | String           | Optional. Email script sys_id [sys_email_script].                                                   |

### Content Type Specific

| Name        | Type   | Description                                                                                                                         |
| ----------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| messageHtml | String | Required for `'text/html'` and `'multipart/mixed'`. HTML message content. Use `\${variable}` format for variable references.        |
| messageText | String | Required for `'text/plain'` and `'multipart/mixed'`. Plain text message content. Use `\${variable}` format for variable references. |

## Digest Configuration

Configures digest settings for grouping notifications. Not available for VCalendar notifications.

| Name            | Type    | Description                                                                                                    |
| --------------- | ------- | -------------------------------------------------------------------------------------------------------------- |
| allow           | Boolean | Optional. Allow digest. Default: `false`. When `false`, digest fields are ignored.                             |
| default         | Boolean | Optional. Enable digest by default for users. Default: `false`.                                                |
| type            | String  | Optional. Digest type. Valid values: `'single'` (one email per interval), `'multiple'` (one email per record). |
| defaultInterval | String  | Optional. Digest interval sys_id [sys_email_digest_interval].                                                  |
| subject         | String  | Optional. Digest email subject. Use `\${variable}` format for variable references.                             |
| html            | String  | Optional. Digest HTML content. Use `\${variable}` format for variable references.                              |
| text            | String  | Optional. Digest plain text content. Use `\${variable}` format for variable references.                        |
| template        | String  | Optional. Digest template sys_id.                                                                              |
| separatorHtml   | String  | Optional. HTML separator between notifications. Use `\${variable}` format for variable references.             |
| separatorText   | String  | Optional. Plain text separator between notifications. Use `\${variable}` format for variable references.       |

## Email Notification Types and Use Cases

### 1. Engine-Based Notifications (generationType: 'engine')

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Incident Assigned Notification"],
  table: "incident",
  name: "Incident Assigned Notification",
  triggerConditions: {
    generationType: "engine",
    onRecordInsert: true,
    onRecordUpdate: true,
    condition: "assigned_toISNOTEMPTY^EQ",
    item: "event.parm1"
  },
  recipientDetails: {
    recipientFields: ["assigned_to"],
    excludeDelegates: false,
    isSubscribableByAllUsers: false,
    sendToCreator: true
  },
  emailContent: {
    subject: "Incident \${number} has been assigned to you",
    messageHtml: `<p>Hello \${assigned_to.name}</p><p>You have been assigned incident \${number}.</p><p>Short Description: \${short_description}</p>`,
    includeAttachments: false,
    omitWatermark: false,
    pushMessageOnly: false,
    forceDelivery: false
  }
});
```

### 2. Event-Based Notifications (generationType: 'event')

**How to get event name:** Query 'sysevent_register' and find the suitable event name for the table using `runQuery` tool

Call `run_query` tool:

- Table: `sysevent_register`
- Encoded Query: `table=table_name`

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Event-Based Notification"],
  table: "incident",
  name: "Event-Based Notification",
  triggerConditions: {
    generationType: "event",
    eventName: "incident.escalated",
    condition: "active=true^EQ",
    item: "event.parm1"
  },
  recipientDetails: {
    recipientFields: ["assignment_group", "assigned_to"],
    excludeDelegates: false,
    isSubscribableByAllUsers: false, // if true, might also set affaffectedFieldOnEvent field
    sendToCreator: true,
    eventParm1WithRecipient: false,
    eventParm2WithRecipient: false
  },
  emailContent: {
    subject: "Incident Escalated",
    messageHtml: `<p>An incident \${number} has been escalated.</p><p>Priority: \${priority}</p>`,
    includeAttachments: false,
    omitWatermark: false,
    pushMessageOnly: false,
    forceDelivery: false
  }
});
```

### 3. Triggered Notifications (generationType: 'triggered')

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Manual Incident Notification"],
  table: "incident",
  name: "Manual Incident Notification",
  triggerConditions: {
    generationType: "triggered"
  },
  recipientDetails: {
    recipientFields: ["assignment_group", "assigned_to"],
    excludeDelegates: false,
    isSubscribableByAllUsers: false,
    sendToCreator: true
  },
  emailContent: {
    subject: "Manual Notification",
    messageHtml: `<p>This is a manually triggered notification.</p><p>Incident: \${number}</p>`,
    includeAttachments: false,
    omitWatermark: false,
    pushMessageOnly: false,
    forceDelivery: false
  }
});
```

### 4. VCalendar Meeting Invitations (notificationType: 'vcalendar')

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["63640574ff79b610166dffffffffff61"],
  table: "incident",
  name: "VCalendar Meeting Invitation",
  notificationType: "vcalendar",
  triggerConditions: {
    generationType: "event",
    eventName: "incident.severity.1",
    condition: "active=true^EQ",
    item: "event.parm1"
  },
  recipientDetails: {
    recipientFields: ["assignment_group", "assigned_to"],
    excludeDelegates: false,
    isSubscribableByAllUsers: false,
    sendToCreator: true,
    eventParm1WithRecipient: false,
    eventParm2WithRecipient: false
  },
  emailContent: {
    messageHtml: `<div>
<div>You are invited to review incident \${number}</div>
</div>`,
    subject: "Incident \${number} Review Meeting",
    includeAttachments: false,
    omitWatermark: false,
    pushMessageOnly: false,
    forceDelivery: false
  }
});
```

## Email Content Type Patterns

### 1. HTML Content (contentType: 'text/html') - DEFAULT

- **ONLY create `messageHtml` field** - Do NOT create messageText
- Use minimal HTML by default (simple `<p>` and `<div>` tags)
- Add advanced styling ONLY if user explicitly requests it

**Implementation Example (Minimal HTML - DEFAULT):**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Weekly Incident Dashboard Report"],
  table: "incident",
  name: "Weekly Incident Dashboard Report",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ",
    item: "event.parm1"
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    subject: "Incident \${number} Review Meeting",
    messageHtml: `<p>Hello \${assigned_to.name}</p><p>You are invited to review incident \${number}.</p><p>Priority: \${priority}</p><p>Status: \${state}</p>`
  }
});
```

**Implementation Example (Advanced HTML - ONLY when explicitly requested):**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Styled Incident Notification"],
  table: "incident",
  name: "Styled Incident Notification",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "priority=1^EQ"
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    subject: "URGENT: Incident \${number}",
    messageHtml: `
            <div style="background-color: #f5f5f5; padding: 20px; font-family: Arial, sans-serif;">
                <div style="background-color: #d32f2f; color: white; padding: 15px; border-radius: 5px;">
                    <h2 style="margin: 0;">🚨 High Priority Incident</h2>
                </div>
                <div style="background-color: white; padding: 20px; margin-top: 10px; border-radius: 5px;">
                    <p><strong>Incident:</strong> \${number}</p>
                    <p><strong>Priority:</strong> \${priority}</p>
                    <p><strong>Description:</strong> \${short_description}</p>
                </div>
            </div>
        `
  }
});
```

### 2. Plain Text Content (contentType: 'text/plain')

- **ONLY create `messageText` field** - Do NOT create messageHtml
- Set `contentType: 'text/plain'` explicitly

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["System Monitoring Alert"],
  table: "incident",
  name: "System Monitoring Alert",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ",
    item: "event.parm1"
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    contentType: "text/plain",
    subject: "Incident \${number} Alert",
    messageText: `Hello \${assigned_to.name},\n\nIncident \${number} requires your attention.\nPriority: \${priority}\nDescription: \${short_description}\n\nPlease review and take action.`
  }
});
```

### 3. Multipart Content (contentType: 'multipart/mixed')

- **Create BOTH `messageHtml` AND `messageText` fields**
- Set `contentType: 'multipart/mixed'` explicitly
- Email clients will display HTML if supported, otherwise fall back to text

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ",
    item: "event.parm1"
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    contentType: "multipart/mixed",
    subject: "Incident \${number} Update",
    messageHtml: `<p>Hello \${assigned_to.name}</p><p>Incident \${number} has been updated.</p><p>Priority: \${priority}</p>`,
    messageText: `Hello \${assigned_to.name},\n\nIncident \${number} has been updated.\nPriority: \${priority}\n\nThank you.`
  }
});
```

## Recipient Configuration Patterns

### 1. Field-Based Recipients (recipientFields)

**How to get fields name:** Query 'sys_dictionary' for table to get field name that can be used in recipientFields

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    recipientFields: ["assignment_group", "assigned_to"]
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting"
  }
});
```

### 2. User-Based Recipients (recipientUsers)

**How to get user sys_id:** Query 'sys_user_list' using `runQuery` tool for specific user sys_ids

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    recipientUsers: ["admin_user_sys_id", "vip_user_sys_id"]
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting"
  }
});
```

### 4. Event Parameter Recipients

**Note:** eventParm1WithRecipient and eventParm2WithRecipient should be set to true only if it is confirmed that parm1_value or parm2_value are defined or not empty in the event.

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    eventParm1WithRecipient: true,
    eventParm2WithRecipient: true
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting"
  }
});
```

## Digest Configuration Use Cases

### 1. Multiple Digest Notifications (`digestType: 'multiple'`)

**How to get digest interval:** Query 'sys_email_digest_interval' using `runQuery` tool for existing digest interval sys_ids or create a new Email Digest Interval and get the sys_id.

Call `run_query` tool:

- Table: `sys_email_digest_interval`
- Encoded Query: `name=daily`

**How to get digest template:** Query 'sysevent_email_template' using `runQuery` tool for existing digest template sys_ids or create a new Email Template and get the sys_id.

Call `run_query` tool:

- Table: `sysevent_email_template`

**⚠️ IMPORTANT:** Digest notifications have built-in scheduling. Do NOT create separate business rules, script includes, or schedulers.

**How it works:**

- Each triggered notification becomes a separate entry in the digest
- Multiple incidents = multiple entries in one email
- Each entry uses the individual `messageHtml` template
- Entries are separated by `separatorHtml`

**Example: Multiple Incidents in One Digest Email:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["multiple_digest_config"],
  table: "incident",
  name: "Daily Multiple Incident Digest",
  description: "Sends one email with multiple incident entries",
  active: true,

  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    onRecordInsert: true,
    condition: "priority<=2^EQ" // High/Critical priority only
  },

  recipientDetails: {
    recipientFields: ["assignment_group.manager"]
  },

  // Template for EACH incident entry in the digest
  emailContent: {
    subject: "Incident \${number} Updated",
    messageHtml: `
<div style="border-bottom: 1px solid #ddd; padding: 10px;">
    <h4>\${number}: \${short_description}</h4>
    <p><strong>Priority:</strong> \${priority} | <strong>State:</strong> \${state}</p>
    <p><strong>Assigned:</strong> \${assigned_to}</p>
</div>
        `
  },

  digest: {
    allow: true,
    type: "multiple", // Each incident = separate entry
    defaultInterval: "daily",
    subject: "Daily High Priority Incidents - \${digest_count} incidents",
    html: "<h2>High Priority Incidents (\${digest_count} total)</h2>",
    separatorHtml: '<hr style="margin: 15px 0;">', // Separates each incident
    from: "incident-management@company.com" // only add if asked or email id provided
  }
});
```

### 2. Single Digest Notifications (`digestType: 'single'`)

**How it works:**

- All triggered notifications are combined into one single email
- Uses only the `digestHtml` template (ignores individual `messageHtml`)
- Perfect for summary reports rather than individual entries

**How to get digest interval:** Query 'sys_email_digest_interval' using `runQuery` tool for existing digest interval sys_ids or create a new Email Digest Interval and get the sys_id.

Call `run_query` tool:

- Table: `sys_email_digest_interval`
- Encoded Query: `name=daily`

**⚠️ CRITICAL: Always check for existing interval first before creating new ones. Do not create duplicate intervals.**

**How to get digest template:** Query 'sysevent_email_template' using `runQuery` tool for existing digest template sys_ids or create a new Email Template and get the sys_id.

Call `run_query` tool:

- Table: `sysevent_email_template`

**Example: Single Summary Email:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["single_digest_config"],
  table: "incident",
  name: "Daily Incident Summary",
  description: "Sends one summary email with incident statistics",
  active: true,

  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    onRecordInsert: true
  },

  recipientDetails: {
    recipientGroups: ["incident_management_team"]
  },

  // Individual email content (NOT used in single digest)
  emailContent: {
    subject: "Incident \${number} Updated",
    messageHtml: "<p>This template is ignored in single digest</p>"
  },

  digest: {
    allow: true,
    template: "digest_template_sys_id", // reference of the template
    type: "single", // One summary email only
    defaultInterval: "daily",
    subject: "Daily Incident Summary - \${digest_count} incidents processed",
    // This template is used for the ENTIRE digest email
    html: `
<div style="font-family: Arial, sans-serif;">
    <h2>Daily Incident Summary</h2>
    <div style="background-color: #f8f9fa; padding: 20px; border-radius: 5px;">
        <h3>📊 Today's Statistics</h3>
        <p><strong>Total Incidents Processed:</strong> \${digest_count}</p>
        <p><strong>Date:</strong> \${digest_date}</p>
        <p>Please check the incident dashboard for detailed information.</p>
    </div>
</div>
        `,
    from: "incident-reports@company.com" // only add if asked to send from a specific email
  }
});
```

## Advanced Configuration Patterns

### 1. Category Configuration

**How to get category sys_ids:** Query 'sys_notification_category' using `runQuery` tool for existing category sys_ids or create a new Notification Category and get the sys_id.

Call `run_query` tool:

- Table: `sys_notification_category`

**⚠️ CRITICAL: Always query for existing category first before creating new ones:**

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";
EmailNotification({
  $id: Now.ID["Category Configuration"],
  table: "incident",
  name: "Incident Notification",
  category: "category_sys_id",
  description: "Notification for incident updates",
  active: true,
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true
  },
  recipientDetails: {
    recipientFields: ["assignment_group", "assigned_to"]
  },
  emailContent: {
    subject: "Incident Update",
    messageHtml: "<p>Incident \${number} has been updated.</p>"
  },
  category: "incident_management"
});
```

### 2. Template Configuration

**⚠️ CRITICAL: Do NOT create email templates unless the user explicitly requests them:**

**Default Approach (NO Template):**

- Create ONLY `subject` and `messageHtml` (or `messageText`) directly in the `emailContent` section
- This is simpler and more maintainable for most use cases
- Templates should be reserved for reusable content across multiple notifications

**When to ADD a template:**

- User explicitly asks to "create a template", "use a template", "with template", or "reusable template"
- User needs to share the same email format across multiple notifications
- User specifically mentions branding or corporate template requirements

**How to get existing template sys_ids:** Query 'sysevent_email_template' using `runQuery` tool for existing template sys_ids

Call `run_query` tool:

- Table: `sysevent_email_template`

**⚠️ CRITICAL: Always query for existing templates first before creating new ones:**

**How to get style sys_ids:** Query 'sys_email_style' using `runQuery` tool for existing style sys_ids or create a new Email Style and get the sys_id.

Call `run_query` tool:

- Table: `sysevent_email_style`

**Implementation Example (WITHOUT Template - DEFAULT):**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Incident Notification No Template"],
  table: "incident",
  name: "Incident Notification",
  description: "Notification for incident updates",
  active: true,
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    subject: "Incident \${number} Update",
    messageHtml: `<p>Hello \${assigned_to.name}</p><p>Incident \${number} has been updated.</p><p>Priority: \${priority}</p>`
  }
});
```

**Implementation Example (WITH Template - ONLY when explicitly requested):**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Template Configuration"],
  table: "incident",
  name: "Incident Notification",
  description: "Notification for incident updates",
  active: true,
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true
  },
  recipientDetails: {
    recipientFields: ["assignment_group", "assigned_to"]
  },
  emailContent: {
    template: "corporate_template_sys_id",
    style: "corporate_style_sys_id",
    subject: "Incident Update",
    messageHtml: "<p>Incident \${number} has been updated.</p>"
  }
});
```

### 3. Conditional Notifications

**When to use:** Smart filtering based on conditions

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    eventParm1WithRecipient: true,
    eventParm2WithRecipient: true
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting"
  }
});
```

### 4. SMS Alternate Notification

**When to use:** Combine email with other notification methods

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    eventParm1WithRecipient: true,
    eventParm2WithRecipient: true
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting",
    smsAlternate: "Critical alert: \${short_description}"
  }
});
```

### 5. Push Notifications and dynamic translation Configuration

**When to use:** Send push notifications to users

**How to get push message list ids sys_ids:** Query 'sys_push_notif_msg' using `runQuery` tool for existing push message sys_ids. **Note**: Creation of new `sys_push_notif_msg` records is not supported - you can only reference existing push notification messages.

Call `run_query` tool:

- Table: `sys_push_notif_msg`

**Implementation Example:**

```typescript
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  enableDynamicTranslation: true,
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    eventParm1WithRecipient: true,
    eventParm2WithRecipient: true
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting",
    pushMessageList: ["mobile_push_msg_sys_id"],
    pushMessageOnly: false
  }
});
```

### 6. Importance and Include attachments Configuration

**When to use:** Set the importance of the email high or low, All the record attachments to be included in the email

**Implementation Example:**

```typescript
EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    eventParm1WithRecipient: true,
    eventParm2WithRecipient: true
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting",
    smsAlternate: "Critical alert: \${short_description}",
    importance: "high",
    includeAttachments: true
  }
});
```

## 7. Omit Watermark Configuration

**When to use:** Omit the servicenow branding from the email

**Implementation Example:**

```typescript
EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    eventParm1WithRecipient: true,
    eventParm2WithRecipient: true
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting",
    smsAlternate: "Critical alert: \${short_description}",
    importance: "high",
    includeAttachments: true,
    omitWatermark: true
  }
});
```

### 8. Reply To and From Configuration

**When to use:** Set the reply to and from email address

**Implementation Example:**

```typescript
EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    eventParm1WithRecipient: true,
    eventParm2WithRecipient: true
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting",
    smsAlternate: "Critical alert: \${short_description}",
    replyTo: "admin@servicenow.com", //optional, should be a valid email address
    from: "maint@servicenow.com" //optional, should be a valid email address
  }
});
```

### 9. Force Delivery Configuration

**When to use:** Force delivery of the email even if user preference blocks it

**Implementation Example:**

```typescript
EmailNotification({
  $id: Now.ID["Customer Service Ticket Updates"],
  table: "incident",
  name: "Customer Service Ticket Updates",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    condition: "active=true^EQ"
  },
  recipientDetails: {
    eventParm1WithRecipient: true,
    eventParm2WithRecipient: true
  },
  emailContent: {
    messageHtml: `<div><div>You are invited to review incident \${number}</div></div>`,
    subject: "Incident \${number} Review Meeting",
    smsAlternate: "Critical alert: \${short_description}",
    forceDelivery: true
  }
});
```

### 10. Multichannel Notification Configuration

**When to use:** Send notifications across multiple channels (email, SMS, push) for critical alerts

**Implementation Example:**

```typescript
EmailNotification({
  $id: Now.ID["Critical Incident Multichannel Alert"],
  table: "incident",
  name: "Critical Incident Multichannel Alert",
  mandatory: true, // Users cannot opt out of critical notifications
  triggerConditions: {
    generationType: "engine",
    onRecordInsert: true,
    onRecordUpdate: true,
    condition: "priority=1^ORpriority=2^state=1" // P1/P2 incidents that are new
  },
  recipientDetails: {
    recipientFields: ["assigned_to", "caller_id"],
    recipientGroups: [
      "incident_managers_group_sys_id",
      "on_call_engineers_group_sys_id"
    ]
  },
  emailContent: {
    contentType: "text/html",
    subject: "CRITICAL: Incident \${number} - \${short_description}",
    messageHtml: `
            <div style="background-color: #ff4444; color: white; padding: 10px; border-radius: 5px;">
                <h2>🚨 CRITICAL INCIDENT ALERT</h2>
                <p><strong>Incident:</strong> \${number}</p>
                <p><strong>Priority:</strong> \${priority}</p>
                <p><strong>Description:</strong> \${short_description}</p>
                <p><strong>Assigned To:</strong> \${assigned_to.name}</p>
                <p><strong>Created:</strong> \${sys_created_on}</p>
                <p><a href="\${instance_url}/incident.do?sys_id=\${sys_id}" style="color: #ffffff; text-decoration: underline;">View Incident</a></p>
            </div>
        `,
    smsAlternate:
      "CRITICAL: Incident \${number} - \${short_description}. Priority: \${priority}. Assigned: \${assigned_to.name}",
    pushMessageList: ["mobile_push_notification_sys_id"], // Push notification configuration
    forceDelivery: true, // Bypass user notification preferences
    importance: "high"
  }
});
```

## Complete Implementation Example with Access Restrictions

This comprehensive example demonstrates how to create a complete email notification system with access restrictions, including all supporting components.

### Implementation Example

```typescript
import { EmailNotification, Record, Now } from "@servicenow/sdk/core";

// 1. Create notification category
const incidentCategory = Record({
  $id: Now.ID["incident_escalation_category"],
  table: "sys_notification_category",
  data: {
    name: "Incident Escalations",
    description: "Notifications for incident escalations and priority changes"
  }
});

// 2. Create email template
const escalationTemplate = Record({
  $id: Now.ID["incident_escalation_template"],
  table: "sysevent_email_template",
  data: {
    name: "Incident Escalation Template",
    subject: "URGENT: Incident \${number} requires immediate attention",
    body: `
<div style="font-family: Arial, sans-serif; max-width: 600px;">
    <div style="background-color: #d32f2f; color: white; padding: 15px; border-radius: 5px 5px 0 0;">
        <h2 style="margin: 0;">🚨 High Priority Incident Escalation</h2>
    </div>
    
    <div style="background-color: #f5f5f5; padding: 20px; border-radius: 0 0 5px 5px;">
        
        <div style="margin-top: 20px; padding: 15px; background-color: #fff3cd; border-left: 4px solid #ffc107; border-radius: 3px;">
            <strong>⚠️ Action Required:</strong>
            <p style="margin: 5px 0 0 0;">This incident requires immediate attention due to its high priority status. Please review and take appropriate action.</p>
        </div>
        
        <div style="margin-top: 20px; text-align: center;">
            <div style="background-color: #1976d2; color: white; padding: 12px 24px; text-decoration: none; border-radius: 5px; display: inline-block;">View Incident Details</div>
        </div>
    </div>
</div>
        `,
    content_type: "text/html"
  }
});

// 3. Create email style
const escalationStyle = Record({
  $id: Now.ID["incident_escalation_style"],
  table: "sysevent_email_style",
  data: {
    name: "Incident Escalation Style",
    body_color: "#ffffff",
    font_color: "#333333",
    font_family: "Arial, sans-serif",
    font_size: "14px",
    border_color: "#d32f2f",
    link_color: "#1976d2"
  }
});

// 4. Create digest interval for high-volume scenarios
const hourlyDigest = Record({
  $id: Now.ID["incident_hourly_digest"],
  table: "sys_email_digest_interval",
  data: {
    name: "Hourly Incident Digest",
    interval: "1970-01-01 01:00:00" // 1 hour
  }
});

// 5. Create the main email notification
const incidentEscalationNotification = EmailNotification({
  $id: Now.ID["incident_escalation_notification"],
  table: "incident",
  name: "High Priority Incident Escalation",
  description:
    "Notifies IT Support team when high-priority incidents are updated",
  category: incidentCategory,
  notificationType: "email",
  active: true,
  mandatory: false,
  enableDynamicTranslation: true,

  // Trigger on incident updates
  triggerConditions: {
    generationType: "engine",
    onRecordInsert: false,
    onRecordUpdate: true,
    weight: 100,
    condition: "priority=1^ORpriority=2^state!=6^state!=7", // High/Critical priority, not resolved/closed
    order: 100
  },

  // Send to IT Support group and assigned user
  recipientDetails: {
    recipientGroups: ["d625dccec0a8016700a222a0f7900d06"], // IT Support group sys_id
    recipientFields: ["assigned_to", "caller_id"],
    sendToCreator: false,
    isSubscribableByAllUsers: false
  },

  // HTML email content
  emailContent: {
    contentType: "text/html",
    template: escalationTemplate, // reference of the template
    style: escalationStyle,
    from: "noreply@company.com",
    replyTo: "itsupport@company.com"
  },

  // Enable digest for high-volume scenarios
  digest: {
    allow: true,
    defaultInterval: hourlyDigest,
    template: escalationTemplate,
    subject:
      "Incident Escalation Digest - \${digest_count} incidents require attention",
    html: `<div>
<div>Incident Escalation Digest</div>
</div>`,
    separatorHtml: '<hr style="margin: 15px 0;">',
    from: "noreply@company.com",
    replyTo: "itsupport@company.com"
  }
});

// 6. Create access restrictions
// Restriction 2: Only priority 1 and 2 incidents
const activeSupportRestriction = Record({
  $id: Now.ID["incident_active_support_restriction"],
  table: "email_access_restriction",
  data: {
    notification: incidentEscalationNotification,
    conditions: "priority=1^ORpriority=2^EQ",
    description: "Restrict to only priority 1 and 2 incidents"
  }
});
```
