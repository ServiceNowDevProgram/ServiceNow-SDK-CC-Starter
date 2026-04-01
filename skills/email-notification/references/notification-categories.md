# Notification Categories

This document contains details about organizing Email Notifications using categories.

## Contents

- Overview
- When to Use Categories
- Category Organization Patterns
- User Subscription Benefits
- Important Notes
- Next Steps

## Overview

Notification categories organize email notifications into logical groups, enabling better management, user subscription preferences, and administrative control.

## When to Use Categories

### Use Categories When:

- ✅ Organizing notifications into logical groups
- ✅ Implementing notification preferences
- ✅ Allowing users to subscribe to specific types
- ✅ Managing large numbers of notifications
- ✅ Providing user control over notification types

### Default Category:

- Most notifications use the default category
- Default category sys_id: `c97d83137f4432005f58108c3ffa917a`
- Automatically applied if no category specified

## Category Organization Patterns

### By Application

Organize categories by application or module (e.g., Incident Management, Change Management, Problem Management).

**When to Use:**

- Clear application boundaries
- Different teams manage different applications
- Users subscribe by application area

### By Priority/Urgency

Organize categories by notification priority (e.g., Critical Alerts, Standard Updates, Informational).

**When to Use:**

- Priority-based filtering needed
- Users want to control urgency levels
- Different response times required

### By Event Type

Organize categories by event type (e.g., Assignments, Approvals, Status Changes).

**When to Use:**

- Event-based workflows
- Users subscribe by activity type
- Cross-application event tracking

## User Subscription Benefits

Categories enable users to:

1. **Subscribe/Unsubscribe** - Control which notification types they receive
2. **Manage Preferences** - Set preferences per category
3. **Reduce Email Volume** - Opt out of non-essential categories
4. **Customize Experience** - Tailor notifications to their role

## Important Notes

- **Default Category**: `c97d83137f4432005f58108c3ffa917a` used if not specified
- Categories use Record API with table `sys_notification_category`
- Use descriptive names for categories
- Provide clear descriptions
- Limit number of categories (5-10 well-organized categories)
- Group related notifications together
- Don't create too many granular categories
- Consider user roles when organizing

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the NOTIFICATION_CATEGORY knowledge source.
