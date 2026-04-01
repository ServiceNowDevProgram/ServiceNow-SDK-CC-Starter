# sendSms Action

Send SMS to users via email-based SMS gateway. Only works if users have SMS device configured.

## When to Use

- Critical incident alerts (P1/P0 requiring immediate response)
- On-call notifications (urgent issues needing immediate action)
- Production system outages (business-critical failures)
- SLA breach escalations (time-sensitive alerts)

## Best Practices

1. **Reserve for Critical Only** - SMS incurs provider costs ($0.01-0.05/message). Use only for urgent, time-sensitive situations.

2. **E.164 Phone Format** - Format: `+[country code][number]` (e.g., `+14155551234`). Remove spaces, dashes, parentheses.

3. **Character Limit** - 160 characters max. Lead with critical info: incident number, severity, action required.

4. **Verify Configuration** - Check users have `mobile_phone` populated before sending. Handle missing numbers gracefully.

5. **Implement Deduplication** - Track sent messages to prevent spam. Add throttling (e.g., max 1 SMS per incident per hour).

## Important Notes

- SMS delivery can fail silently - always combine with email/notifications
- Check sys_email table for send status
- Test SMS gateway configuration in dev/test first
- Consider time zones and business hours (avoid non-emergency SMS outside business hours)

## sendSms vs Other Methods

- **SMS** - Critical alerts requiring immediate attention ($$ cost)
- **Email** - Detailed communication, non-urgent updates (free)
- **Notification** - In-platform alerts, user online (free)

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_ACTIONS_COMMUNICATION** knowledge source.
