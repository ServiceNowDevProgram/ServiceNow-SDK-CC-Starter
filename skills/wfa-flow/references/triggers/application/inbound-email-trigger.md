# inboundEmail Trigger

Fires when an email is received and matches specified conditions. Processes emails and creates/updates records based on email content.

## Table of Contents

- [⚠️ Critical Limitations](#️-critical-limitations)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Important Notes](#important-notes)
- [Next Steps](#next-steps)

## ⚠️ Critical Limitations

**IMPORTANT:** WFA flows are **declarative**, not imperative JavaScript. The following patterns are **NOT supported**:

❌ **JavaScript String Operations:**

- `subject.includes("URGENT")` - NOT SUPPORTED
- `subject.match(/INC\d{7}/)` - NOT SUPPORTED
- `email.endsWith("@company.com")` - NOT SUPPORTED

❌ **Assigning Data Pills to Variables:**

- `const subject = wfa.dataPill(...)` - NOT SUPPORTED

✅ **CORRECT - Use data pills directly in flow conditions and actions:**

```typescript
wfa.flowLogic.if(
  {
    $id: Now.ID["check_urgent"],
    condition: `${wfa.dataPill(_params.trigger.subject, "string_full_utf8")}CONTAINSurgent`
  },
  () => {
    wfa.action(
      action.core.createRecord,
      { $id: Now.ID["create_inc"] },
      {
        table_name: "incident",
        values: TemplateValue({
          short_description: wfa.dataPill(
            _params.trigger.subject,
            "string_full_utf8"
          ),
          description: wfa.dataPill(
            _params.trigger.body_text,
            "string_full_utf8"
          ),
          priority: "1"
        })
      }
    );
  }
);
```

**For Complex Parsing:** Use Script actions or custom APIs for regex/JavaScript operations.

## Best Practices

1. **Use email_conditions:** Filter emails at trigger level to avoid processing unwanted messages.

2. **Validate Email Source:** Filter by sender domain using conditions like `from_addressENDSWITH@company.com`.

3. **Prevent Duplicate Processing:** Check for existing records before creating new ones.

4. **Error Handling:** Implement comprehensive error handling with notifications.

5. **Auto-Reply Loop Prevention:** Filter auto-replies using `subjectNOT CONTAINSOUT OF OFFICE`.

6. **Logging:** Log email processing for audit purposes using correct API:

   ```typescript
   wfa.action(
     action.core.log,
     { $id: Now.ID["log"] },
     {
       log_level: "info",
       log_message: `Processed email from ${wfa.dataPill(_params.trigger.from_address, "string_full_utf8")}`
     }
   );
   ```

## Common Use Cases

- **Creating Incidents from Support Emails:** Parse customer emails and create incident records automatically
- **Routing Emails to Teams:** Analyze subject/content and route to appropriate assignment groups
- **Email-Based Approvals:** Parse approval responses and update approval records

## Important Notes

- **Email Trigger Data:** Access via `_params.trigger.inbound_email`, `from_address`, `subject`, `body_text`, `user`
- **Encoding:** Use `body_text` for plain text, `body_html` for formatted content
- **Attachment Processing:** Requires custom actions or scripts for advanced processing
- **Security:** Never execute code or commands from email content

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_APPLICATION** knowledge source.
