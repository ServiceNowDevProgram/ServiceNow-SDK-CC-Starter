# Email Access Restrictions

This document contains details about access restrictions for Email Notifications.

## Contents

- Overview
- When to Use
- Script Requirements
- Important Notes
- Next Steps

## Overview

Email access restrictions provide script-based access control for email notifications, allowing you to conditionally show or hide notification content based on user permissions, roles, or custom logic.

## When to Use

### Use Access Restrictions When

- ✅ Security-sensitive notifications
- ✅ Conditional visibility required
- ✅ Role-based content access
- ✅ Privacy requirements
- ✅ Compliance needs

### Don't Use By Default

- ❌ Standard notifications
- ❌ Public information
- ❌ No security requirements

## Script Requirements

The script must set the `answer` variable:

- `answer = true` - Allow access to notification content
- `answer = false` - Deny access to notification content

Available variables in script:

- `current` - Current record
- `event` - Event object (for event-based notifications)
- `gs` - GlideSystem object
- `email` - Email object

## What Happens When Access is Denied

When `answer = false`:

- User receives notification email
- Content is replaced with generic message
- Subject line may still be visible
- User knows notification was sent but cannot see details

## Important Notes

- Access restrictions use Record API with table `sys_email_access_restriction`
- Script must always set `answer` variable
- Use for security-sensitive content only
- Don't expose sensitive data in subject lines
- Keep subjects generic when using restrictions
- Combine with recipient configuration for additional protection
- Test with different user roles
- Keep restriction logic simple and clear

## Usage Guidelines for Email Access Restriction

### Pre-Creation Validation

Before creating any email access restriction, you **MUST** first query to check if a restriction already exists for the specified notification to prevent errors and conflicts.

### Getting the Notification ID

**Do not use `Now.ID` for the 'notification' field.** To get the correct notification ID:

1. Query the notification ID from the `sysevent_email_action` table using the `runQuery` tool
2. If the notification is created using a plugin, import the notification if it exists in a different file

### Critical Rule

**Do not create multiple restrictions for the same notification:**

Only one email access restriction is allowed per notification. If you need multiple conditions, combine them using AND (`^`) or OR (`^OR`) operators in the conditions field.

### Condition Field Matching

Condition fields must match the table that the notification is configured for:

- **Incident notifications**: Use incident table fields (e.g., `state`, `priority`, `assigned_to`, `caller_id`)
- **User notifications**: Use sys_user table fields (e.g., `active`, `roles`, `department`, `manager`)
- **Task notifications**: Use task table fields (e.g., `state`, `assigned_to`, `assignment_group`)
- **Any other notification type**: Use the corresponding table fields
- **condition should always be in the format**: field_name OPERATOR value (e.g., `state = 1` or `priority > 3`)

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the EMAIL_ACCESS_RESTRICTION knowledge source.
- For email scripts, see `email-scripts.md`
- For recipient configuration, see `recipient-configuration.md`
