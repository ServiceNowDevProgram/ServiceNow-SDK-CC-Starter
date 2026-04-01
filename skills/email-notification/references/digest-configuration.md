# Digest Configuration

This document contains details about configuring digest for Email Notifications.

## Contents

- Overview
- Decision Tree
- Trigger Phrases
- Digest Types
- Enabling Digest
- Digest Content
- Decision Guide
- Important Notes
- Next Steps

## Overview

Digest allows grouping multiple notifications into a single email, reducing email volume while keeping users informed. Instead of receiving individual emails for each event, users receive one consolidated email at specified intervals.

**Important:** VCalendar notifications do not support digest configuration.

## Decision Tree: Should I Add Digest?

Use this decision tree to determine if you should add digest configuration:

- **If user says**: "send email" → **NO digest**
- **If user says**: "create notification" → **NO digest**
- **If user says**: "daily digest" → **YES, add digest**
- **If user says**: "group emails" → **YES, add digest**
- **If user says**: "digest notification" → **YES, add digest**
- **If user says**: "batch notifications" → **YES, add digest**

## Trigger Phrases

Add digest ONLY when user uses these specific phrases:

### Digest Requested:

- "daily summary"
- "digest emails"
- "batch notifications"
- "group emails"
- "weekly digest"
- "consolidated email"
- "digest notification"

### Digest NOT Requested:

- "send email" (no digest)
- "create notification" (no digest)
- "email when..." (no digest)

## Digest Types

### Single Digest

One consolidated email per digest interval containing all notifications.

**Characteristics:**

- One email per interval
- All notifications grouped together
- Most common digest type
- Reduces email volume significantly

**Use Cases:**

- High-volume notifications
- Daily/weekly summaries
- Status reports
- Batch updates

### Multiple Digest

Separate emails per record, but still grouped by interval.

**Characteristics:**

- Multiple emails per interval
- One email per record
- More detailed than single digest
- Moderate email volume reduction

**Use Cases:**

- Record-specific updates
- Individual tracking needed
- Moderate volume notifications

## Enabling Digest

### Basic Configuration

Set `allow: true` to enable digest and `default: true` to enable by default for users.

### Disabling Digest

Set `allow: false` to disable digest. When `allow` is `false` or `undefined`, digest fields are allowed in the type but will be ignored.

## Digest Content

### Digest Subject

Custom subject line for digest emails using `subject` property.

### Digest HTML/Text Content

HTML and plain text content for digest emails using `html` and `text` properties.

### Separators

Separators between individual notifications in digest using `separatorHtml` and `separatorText` properties.

### Digest Intervals

Control how often digest emails are sent using `defaultInterval` property. See `digest-intervals.md` for more information.

### Digest Templates

Use templates for reusable digest formatting using `template` property. See `email-templates.md` for more information.

## Decision Guide

### Use Single Digest When:

- ✅ High-volume notifications
- ✅ Users prefer consolidated emails
- ✅ Daily/weekly summaries needed
- ✅ Maximum email volume reduction desired

### Use Multiple Digest When:

- ✅ Record-specific tracking needed
- ✅ Individual emails per record preferred
- ✅ Moderate volume notifications
- ✅ More detailed than single digest

### Don't Use Digest When:

- ❌ Immediate notifications required
- ❌ Time-sensitive alerts
- ❌ VCalendar notifications
- ❌ Low-volume notifications

## Important Notes

- **Single Digest**: One email per interval, all notifications grouped
- **Multiple Digest**: Multiple emails per interval, one per record
- VCalendar notifications cannot have digest configuration
- Must set `allow: true` to enable digest
- Set `default: true` to enable by default for users
- Use `defaultInterval` to control timing (see `digest-intervals.md`)
- Provide clear digest content (subject, html/text, separators)
- Don't use digest for time-sensitive notifications

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the EMAIL_NOTIFICATION knowledge source.
- For digest intervals, see `digest-intervals.md`
- For email templates, see `email-templates.md`
