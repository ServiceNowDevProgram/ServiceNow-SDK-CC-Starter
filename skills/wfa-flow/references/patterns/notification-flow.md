# Notification Flow Pattern

Multi-channel communication strategies - Email, SMS, in-platform notifications.

## Table of Contents

- [When to Use](#when-to-use)
- [Pattern Structure](#pattern-structure)
- [Communication Channels](#communication-channels)
- [Example 1: Multi-Channel Critical Alert](#example-1-multi-channel-critical-alert)
- [Example 2: Status Change Notifications](#example-2-status-change-notifications)
- [Example 3: Notify Multiple Recipients](#example-3-notify-multiple-recipients)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Performance Tips](#performance-tips)

## When to Use

- Multi-recipient notifications
- Multi-channel notifications (email + SMS)
- Escalation notifications
- Team collaboration notifications

## Pattern Structure

```
Trigger → Lookup Recipients → Send Notifications (Email/SMS/Notification)
```

## Communication Channels

| Channel          | Best For                      | Cost | Target        |
| ---------------- | ----------------------------- | ---- | ------------- |
| sendEmail        | Detailed info, external users | Low  | Email inbox   |
| sendNotification | Internal ServiceNow users     | Free | ServiceNow UI |
| sendSMS          | Critical/urgent alerts        | High | Mobile device |

## Example 1: Multi-Channel Critical Alert

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["critical_incident_alert"],
    name: "Critical Incident Alert",
    description: "Sends email + SMS for P1 incidents"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["critical_incident"] },
    {
      table: "incident",
      condition: "priority=1^active=true",
      run_flow_in: "background"
    }
  ),

  _params => {
    const oncallEngineer = wfa.action(
      action.core.lookUpRecord,
      { $id: Now.ID["find_oncall"] },
      {
        table: "sys_user",
        conditions: "user_name=oncall.engineer^active=true"
      }
    );

    wfa.action(
      action.core.sendEmail,
      { $id: Now.ID["send_email"] },
      {
        ah_to: wfa.dataPill(oncallEngineer.Record.email, "string"),
        ah_subject: "CRITICAL: Incident requires immediate attention",
        ah_body: `
          <h2 style="color: red;">CRITICAL INCIDENT</h2>
          <p><strong>A critical incident has been created and requires immediate attention.</strong></p>
          <p>Please check the incident details in ServiceNow immediately.</p>
        `,
        record: wfa.dataPill(_params.trigger.current, "reference"),
        table_name: "incident"
      }
    );

    wfa.action(
      action.core.sendSms,
      { $id: Now.ID["send_sms"] },
      {
        recipients: "+1234567890",
        message: "CRITICAL INCIDENT: Check email immediately for details."
      }
    );

    wfa.action(
      action.core.sendNotification,
      { $id: Now.ID["send_notification"] },
      {
        notification: "incident.critical_assigned",
        record: wfa.dataPill(_params.trigger.current, "reference"),
        table_name: "incident"
      }
    );
  }
);
```

## Example 2: Status Change Notifications

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["status_change_notifications"],
    name: "Status Change Notifications",
    description: "Notify stakeholders based on status"
  },

  wfa.trigger(
    trigger.record.updated,
    { $id: Now.ID["incident_updated"] },
    {
      table: "incident",
      condition: "stateISNOTEMPTY",
      trigger_strategy: "unique_changes",
      run_flow_in: "background"
    }
  ),

  _params => {
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_resolved"],
        condition: `${wfa.dataPill(_params.trigger.current.state, "string")}=6`
      },
      () => {
        wfa.action(
          action.core.sendEmail,
          { $id: Now.ID["notify_resolved"] },
          {
            ah_to: wfa.dataPill(
              _params.trigger.current.caller_id.email,
              "string"
            ),
            ah_subject: "Your incident has been resolved",
            ah_body:
              "Your incident has been resolved. Please check ServiceNow for resolution details."
          }
        );
      }
    );

    wfa.flowLogic.elseIf(
      {
        $id: Now.ID["check_in_progress"],
        condition: `${wfa.dataPill(_params.trigger.current.state, "string")}=2`
      },
      () => {
        wfa.action(
          action.core.sendEmail,
          { $id: Now.ID["notify_progress"] },
          {
            ah_to: wfa.dataPill(
              _params.trigger.current.caller_id.email,
              "string"
            ),
            ah_subject: "Your incident is now in progress",
            ah_body:
              "Your incident is now being worked on. Please check ServiceNow for current status and assignment details."
          }
        );

        wfa.flowLogic.if(
          {
            $id: Now.ID["has_assignee"],
            condition: `${wfa.dataPill(_params.trigger.current.assigned_to, "string")}ISNOTEMPTY`
          },
          () => {
            wfa.action(
              action.core.sendNotification,
              { $id: Now.ID["notify_assignee"] },
              {
                notification: "incident.in_progress",
                record: wfa.dataPill(_params.trigger.current, "reference"),
                table_name: "incident"
              }
            );
          }
        );
      }
    );
  }
);
```

## Example 3: Notify Multiple Recipients

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["notify_team"],
    name: "Notify Team Members",
    description: "Sends notifications to team members"
  },

  wfa.trigger(
    trigger.record.updated,
    { $id: Now.ID["incident_updated"] },
    {
      table: "incident",
      condition: "state=2^active=true",
      trigger_strategy: "unique_changes",
      run_flow_in: "background"
    }
  ),

  _params => {
    const teamMembers = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["find_team"] },
      {
        table: "sys_user",
        conditions: "active=true^department=IT",
        max_results: 50
      }
    );

    wfa.flowLogic.forEach(
      wfa.dataPill(teamMembers.Records, "records"),
      { $id: Now.ID["loop_team"] },
      member => {
        wfa.action(
          action.core.sendEmail,
          { $id: Now.ID["notify_member"] },
          {
            ah_to: wfa.dataPill(member.email, "string"),
            ah_subject: "Incident update notification",
            ah_body:
              "An incident in your department has been updated. Please check ServiceNow for details.",
            record: wfa.dataPill(_params.trigger.current, "reference"),
            table_name: "incident"
          }
        );
      }
    );
  }
);
```

## Best Practices

1. **Channel Selection:**
   - Internal users → sendNotification (preferred, free)
   - External stakeholders → sendEmail
   - Critical/urgent → sendSMS (sparingly, high cost)

2. **Email Body Constraints:**
   - **IMPORTANT:** `ah_body` does NOT support dataPills - use static strings only
   - `ah_subject` and `ah_to` CAN use dataPills
   - Use `record` and `table_name` parameters to link to the record

3. **Batch Recipients:**

   ```typescript
   ah_to: `${email1},${email2},${email3}`; // Comma-separated, avoid loops
   ```

4. **SMS Messages:** Use static strings only (160 character limit)

5. **Respect Limits:** Keep forEach loops under 50 iterations

## Common Use Cases

- Critical incident alerts (email + SMS + notification)
- Status change notifications (resolved, closed, assigned)
- SLA escalation chains
- Group assignment notifications

## Performance Tips

1. **Batch recipients** using comma-separated lists instead of forEach loops
2. **Reserve SMS** for truly critical notifications (cost ~$0.01-0.05 per message)
3. **Limit iterations** - max 50-100 recipients per loop
4. **Reuse notification templates** when possible

## Next Steps

For Fluent API signatures, parameters, outputs, and complete coding examples, use `get_knowledge_source` tool to retrieve:

- **WFA_FLOW_TRIGGER_RECORD** - Record trigger API (created, updated, createdOrUpdated)
- **WFA_FLOW_ACTIONS_TABLE** - Table action APIs (lookUpRecord, lookUpRecords)
- **WFA_FLOW_ACTIONS_COMMUNICATION** - Communication action APIs (sendEmail, sendNotification, sendSms)
- **WFA_FLOW_LOGICS** - Flow logic API (forEach, if/elseIf/else) with condition syntax

These knowledge sources contain authoritative API documentation with all parameters, data types, and working examples.
