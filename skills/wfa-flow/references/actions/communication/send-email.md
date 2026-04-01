# sendEmail Action

Send rich text emails to email addresses, user records, and group records.

## When to Use

- External recipient notifications (customers, vendors)
- Detailed reports or summaries requiring formatting
- Notifications requiring rich content (tables, formatted lists)
- Off-platform communication (recipients without ServiceNow access)

## Best Practices

1. **Prefer sendNotification for Internal Users** - In-platform notifications better for ServiceNow users (no email clutter, better tracking)

2. **Keep HTML Simple** - Use basic tags only (`<h2>`, `<p>`, `<strong>`, `<ul>`, `<li>`, `<br>`). Avoid CSS/JavaScript.

3. **Link to Records** - Always set `record` and `table_name` parameters for tracking and audit trails

4. **Validate Email Addresses** - Check users have populated email fields before sending. Handle missing emails gracefully.

5. **Avoid Email Loops** - Don't send bulk emails in forEach without batching. Aggregate items into single summary email.

## Important Notes

- **⚠️ CRITICAL - Email Body Constraint:** `ah_body` does NOT support data pills - use static strings only. Data pills CAN be used in `ah_subject` and `ah_to`.
- **Recipients Format** - Comma-separated: `"user1@example.com,user2@example.com"` or use data pills in `ah_to`
- **HTML Support** - Basic HTML only. Test in multiple clients (Outlook, Gmail, mobile)
- **Watermark** - Default adds "Sent by ServiceNow" footer. Set `watermark_email: false` for professional external emails
- **Rate Limits** - Sending 100+ individual emails can trigger spam filters
- **Email Tracking** - Emails logged in `sys_email` table and related record history

## sendEmail vs Other Methods

- **sendEmail** - External recipients, rich HTML formatting, off-platform
- **sendNotification** - Internal ServiceNow users, in-platform (preferred)
- **sendSms** - Critical alerts only ($$ cost)

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_COMMUNICATION** knowledge source.
