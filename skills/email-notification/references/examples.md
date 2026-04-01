# Email Notification Examples

This document contains concrete examples of email notification scenarios.

## Contents

- [Example 1: Simple Insert/Update Notification](#example-1-simple-insertupdate-notification)
- [Example 2: Insert Only Notification](#example-3-notification-with-template-explicitly-requested)
- [Example 3: Digest Notification (Explicitly Requested)](#example-4-digest-notification-explicitly-requested)
- [Example 4: Event-Based Notification](#example-5-event-based-notification)
- [Example 5: Triggered Notification](#example-6-triggered-notification)
- [Example 6: Plain Text Notification](#example-7-plain-text-notification)
- [Example 7: Multipart Notification (HTML + Text)](#example-8-multipart-notification-html--text)
- [Example 8: Notification with Advanced Features](#example-9-notification-with-advanced-features)
- [Example 9: VCalendar Meeting Invitation](#example-10-vcalendar-meeting-invitation)
- [Example 10: Notification with Email Script](#example-11-notification-with-email-script)
- [Example 11: Notification with Email Style](#example-12-notification-with-email-style)
- [Example 12: Notification with Access Restriction](#example-13-notification-with-access-restriction)
- [Example 13: Notification with Category](#example-14-notification-with-category)
- [Example 14: Complete Notification with All Advanced Features](#example-15-complete-notification-with-all-advanced-features)

## Example 1: Simple Insert/Update Notification

**User Request:** "Create a table called 'my_requests' and send email when record is inserted or updated"
**Key Principle:** Use ONE notification with both insert and update flags instead of creating separate notifications.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Incident Notification No Template"],
  table: "incident",
  name: "Incident Notification",
  description: "Notification for incident updates",
  active: true,
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true,
    onRecordInsert: true
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

## Example 2 Notification with Template (Explicitly Requested)

**User Request:** "Create email notification with template for incident updates"
**Key Principle:** Create templates ONLY when explicitly requested. User said "with template" so we create one.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification, Record, Now } from "@servicenow/sdk/core";

// Create email template
const incidentTemplate = Record({
  $id: Now.ID["incident_update_template"],
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

// Create notification with template reference
EmailNotification({
  $id: Now.ID["Incident Notification With Template"],
  table: "incident",
  name: "Incident Notification With Template",
  description: "Notification using template for incident updates",
  active: true,
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    template: incidentTemplate
  }
});
```

## Example 3: Digest Notification (Explicitly Requested)

**User Request:** "Send daily digest email for new incidents"
**Key Principle:** Add digest ONLY when explicitly requested. User said "daily digest" so we add digest configuration.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Daily Incident Digest"],
  table: "incident",
  name: "Daily Incident Digest",
  description: "Daily digest of new incidents",
  active: true,
  triggerConditions: {
    generationType: "engine",
    onRecordInsert: true
  },
  recipientDetails: {
    recipientFields: ["assignment_group"]
  },
  emailContent: {
    subject: "Incident \${number} Created",
    messageHtml: `<div style="border-bottom: 1px solid #ddd; padding: 10px;">
      <h4>\${number}: \${short_description}</h4>
      <p><strong>Priority:</strong> \${priority} | <strong>State:</strong> \${state}</p>
    </div>`
  },
  digest: {
    allow: true,
    type: "multiple",
    defaultInterval: "daily_digest_sys_id", // System ID for daily digest
    subject: "Daily New Incidents - \${digest_count} incidents",
    html: "<h2>New Incidents (\${digest_count} total)</h2>",
    separatorHtml: '<hr style="margin: 15px 0;">'
  }
});
```

## Example 4: Event-Based Notification

**User Request:** "Send email when incident is escalated"
**Key Principle:** Use event-based notifications for custom events, not database operations.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Incident Escalation Notification"],
  table: "incident",
  name: "Incident Escalation Notification",
  triggerConditions: {
    generationType: "event",
    eventName: "incident.escalated",
    condition: "active=true^EQ"
  },
  recipientDetails: {
    recipientFields: ["assignment_group", "assigned_to"],
    sendToCreator: true
  },
  emailContent: {
    subject: "Incident \${number} Escalated",
    messageHtml: `<p>Incident \${number} has been escalated.</p><p>Priority: \${priority}</p><p>Please take immediate action.</p>`
  }
});
```

## Example 5: Triggered Notification

**User Request:** "Create manual notification for incidents"
**Key Principle:** Triggered notifications are manually invoked, not automatic.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Manual Incident Notification"],
  table: "incident",
  name: "Manual Incident Notification",
  triggerConditions: {
    generationType: "triggered"
  },
  recipientDetails: {
    recipientFields: ["assigned_to"],
    sendToCreator: true
  },
  emailContent: {
    subject: "Manual Notification - Incident \${number}",
    messageHtml: `<p>This is a manually triggered notification.</p><p>Incident: \${number}</p><p>Status: \${state}</p>`
  }
});
```

## Example 6: Plain Text Notification

**User Request:** "Send plain text email for system alerts"
**Key Principle:** Use plain text for simple, system-level notifications.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["System Alert Plain Text"],
  table: "incident",
  name: "System Alert Plain Text",
  triggerConditions: {
    generationType: "engine",
    onRecordInsert: true,
    condition: "priority=1^EQ"
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    contentType: "text/plain",
    subject: "ALERT: Incident \${number}",
    messageText: `Hello \${assigned_to.name},\n\nIncident \${number} requires attention.\nPriority: \${priority}\nDescription: \${short_description}\n\nPlease review immediately.`
  }
});
```

## Example 7: Multipart Notification (HTML + Text)

**User Request:** "Send email with both HTML and text versions"
**Key Principle:** Multipart provides fallback for email clients that don't support HTML.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Multipart Incident Notification"],
  table: "incident",
  name: "Multipart Incident Notification",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    contentType: "multipart/mixed",
    subject: "Incident \${number} Update",
    messageHtml: `<p>Hello \${assigned_to.name}</p><p>Incident \${number} has been updated.</p><p><strong>Priority:</strong> \${priority}</p>`,
    messageText: `Hello \${assigned_to.name},\n\nIncident \${number} has been updated.\nPriority: \${priority}\n\nThank you.`
  }
});
```

## Example 8: Notification with Advanced Features

**User Request:** "Send high priority email with attachments and SMS"
**Key Principle:** Combine features in ONE notification when requested.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["High Priority With Features"],
  table: "incident",
  name: "High Priority With Features",
  triggerConditions: {
    generationType: "engine",
    onRecordInsert: true,
    condition: "priority=1^EQ"
  },
  recipientDetails: {
    recipientFields: ["assigned_to", "caller_id"]
  },
  emailContent: {
    subject: "URGENT: Incident \${number}",
    messageHtml: `<div style="background-color: #ff4444; color: white; padding: 15px;">
      <h2>🚨 CRITICAL INCIDENT</h2>
      <p><strong>Incident:</strong> \${number}</p>
      <p><strong>Description:</strong> \${short_description}</p>
    </div>`,
    importance: "high",
    includeAttachments: true,
    smsAlternate: "URGENT: Incident \${number} - \${short_description}"
  }
});
```

## Example 9: VCalendar Meeting Invitation

**User Request:** "Send calendar invitation for incident review"
**Key Principle:** VCalendar is for meeting invitations, not regular notifications.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification } from "@servicenow/sdk/core";

EmailNotification({
  $id: Now.ID["Incident Review Meeting"],
  table: "incident",
  name: "Incident Review Meeting",
  notificationType: "vcalendar",
  triggerConditions: {
    generationType: "event",
    eventName: "incident.severity.1",
    condition: "active=true^EQ"
  },
  recipientDetails: {
    recipientFields: ["assignment_group", "assigned_to"]
  },
  emailContent: {
    subject: "Incident \${number} Review Meeting",
    messageHtml: `<div>
      <div>You are invited to review incident \${number}</div>
      <p><strong>Priority:</strong> \${priority}</p>
      <p><strong>Time:</strong> As scheduled</p>
    </div>`
  }
});
```

## Example 10: Notification with Email Script

**User Request:** "Create notification with custom script to format incident details"
**Key Principle:** Use email scripts ONLY when complex logic or dynamic content generation is needed.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification, Record, Now } from "@servicenow/sdk/core";

// Create email script for dynamic content
const incidentScript = Record({
  $id: Now.ID["incident_details_script"],
  table: "sys_script_email",
  data: {
    name: "Format Incident Details",
    new_lines_to_html: false,
    script: `(function runMailScript(/* GlideRecord */ current, /* TemplatePrinter */ template,
      /* Optional EmailOutbound */ email, /* Optional GlideRecord */ email_action,
      /* Optional GlideRecord */ event) {
      // Add custom logic here
      template.print("Incident: " + current.number);
      template.print("\\nPriority: " + current.getDisplayValue('priority'));
      template.print("\\nStatus: " + current.getDisplayValue('state'));
      template.print("\\nAssigned to: " + current.getDisplayValue('assigned_to'));
      
      // Add parent incident info if exists
      if (current.parent) {
        template.print("\\nParent Status: " + current.getDisplayValue('parent.state'));
      }
})(current, template, email, email_action, event);`
  }
});

EmailNotification({
  $id: Now.ID["Incident With Script"],
  table: "incident",
  name: "Incident With Script",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    subject: "Incident \${number} Update",
    messageHtml: `<p>Hello \${assigned_to.name}</p><p>Incident details:</p>\${mail_script:Format Incident Details}`
  }
});
```

## Example 11: Notification with Email Style

**User Request:** "Create styled notification with branded email design"
**Key Principle:** Use email styles ONLY when consistent branding or complex CSS is explicitly requested.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification, Record, Now } from "@servicenow/sdk/core";

// Create email style for branding
const corporateStyle = Record({
  $id: Now.ID["corporate_email_style"],
  table: "sysevent_email_style",
  data: {
    name: "Corporate Email Style",
    style: `
      <p><span style="font-size: 14pt; color: rgb(230, 126, 35); background-color: rgb(251, 238, 184);"><strong><em>Hello World</strong></span></p>
    `
  }
});

// Create notification using the style
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
      <div class="email-container">
        <div class="header">
          <h1>🚨 High Priority Incident</h1>
        </div>
        <div class="content">
          <p><strong>Incident:</strong> \${number}</p>
          <p><span class="priority-high">Priority: \${priority}</span></p>
          <p><strong>Description:</strong> \${short_description}</p>
          <a href="\${instance_url}/incident.do?sys_id=\${sys_id}" class="action-button">View Incident</a>
        </div>
      </div>
    `,
    style: corporateStyle
  }
});
```

## Example 12: Notification with Access Restriction

**User Request:** "Create notification that only sends to active users for high priority incidents"
**Key Principle:** Use ONE access restriction per notification. Combine multiple conditions using AND (^) or OR (^OR) operators.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification, Record, Now } from "@servicenow/sdk/core";

// Create notification
const restrictedNotification = EmailNotification({
  $id: Now.ID["restricted_incident_notification"],
  table: "incident",
  name: "Restricted Incident Notification",
  triggerConditions: {
    generationType: "engine",
    onRecordUpdate: true
  },
  recipientDetails: {
    recipientFields: ["assigned_to"]
  },
  emailContent: {
    subject: "Incident \${number} Update",
    messageHtml: `<p>Incident \${number} has been updated.</p><p>Priority: \${priority}</p>`
  }
});

// Create access restriction (ONLY ONE per notification)
const accessRestriction = Record({
  $id: Now.ID["incident_access_restriction"],
  table: "email_access_restriction",
  data: {
    notification: restrictedNotification.$id,
    conditions: "priority=1^ORpriority=2^state!=6^state!=7^EQ", // High/Critical priority, not resolved/closed
    description: "Only send for P1/P2 incidents that are not resolved or closed"
  }
});
```

## Example 13: Notification with Category

**User Request:** "Create incident notification in Security category"
**Key Principle:** Create categories ONLY when organizing notifications by type or department. Always query for existing categories first.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification, Record, Now } from "@servicenow/sdk/core";

// Create notification category
const securityCategory = Record({
  $id: Now.ID["security_notifications_category"],
  table: "sys_notification_category",
  data: {
    name: "Security Notifications",
    description: "All security-related incident notifications"
  }
});

// Create notification with category
EmailNotification({
  $id: Now.ID["Security Incident Notification"],
  table: "incident",
  name: "Security Incident Notification",
  category: securityCategory,
  triggerConditions: {
    generationType: "engine",
    onRecordInsert: true,
    condition: "category.name=Security^EQ"
  },
  recipientDetails: {
    recipientFields: ["assignment_group"],
    recipientGroups: ["security_team_sys_id"]
  },
  emailContent: {
    subject: "Security Incident \${number} Created",
    messageHtml: `<p>A new security incident has been created.</p><p>Incident: \${number}</p><p>Description: \${short_description}</p>`
  }
});
```

## Example 14: Complete Notification with All Advanced Features

**User Request:** "Create complete notification system with category, style, script, and access restriction"
**Key Principle:** Create ALL components ONLY when explicitly requested for complex requirements.

**Implementation Example:**

```typescript
import "@servicenow/sdk/global";
import { EmailNotification, Record, Now } from "@servicenow/sdk/core";

// 1. Create category
const criticalCategory = Record({
  $id: Now.ID["critical_incidents_category"],
  table: "sys_notification_category",
  data: {
    name: "Critical Incidents",
    description: "Critical priority incident notifications"
  }
});

// 2. Create email style
const criticalStyle = Record({
  $id: Now.ID["critical_incident_style"],
  table: "sysevent_email_style",
  data: {
    name: "Critical Incident Style",
    style: `
      .critical-header {
        background: linear-gradient(135deg, #dc3545, #c82333);
        color: white;
        padding: 25px;
        text-align: center;
        border-radius: 8px 8px 0 0;
      }
      .critical-content {
        background-color: white;
        padding: 20px;
        border-left: 5px solid #dc3545;
      }
      .urgent-badge {
        background-color: #dc3545;
        color: white;
        padding: 8px 16px;
        border-radius: 20px;
        font-weight: bold;
        display: inline-block;
      }
    `
  }
});

// 3. Create email script
const criticalScript = Record({
  $id: Now.ID["critical_incident_script"],
  table: "sys_script_email",
  data: {
    name: "Critical Incident Details Script",
    new_lines_to_html: true,
    script: `(function runMailScript(/* GlideRecord */ current, /* TemplatePrinter */ template,
      /* Optional EmailOutbound */ email, /* Optional GlideRecord */ email_action,
      /* Optional GlideRecord */ event) {
      // Calculate time since creation
      var created = new GlideDateTime(current.sys_created_on);
      var now = new GlideDateTime();
      var duration = GlideDateTime.subtract(created, now);
      
      template.print("Time since creation: " + duration.getDisplayValue());
      template.print("\\nAssigned to: " + current.getDisplayValue('assigned_to'));
      template.print("\\nAssignment Group: " + current.getDisplayValue('assignment_group'));
      
      // Add escalation info if needed
      if (current.escalation > 0) {
        template.print("\\n⚠️ ESCALATED " + current.escalation + " times");
      }
})(current, template, email, email_action, event);`
  }
});

// 4. Create notification
const criticalNotification = EmailNotification({
  $id: Now.ID["critical_incident_notification"],
  table: "incident",
  name: "Critical Incident Notification",
  category: criticalCategory,
  mandatory: true,
  triggerConditions: {
    generationType: "engine",
    onRecordInsert: true,
    onRecordUpdate: true,
    condition: "priority=1^EQ"
  },
  recipientDetails: {
    recipientFields: ["assigned_to", "caller_id"],
    recipientGroups: ["incident_managers_sys_id"],
    sendToCreator: false
  },
  emailContent: {
    contentType: "text/html",
    subject: "🚨 CRITICAL: Incident \${number}",
    messageHtml: `
      <div class="critical-header">
        <h1>🚨 CRITICAL INCIDENT ALERT</h1>
        <span class="urgent-badge">IMMEDIATE ACTION REQUIRED</span>
      </div>
      <div class="critical-content">
        <p><strong>Incident:</strong> \${number}</p>
        <p><strong>Priority:</strong> \${priority}</p>
        <p><strong>Description:</strong> \${short_description}</p>
        <hr>
        <p><strong>Additional Details:</strong></p>
      </div>
    `,
    style: criticalStyle,
    emailScript: criticalScript,
    importance: "high",
    includeAttachments: true,
    forceDelivery: true
  }
});

// 5. Create access restriction
const criticalRestriction = Record({
  $id: Now.ID["critical_incident_restriction"],
  table: "email_access_restriction",
  data: {
    notification: criticalNotification.$id,
    conditions: "priority=1^state!=6^state!=7^EQ",
    description: "Only P1 incidents that are not resolved or closed"
  }
});
```
