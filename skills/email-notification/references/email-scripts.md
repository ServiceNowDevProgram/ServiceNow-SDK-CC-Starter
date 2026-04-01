# Email Scripts

This document contains details about Email Scripts for dynamic content generation.

## Contents

- Overview
- When to Use
- Script Variables
- Important Notes
- Next Steps

## Overview

Email scripts provide server-side scripting capabilities for email notifications, allowing you to generate dynamic content, perform calculations, and execute custom logic when notifications are sent.

## When to Use

### Use Email Scripts When

- ✅ Dynamic content generation needed
- ✅ Custom calculations required
- ✅ Complex logic for content
- ✅ Data aggregation needed
- ✅ Conditional content generation

### Don't Use By Default

- ❌ Simple static content
- ❌ Standard variable substitution sufficient
- ❌ No custom logic needed

## Script Variables

Available in email scripts:

- `current`: GlideRecord of the target record
- `template`: TemplatePrinter for formatting output
- `email`: (Optional) EmailOutbound object
- `email_action`: (Optional) GlideRecord of email action
- `event`: (Optional) GlideRecord of event

## Script validation rules

- Must maintain the function structure exactly as shown in knowledge source example
- Can use any of the provided parameters
- Must use `template.print()` to output content
- Maximum script length: 4000 characters

## Important Notes

- Email scripts use Record API with table `sys_email_script`
- Use `template` object to set variables for use in email content
- Scripts execute server-side when notification is sent
- Keep scripts simple and efficient
- Test scripts thoroughly
- Use for dynamic content that can't be achieved with variable substitution
- Don't use for simple variable substitution

## Next Steps

- For fluent API and coding examples, use `get_knowledge_source` tool to get the EMAIL_SCRIPT knowledge source.
- For email access restrictions, see `email-access-restrictions.md`
