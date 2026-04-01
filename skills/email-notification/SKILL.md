---
name: email-notification
description: Guide for creating ServiceNow Email Notifications [sysevent_email_action] to send automated emails based on database operations, custom events, or manual triggers. Use when user mentions email notifications, alerts, or automated emails, calendar invite. Business rules are not needed to trigger notifications - the notification engine handles triggering automatically. Only supported in SDK 4.3.0 or higher
user-invocable: true
---
> [!NOTE]
> This skill was authored for an agent runtime that provides two tools not
> available here. When you encounter references to these tools, resolve them
> as follows:
>
> **`get_knowledge_source`** — When instructed to use this tool with a
> knowledge source name like `BUSINESS_RULE`, read the file at
> `references/knowledge/<name>.md` where `<name>` is the lowercase,
> hyphenated version (e.g. `BUSINESS_RULE` →
> `references/knowledge/business-rule.md`).
>
> **`load_skill_resource`** — When instructed to use this tool with a file
> path like `references/column.md`, read that file directly from the
> `references/` directory within this skill.


# Email Notifications

## When to use this skill

- When user explicitly asks for email notifications or automated emails
- When creating or modifying email notifications for database operations (insert/update)
- When implementing event-based email notifications triggered by custom events
- When setting up manual/on-demand notifications triggered by scripts
- When configuring recipient lists based on users, groups, or record fields
- When implementing calendar invitations (vCalendar notifications)
- When user explicitly requests digest notifications or email templates (advanced features)

## Core Principles (CRITICAL)

1. **API Documentation First**: **ALWAYS** use `get_knowledge_source` tool to retrieve the appropriate knowledge source BEFORE creating any records:
   - Email Notifications: `EMAIL_NOTIFICATION` knowledge source
   - Email Templates: `EMAIL_TEMPLATE` knowledge source
   - Email Styles: `EMAIL_STYLE` knowledge source
   - Email Access Restrictions: `EMAIL_ACCESS_RESTRICTION` knowledge source
   - Email Scripts: `EMAIL_SCRIPT` knowledge source
   - Email Digest Intervals: `EMAIL_DIGEST_INTERVAL` knowledge source
   - Notification Categories: `NOTIFICATION_CATEGORY` knowledge source
   - Follow the exact API structure, field names, and validation rules from the documentation. Never assume field names or structure.

2. **ONE Notification Rule**: Create **ONE notification** per request unless explicitly asked for multiple. Use both `onRecordInsert: true` AND `onRecordUpdate: true` in ONE notification instead of separate notifications if user requests for create/insert and update.

3. **Simplicity First**: Default to inline content with minimal HTML (`<p>` and `<div>` tags only). NO templates, NO digest, NO styles unless explicitly requested.

4. **Template Order**: If user requests templates, create template FIRST, then reference it. Never create inline content then retrofit.

5. **Field Validation**: Only use actual table fields (e.g., `${field_name}`). Email fields (`replyTo`, `from`) must be plain strings, never template variables.

## Quick Decision Guide

**How Many Notifications?**

- "send email on insert/update" → ONE notification with both flags
- "when created **or** updated" / "when created **and** updated" → ONE notification with both flags
- "multiple notifications" → Ask: "How many and for what purposes?"

**Should I Use Templates?**

- Default: NO (use inline content)
- Only if explicitly requested: "template", "reusable content"
- See `references/email-templates.md`

**Should I Add Digest?**

- Default: NO
- Only if explicitly requested: "daily digest", "group emails"
- See `references/digest-configuration.md`

**Should I Add Styles?**

- Default: NO (minimal HTML only)
- Only if explicitly requested: "styled", "branded", "formatted"
- See `references/email-styles.md`

## Request Classification

- **SIMPLE** ("send email when...") → ONE notification, inline content, no advanced features
- **COMPREHENSIVE** ("email system", "multiple notifications") → Ask clarifying questions first
- **UNCLEAR** → Ask: "How many notifications?", "Need templates?", "Group into digests?"

## Example: What NOT to Do

**User:** "Send email when incident is created"

❌ **WRONG**: 5 notifications, 3 templates, digest, styles
✅ **CORRECT**: 1 notification with `onRecordInsert: true`, inline content

See `references/examples.md` for more examples.

## Key Concepts

### Trigger Types

**Engine-Based** (`generationType: 'engine'`): Auto-triggers on insert/update. **DO NOT create business rules to trigger the notification** - Email_notification skill handles triggering automatically via ServiceNow's notification engine. Most common type for standard CRUD operations.

**Event-Based** (`generationType: 'event'`): Triggers on custom ServiceNow events. **Might requires business rules** to create the event. Use for custom workflows with complex logic.

**Triggered** (`generationType: 'triggered'`): Manual/script-triggered. **Requires explicit triggering logic** via scripts or business rules. Use for on-demand notifications with maximum control.

#### Decision Guide for Trigger Types

**Choose Engine-Based When:**

- Notification should fire on record insert/update
- No custom business logic needed
- Standard CRUD operation monitoring
- No business rules should be created

**Choose Event-Based When:**

- Custom workflow events trigger notification
- Complex business logic determines when to send
- Event parameters contain recipient information
- Business rules to create event, not required if using existing events

**Choose Triggered When:**

- Manual/on-demand sending required
- Script-controlled timing needed
- Maximum control over when notification sends
- Explicit triggering logic will be implemented

### Content Types

**HTML** (`'text/html'`, default): Rich emails with minimal HTML (`<p>`, `<div>`). Only create `messageHtml` field.

**Plain Text** (`'text/plain'`): Simple text-only for maximum compatibility or SMS. Only create `messageText` field.

**Multipart** (`'multipart/mixed'`): Both HTML and plain text versions for best compatibility. Must create both `messageHtml` AND `messageText` fields.

#### Variable Syntax (CRITICAL)

When writing TypeScript code, use **single backslash** to escape variables in template literals:

```typescript
// ✅ CORRECT - Single backslash in TypeScript code
messageHtml: `<p>Hello \${assigned_to.name}</p><p>Incident \${number}</p>`;

// ❌ WRONG - Double backslash in TypeScript code
messageHtml: `<p>Hello \\${assigned_to.name}</p><p>Incident \\${number}</p>`;

// ❌ WRONG - No backslash in TypeScript code
messageHtml: `<p>Hello ${assigned_to.name}</p><p>Incident ${number}</p>`;
```

**Remember:** In TypeScript template literals (backticks), write `\${field_name}` with **ONE backslash**. This escapes the dollar sign so JavaScript doesn't try to interpolate it, producing the runtime string `${field_name}` which ServiceNow will then substitute.

#### Decision Guide for Content Types

**Choose HTML When:**

- Rich formatting needed
- Modern email clients
- Images or links required
- Default choice for most notifications
- Only `messageHtml` should be created

**Choose Plain Text When:**

- Maximum compatibility required
- Simple text-only content
- SMS notifications
- Minimal formatting needed
- Only `messageText` should be created

**Choose Multipart When:**

- Professional emails
- Support all email clients
- Best user experience desired
- Both HTML and plain text needed
- Both `messageHtml` AND `messageText` must be created

### Recipients

**User Recipients**: Array of user sys_ids for fixed lists of specific individuals.

**Group Recipients**: Array of group sys_ids for teams/departments (preferred over individual users for scalability).

**Field Recipients**: Dynamic recipients from record fields (e.g., `assigned_to`, `caller`) - most common and flexible pattern.

**Special Recipients**:

- `sendToCreator: true` - Send to record creator (confirmations/acknowledgments)
- `eventParm1WithRecipient` / `eventParm2WithRecipient` - Event-based only, for custom workflows
- `excludeDelegates: true` - Exclude out-of-office delegates
- `isSubscribableByAllUsers: true` - Allow user subscriptions

#### Decision Guide for Recipients

**Choose User Recipients When:**

- Specific named individuals must always be notified
- Fixed recipient list required
- Small number of recipients

**Avoid User Recipients When:**

- Large or changing teams

**Choose Group Recipients When:**

- Notifying entire teams or departments
- Flexible membership management needed
- Scalable recipient lists
- Preferred over individual users for teams

**Choose Field Recipients When:**

- Dynamic recipients based on record data
- Notify assigned user, caller, or other references
- Most flexible approach
- Most common pattern

**Choose Special Recipients When:**

- Send to creator: Confirmation or acknowledgment
- Event parameters: Custom workflow recipients
- Exclude delegates: Direct notifications only
- User subscription: Optional notifications

### Notification Categories

Organize notifications into logical groups for management and user subscription. See `references/notification-categories.md`.

## Common Mistakes to Avoid

❌ **Do not use incorrect $id syntax** - always use `$id: Now.ID["value"]`

```typescript
// ✅ CORRECT
$id: Now.ID["email-notification"];

// ❌ WRONG
$id: "email-notification";
```

❌ **Don't use double backslash in TypeScript code** - In template literals, write `\${field_name}` (ONE backslash) to produce the runtime string `${field_name}`. Never write `\\${field_name}` (two backslashes) which produces the literal string `\${field_name}` that ServiceNow cannot substitute. See "Variable Syntax (CRITICAL)" section for examples.

❌ **Don't hallucinate fields** - only use actual table fields in message content (e.g., `${field_name}`), never invent fields

✅ **Do refer to knowledge sources and examples** - always check knowledge sources and `references/examples.md` before creating notifications to ensure correct syntax and patterns

✅ **Do always import SDK global** - always include `import "@servicenow/sdk/global"` at the top of your code

## Critical Implementation Rules

### 1. One Notification Rule (CRITICAL)

- **Default**: Create **ONE notification** per user request
- **Insert/Update**: Use **ONE notification** with both `onRecordInsert: true` AND `onRecordUpdate: true`
- **Multiple**: Only create multiple if user explicitly requests (e.g., "create 3 different notifications for different purposes")
- **Never**: Create separate notifications for insert vs update unless explicitly requested

### 2. Simplicity Rule (CRITICAL)

- **Default**: Inline content (NO templates, NO digest, NO styles)
- **Templates**: Only create when user explicitly asks for "template" or "reusable content"
- **Digest**: Only add when user explicitly requests "digest" or "grouped emails"
- **Styles**: Only add when user requests "styled", "branded", or "formatted" emails

### 3. Business Rules (CRITICAL)

- **Engine-based** (`generationType: 'engine'`) → **DO NOT create business rules** - Automatically triggered
- **Event-based** (`generationType: 'event'`) → **might need business rules** to fire the event not to trigger the notification
- **Triggered** (`generationType: 'triggered'`) → **Typically need business rules** or scripts to trigger

### 4. Content Type Fields (CRITICAL)

Only create fields needed for the specific content type:

- HTML → ONLY `messageHtml`
- Text → ONLY `messageText`
- Multipart → BOTH `messageHtml` AND `messageText`

**Field References:**

- Only use actual table fields in message content. Never hallucinate fields that don't exist in the table.
- In TypeScript code, write `\${field_name}` (ONE backslash in template literals) to produce the runtime string `${field_name}` which ServiceNow substitutes.

**Email Address Fields (CRITICAL):**

- `replyTo` and `from` fields are **OPTIONAL** - only add if explicitly requested or provided by user
- **Never assume or auto-populate** these fields with default email addresses
- **Ask the user** if these fields are needed when not specified
- When provided, these fields must be **plain static email strings** (e.g., `"hr-no-reply@company.com"`)
- **Never use template variables** like `\${hr_coordinator.email}` in `replyTo` or `from` fields
- These fields require static, valid email addresses when included

### 5. GUID Validation

All reference fields must contain valid 32-character hexadecimal sys_ids:

- `category` → `sys_notification_category`
- `emailContent.template` → `sysevent_email_template`
- `emailContent.style` → `sys_email_style`
- `recipientDetails.recipientUsers` → `sys_user`
- `recipientDetails.recipientGroups` → `sys_user_group`
- `digest.defaultInterval` → `sys_email_digest_interval`

### 6. Default Values

- `category`: Defaults to `'c97d83137f4432005f58108c3ffa917a'` (default email category)
- `contentType`: Defaults to `'text/html'`
- `generationType`: Defaults to `'engine'`
- `notificationType`: Defaults to `'email'`
- `active`: Defaults to `true`

### 7. Template Implementation Order (CRITICAL)

**When user requests template:**

- ✅ Create template FIRST using Record API (`sysevent_email_template`)
- ✅ Reference template in notification (`emailContent.template`)
- ✅ ONLY set `template` field - do NOT add `subject`, `messageHtml`, `messageText` (template overrides these)
- ❌ Never create inline content first then retrofit template

## Advanced Features

**Important**: These are advanced features. Only use when explicitly requested by the user.

### Email Templates

**Only create when user explicitly requests templates.**

Reusable email content structures:

- Define subject, message, and formatting once
- Reference across multiple notifications
- Support both HTML and plain text
- Can include email layouts for consistent styling

Check `references/email-templates.md` for detailed patterns.

### Digest Configuration

**Only add when user explicitly requests digest functionality.**

Digest allows grouping multiple notifications into a single email:

- **Single Digest**: One email per digest interval
- **Multiple Digest**: Separate emails per record
- **Intervals**: Daily, weekly, custom intervals
- **Not available for vCalendar notifications**

Check `references/digest-configuration.md` for detailed patterns.

### Email Styles

**Only add when user requests "styled", "branded", or "formatted" emails.**

CSS styling for HTML emails:

- Define reusable styles
- Apply consistent branding
- Support responsive design

Check `references/email-styles.md` for detailed patterns.

### Access Restrictions

Control who can see notification content:

- Script-based access control
- Conditional visibility
- Security-sensitive notifications

Check `references/email-access-restrictions.md` for detailed patterns.

### Email Scripts

Server-side scripting for dynamic content:

- Custom logic in notifications
- Dynamic recipient calculation
- Conditional content generation

Check `references/email-scripts.md` for detailed patterns.

### Digest Intervals

Configure how often digest emails are sent:

- Predefined intervals (daily, weekly)
- Custom intervals
- User preferences

Check `references/digest-intervals.md` for detailed patterns.

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the EMAIL_NOTIFICATION knowledge source
- For concrete examples, see `references/examples.md`
- For digest configuration, see `references/digest-configuration.md`
- For email templates, see `references/email-templates.md`
- For notification categories, see `references/notification-categories.md`
- For email styles, see `references/email-styles.md`
- For access restrictions, see `references/email-access-restrictions.md`
- For email scripts, see `references/email-scripts.md`
- For digest intervals, see `references/digest-intervals.md`
- Use `load_skill_resource` with skill name "email-notification" and the file path to get references
- For foundational Fluent DSL guidance (file structure, imports, usage patterns), use `get_knowledge_source` with **GENERAL**.
