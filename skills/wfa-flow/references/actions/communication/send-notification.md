# sendNotification Action

Send in-platform notification using pre-configured notification templates.

## When to Use

- Internal ServiceNow user notifications (preferred over email)
- Using existing notification templates for consistency
- Notifications requiring centralized template management
- Multi-channel delivery (email + SMS + push) via single template

## Best Practices

1. **Fetch Notification IDs Dynamically** - Use lookUpRecord on `sysevent_email_action` table by name, don't hardcode sys_ids:

```typescript
const notif = wfa.action(
  action.core.lookUpRecord,
  { $id: Now.ID["get_notification"] },
  {
    table: "sysevent_email_action",
    conditions: "name=incident.assigned"
  }
);

wfa.action(
  action.core.sendNotification,
  { $id: Now.ID["notify"] },
  {
    record: wfa.dataPill(_params.trigger.current, "reference"),
    notification: wfa.dataPill(notif.Record, "reference")
  }
);
```

2. **Leverage Existing Templates** - Use pre-configured templates for consistency. Templates define recipients, subject, body, and delivery channels.

3. **Link to Record Context** - Always set `record` parameter for notification variables and "View Record" links.

4. **Verify Template Exists** - Check notification exists before triggering (invalid refs fail silently).

## Important Notes

- **Recipients Defined in Template** - Cannot override recipients from flow. Recipients configured in notification record.
- **Content Defined in Template** - Subject/body pre-configured. Cannot customize from action.
- **Delivery Channels** - Email, SMS, push configured in template settings
- **Template Location** - System Policy > Email > Notifications
- **Fails Silently** - Invalid notification references don't throw errors

## sendNotification vs sendEmail

- **sendNotification** - Internal users, uses templates, in-platform (preferred)
- **sendEmail** - External recipients, custom HTML, off-platform

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_COMMUNICATION** knowledge source.
