# Digest Intervals

This document contains details about configuring digest intervals for Email Notifications.

## Contents

- Overview
- Common Interval Patterns
- Decision Guide
- Important Notes
- Next Steps

## Overview

Digest intervals control the frequency at which digest notifications are sent to users. Instead of receiving individual emails immediately, users receive consolidated digest emails at specified intervals.

## Common Interval Patterns

### Daily Digest

Send once per day at specified time.

**Use Cases:**

- Regular daily updates
- End-of-day summaries
- Morning briefings
- Moderate notification volume

### Weekly Digest

Send once per week on specified day.

**Use Cases:**

- Low-frequency summaries
- Weekly reports
- Status updates
- Low notification volume

### Hourly Digest

Send every hour.

**Use Cases:**

- High-frequency updates
- Near real-time summaries
- Operational monitoring
- High notification volume

### Custom Interval

Send at custom minute intervals.

**Use Cases:**

- Specific timing requirements
- Non-standard intervals
- Business-specific schedules
- Unique use cases

## Decision Guide

### Choose Daily When:

- ✅ Regular daily updates needed
- ✅ End-of-day summaries
- ✅ Morning briefings
- ✅ Moderate notification volume

### Choose Weekly When:

- ✅ Low-frequency summaries
- ✅ Weekly reports
- ✅ Status updates
- ✅ Low notification volume

### Choose Hourly When:

- ✅ High-frequency updates
- ✅ Near real-time summaries
- ✅ Operational monitoring
- ✅ High notification volume

### Choose Custom When:

- ✅ Specific timing requirements
- ✅ Non-standard intervals
- ✅ Business-specific schedules
- ✅ Unique use cases

## Important Notes

- Digest intervals use Record API with table `sys_email_digest_interval`
- Match interval to business needs
- Consider user time zones when setting times
- Use descriptive names for intervals
- Don't make intervals too short (defeats purpose)
- Don't use digest intervals for urgent notifications
- Provide 2-3 common options for users
- Day of week values: Monday=1, Sunday=7
- Time format: 24-hour (HH:MM:SS)

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the EMAIL_DIGEST_INTERVAL knowledge source.
- For digest configuration, see `digest-configuration.md`
