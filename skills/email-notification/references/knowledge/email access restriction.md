# Email Access Restriction

# ServiceNow Email Access Restriction - Record API Reference

Create email access restrictions using the Record API to define conditional access controls for email notifications.

## Data Properties

| Field          | Type         | Required | Default | Description                                                           |
| -------------- | ------------ | -------- | ------- | --------------------------------------------------------------------- |
| `notification` | `Reference`  | Yes      | -       | Reference to sysevent_email_action record                             |
| `conditions`   | `Conditions` | Yes      | -       | Conditions that control access to the notification (max length: 8000) |
| `description`  | `String`     | No       | -       | Description of the access restriction (max length: 1000)              |

**Correct Conditions Examples by Table:**

```javascript
// For INCIDENT table notifications - use incident fields
"state=6^EQ"; // Only for resolved incidents
"priority=1^EQ"; // Only for P1 incidents
"assignment_group=IT Support^EQ"; // Only IT Support group

// For SYS_USER table notifications - use sys_user fields
"active=true^EQ"; // Only active users
"roles=admin^EQ"; // Only users with admin role
"department.name=IT^EQ"; // Only IT department users
"u_on_leave!=true^EQ"; // Exclude users on leave
"sys_id=javascript:gs.getUserID()^EQ"; // Only assigned user

// For TASK table notifications - use task fields
"state=3^EQ"; // Only for work in progress tasks
```

### Example: Query-First Pattern

**Step 1: Query Notification Id from sysevent_email_action table using runQuery tool:**

`table`: sysevent_email_action,
`encodedQuery`: name=<notification_name>^table=<table_name>

**Step 2: Query existing email access restrictions for the notification:**

`table`: email_access_restriction,
`encodedQuery`: notification=<notification_id>

**Step 3: Create only if doesn't exist:**

**Important Note: Do not use Now.ID for notification id:**

**Valid Notification Id example:**
'1234567890abcdef1234567890abcdef',
incidentNotification.$id

**Invalid Notification Id example:**
Now.ID['incident_escalation_restriction']

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";
import { incidentNotification } from "./incidentEscalationNotification";

const incidentEscalationRestriction = Record({
  $id: Now.ID["incident_escalation_restriction"],
  table: "email_access_restriction",
  data: {
    notification: incidentNotification.$id, // or Reference to notification queried in step 1, do not use Now.ID
    conditions: "priority=1^state!=6^EQ", // Only P1 incidents that are not resolved (using incident table fields)
    description:
      "Incident Escalation Access - Only for P1 incidents that are not resolved"
  }
});
```
