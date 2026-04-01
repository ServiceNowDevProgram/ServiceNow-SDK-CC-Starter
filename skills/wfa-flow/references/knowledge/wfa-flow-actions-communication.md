# WFA_FLOW_ACTIONS_COMMUNICATION

# Workflow Automation Flow Actions - Communication

The Communication Actions API reference provides complete technical specifications for sending notifications via email, in-platform notifications, and SMS. This document contains action signatures, parameter definitions, types, and syntax examples. For conceptual guidance, when-to-use recommendations, and best practices, refer to skill references.

---

## action.core.sendEmail

Send rich text emails to a comma separated list of email addresses, user records, and group records. If user records do not have an email address configured, the email will not be sent to that user. Use pills to decorate the subject line and email body. Note: Commas are not required between pills, only static email addresses.

### Input Parameters

| Parameter       | Type      | Default | Mandatory | Description                                                                                                                                                                                              |
| --------------- | --------- | ------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ah_to           | string    | -       | Yes       | Email recipient(s) - comma-separated list of email addresses, user references, or group references                                                                                                       |
| ah_subject      | string    | -       | Yes       | Email subject line                                                                                                                                                                                       |
| ah_body         | html      | -       | No        | Email body content (supports HTML formatting). **IMPORTANT:** ah_body only accepts static strings - data pills are NOT supported. Use string concatenation or template literals to embed dynamic values. |
| record          | reference | -       | No        | Related record sys_id for tracking and association                                                                                                                                                       |
| table_name      | string    | -       | No        | Table name of the related record                                                                                                                                                                         |
| ah_cc           | string    | -       | No        | CC recipients - comma-separated list                                                                                                                                                                     |
| ah_bcc          | string    | -       | No        | BCC recipients - comma-separated list                                                                                                                                                                    |
| watermark_email | boolean   | true    | No        | Include "Sent by ServiceNow" watermark footer                                                                                                                                                            |

### Output Fields

| Field | Type      | Description                                        |
| ----- | --------- | -------------------------------------------------- |
| email | reference | sys_id of the sent email record in sys_email table |

### Usage Examples

**Basic Email Notification with Static Body:**

```typescript
wfa.action(
  action.core.sendEmail,
  { $id: Now.ID["notify_user"] },
  {
    ah_to: wfa.dataPill(_params.trigger.current.assigned_to.email, "string"),
    ah_subject: `Incident ${wfa.dataPill(_params.trigger.current.number, "string")} assigned to you`,
    ah_body:
      "A new incident has been assigned to you. Please review the details in your queue.",
    record: wfa.dataPill(_params.trigger.current, "reference"),
    table_name: "incident"
  }
);
```

**Note:** ah_subject supports data pills, but ah_body only accepts static strings. For dynamic email content, use `action.core.sendNotification` with a notification template instead.

**Email to Multiple Recipients with CC/BCC:**

```typescript
wfa.action(
  action.core.sendEmail,
  { $id: Now.ID["notify_team"] },
  {
    ah_to: "user1@example.com,user2@example.com",
    ah_cc: wfa.dataPill(_params.trigger.current.manager.email, "string"),
    ah_subject: "Team Notification",
    ah_body: "This is a team-wide notification about an important update.",
    watermark_email: false
  }
);
```

**HTML Formatted Email with Static Body:**

```typescript
wfa.action(
  action.core.sendEmail,
  { $id: Now.ID["formatted_email"] },
  {
    ah_to: wfa.dataPill(_params.trigger.current.opened_by.email, "string"),
    ah_subject: "Incident Report",
    ah_body: `
      <h2>Incident Details</h2>
      <p>An incident has been created and requires your attention.</p>
      <p>Please check your ServiceNow dashboard for complete details.</p>
    `,
    record: wfa.dataPill(_params.trigger.current, "reference"),
    table_name: "incident"
  }
);
```

**Alternative for Dynamic Content:** Use `action.core.sendNotification` with a notification template that supports dynamic field substitution.

---

## action.core.sendNotification

Send a notification in one or more formats as specified by a notification record. The notification record you select determines the notification format and recipients.

### Input Parameters

| Parameter    | Type      | Default | Mandatory | Description                                                                            |
| ------------ | --------- | ------- | --------- | -------------------------------------------------------------------------------------- |
| notification | reference | -       | Yes       | The notification template to trigger (sys_id or name from sysevent_email_action table) |
| record       | reference | -       | No        | The record to associate with the notification for context                              |
| table_name   | string    | -       | No        | The table name of the associated record                                                |

### Output Fields

None - this action does not return any output fields.

### Usage Examples

**Trigger Incident Assignment Notification:**

```typescript
wfa.action(
  action.core.sendNotification,
  { $id: Now.ID["notify_assignment"] },
  {
    notification: "incident.assigned",
    record: wfa.dataPill(_params.trigger.current, "reference"),
    table_name: "incident"
  }
);
```

**Use Notification sys_id:**

```typescript
wfa.action(
  action.core.sendNotification,
  { $id: Now.ID["notify_change_approved"] },
  {
    notification: "5e7f8c9d2b3a4e5f6a7b8c9d0e1f2a3b",
    record: wfa.dataPill(_params.trigger.current, "reference"),
    table_name: "change_request"
  }
);
```

**Trigger Notification Without Record Context:**

```typescript
wfa.action(
  action.core.sendNotification,
  { $id: Now.ID["general_alert"] },
  {
    notification: "system_maintenance_alert"
  }
);
```

---

## action.core.sendSms

Send SMS to user records and group records using email-based SMS. If user records do not have an SMS device configured, an SMS will not be sent to that user.

### Input Parameters

| Parameter  | Type   | Default | Mandatory | Description                                                                      |
| ---------- | ------ | ------- | --------- | -------------------------------------------------------------------------------- |
| recipients | string | -       | Yes       | The recipient's phone number (E.164 format recommended: +[country code][number]) |
| message    | string | -       | Yes       | The SMS message content (plain text)                                             |

### Output Fields

| Field | Type      | Description                                                     |
| ----- | --------- | --------------------------------------------------------------- |
| email | reference | sys_id of the sent email record (SMS is sent via email gateway) |

### Usage Examples

**Send Critical Alert to On-Call Engineer:**

```typescript
wfa.action(
  action.core.sendSms,
  { $id: Now.ID["alert_oncall"] },
  {
    recipients: `${wfa.dataPill(
      _params.trigger.current.assigned_to.mobile_phone,
      "string"
    )}`,
    message: `URGENT: Critical incident ${wfa.dataPill(_params.trigger.current.number, "integer")} requires immediate attention`
  }
);
```

**Send SMS with Static Phone Number:**

```typescript
wfa.action(
  action.core.sendSms,
  { $id: Now.ID["notify_manager"] },
  {
    recipients: "+14155551234",
    message: "Production outage detected. Check monitoring dashboard."
  }
);
```

**Send SMS to Multiple Recipients:**

```typescript
wfa.action(
  action.core.sendSms,
  { $id: Now.ID["alert_team"] },
  {
    recipients: "+14155551234,+14155555678",
    message: `P1 incident ${wfa.dataPill(_params.trigger.current.number, "integer")} created. Immediate action required.`
  }
);
```

---
