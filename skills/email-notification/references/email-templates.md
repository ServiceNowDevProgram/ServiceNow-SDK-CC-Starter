# Email Templates

This document contains details about Email Templates for reusable notification content.

## Contents

- Overview
- When to Use Templates
- Decision Tree
- Trigger Phrases
- Template vs Inline Content
- Email Layouts
- Important Notes
- Next Steps

## Overview

Email templates provide reusable email content structures that can be referenced across multiple notifications. Templates define subject, message content, and formatting once and can be used by many notifications.

**Important:** Only create templates when explicitly requested by the user.

## When to Use Templates

### Create Templates When User Requests:

- ✅ "create a template"
- ✅ "use a template"
- ✅ "reusable template"
- ✅ "with template"
- ✅ Multiple notifications need same formatting

### Don't Create Templates By Default:

- ❌ Simple one-off notifications
- ❌ User doesn't mention templates
- ❌ Single notification use case
- ❌ Default approach

## Decision Tree: Should I Create Templates?

Use this decision tree to determine if you should create email templates:

- **If user says**: "send email" → **NO template** (use inline content)
- **If user says**: "create notification" → **NO template** (use inline content)
- **If user says**: "create email with template" → **YES, create template**
- **If user says**: "reusable email template" → **YES, create template**
- **If user says**: "use a template" → **YES, create template**
- **If user says**: "branded emails" → **YES, create template** (with layout)

## Trigger Phrases

Create templates ONLY when user uses these specific phrases:

### Templates Requested:

- "create a template"
- "reusable template"
- "email template"
- "use a template"
- "with template"
- "template for emails"

### Templates NOT Requested:

- "send email" (use inline content)
- "create notification" (use inline content)
- "email when..." (use inline content)

## Template vs Inline Content

### Inline Content (Default Approach)

Create content directly in the notification without templates. This is the default and preferred approach for most notifications.

**When to Use:**

- Simple one-off notifications
- User doesn't mention templates
- Single notification use case
- Quick implementation

### Templates (When Requested)

Create reusable templates that can be referenced by multiple notifications.

**When to Use:**

- User explicitly requests templates
- Multiple notifications need same formatting
- Reusable content structures
- Consistent branding across notifications

## Email Layouts

Email layouts provide consistent styling across multiple templates. Layouts define the overall structure (header, footer, container) while templates define the content.

**When to Use:**

- Advanced styling needed
- Consistent branding across multiple notifications
- User requests styled or branded emails

**Note:** Only create layouts when user explicitly requests styled or branded emails.

## Important Notes

- **Default**: Create content inline without templates
- **Templates**: Only create when user explicitly requests
- Templates use Record API with table `sysevent_email_template`
- Layouts use Record API with table `sys_email_layout`
- Templates support both HTML and plain text versions
- Variable substitution uses `\${variable}` syntax
- Don't create templates for one-off notifications

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the EMAIL_TEMPLATE knowledge source.
- For email styles, see `email-styles.md`
- For content types, see `content-types.md`
