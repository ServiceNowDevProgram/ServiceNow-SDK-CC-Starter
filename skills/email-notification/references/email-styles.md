# Email Styles

This document contains details about CSS styling for HTML Email Notifications.

## Contents

- Overview
- When to Use Styles
- Decision Tree
- Trigger Phrases
- Email Client Compatibility
- Important Notes
- Next Steps

## Overview

Email styles provide reusable CSS styling for HTML email notifications, enabling consistent branding and formatting across multiple notifications.

**Important:** Only use styles when user explicitly requests styled, branded, or formatted emails.

## When to Use Styles

### Use Styles When User Requests

- ✅ "styled emails"
- ✅ "branded notifications"
- ✅ "formatted emails"
- ✅ "with company branding"
- ✅ "custom styling"

### Don't Use Styles By Default

- ❌ Simple notifications
- ❌ User doesn't mention styling
- ❌ Default approach

## Decision Tree: Should I Add Styles?

Use this decision tree to determine if you should add email styles:

- **If user says**: "send email" → **NO styles** (minimal HTML)
- **If user says**: "create notification" → **NO styles** (minimal HTML)
- **If user says**: "styled email" → **YES, add styles**
- **If user says**: "branded email" → **YES, add styles**
- **If user says**: "formatted email" → **YES, add styles**
- **If user says**: "professional formatting" → **YES, add styles**

## Trigger Phrases

Add styles ONLY when user uses these specific ask to add styles phrases.

**Important:** Use inline styles wherever possible for maximum email client compatibility. Inline styles have the highest priority and are most widely supported across all email clients.

### Styles Requested

- "styled emails"
- "branded"
- "professional formatting"
- "custom styling"
- "with company branding"
- "formatted emails"
- "CSS styling"

### Styles NOT Requested

- "send email" (use minimal HTML)
- "create notification" (use minimal HTML)
- "email when..." (use minimal HTML)

## Email Client Compatibility

### Supported CSS Properties

Most email clients support:

- ✅ Colors (color, background-color)
- ✅ Fonts (font-family, font-size, font-weight)
- ✅ Spacing (padding, margin)
- ✅ Borders (border, border-radius)
- ✅ Text alignment (text-align)
- ✅ Display properties (display: block, inline-block)

### Limited Support

Some email clients have limited support for:

- ⚠️ Flexbox
- ⚠️ Grid layouts
- ⚠️ Advanced selectors
- ⚠️ Pseudo-classes (:hover may not work)
- ⚠️ Media queries

### Best Practices for Compatibility

- Use inline styles for critical styling
- Use tables for layout (better compatibility)
- Test in multiple email clients
- Provide fallbacks for unsupported features
- Keep CSS simple and straightforward

## Important Notes

- **Default**: No styles, simple HTML only
- **Styled**: Only when user explicitly requests
- Styles use Record API with table `sys_email_style`
- Use simple, compatible CSS
- Test in multiple email clients
- Don't rely solely on CSS for critical information
- Ensure content is readable without CSS

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the EMAIL_STYLE knowledge source.
- For email templates, see `email-templates.md`
- For content types, see `content-types.md`
