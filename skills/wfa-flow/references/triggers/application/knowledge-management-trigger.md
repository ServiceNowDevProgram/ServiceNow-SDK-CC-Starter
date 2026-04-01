# knowledgeManagement Trigger

Fires when a knowledge article is created, updated, or changes state. Enables automation around knowledge base management.

## Table of Contents

- [⚠️ Critical Limitations](#️-critical-limitations)
- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Important Notes](#important-notes)
- [Next Steps](#next-steps)

## ⚠️ Critical Limitations

**IMPORTANT:** WFA flows are **declarative**, not imperative JavaScript. The following patterns are **NOT supported**:

❌ **JavaScript String Operations:**

- `article.text.includes("urgent")` - NOT SUPPORTED
- `category.match(/^IT/)` - NOT SUPPORTED

❌ **Assigning Data Pills to Variables:**

- `const category = wfa.dataPill(...)` - NOT SUPPORTED

✅ **CORRECT - Use data pills directly in flow conditions and actions:**

```typescript
wfa.flowLogic.if(
  {
    $id: Now.ID["check_content"],
    condition: `${wfa.dataPill(_params.trigger.knowledge.text, "string_full_utf8")}ISNOTEMPTY`
  },
  () => {
    wfa.action(
      action.core.sendNotification,
      { $id: Now.ID["notify"] },
      {
        record: wfa.dataPill(_params.trigger.knowledge, "reference"),
        users: [wfa.dataPill(_params.trigger.knowledge.author, "reference")],
        message: "Your knowledge article has been published"
      }
    );
  }
);
```

**For Complex Processing:** Use Script actions for text parsing, category logic, or custom validations.

## Best Practices

1. **Check Article State:** Filter by workflow_state (draft, pending_approval, published, retired) at trigger level when possible.

2. **Approval Workflows:** Integrate with askForApproval actions for article review. Implement multi-level review process.

3. **Notification Timing:** Notify authors/reviewers at appropriate workflow stages. Use article state to determine which notifications to send.

4. **Access Control:** Verify user permissions before granting access to knowledge articles. Check read/write access based on article visibility.

## Common Use Cases

- **Article Review Workflow:** Route draft articles to reviewers for approval (filter: `"workflow_state=pending_approval"`)
- **Publication Notifications:** Notify users when articles are published (filter: `"workflow_state=published"`)
- **Expiration Monitoring:** Alert owners when articles approaching retirement date
- **Quality Checks:** Validate article completeness (required fields, content length, attachments)
- **Category-Based Routing:** Route articles to appropriate review teams based on knowledge base category

## Important Notes

- **Trigger Scope:** Fires for knowledge article (kb_knowledge) operations
- **Article Reference:** Access article details via `_params.trigger.knowledge` data pill
- **Workflow States:** common states include: draft, pending_approval, published, scheduled, retired
- **Table Access:** Trigger provides reference to sys_knowledge table (`_params.trigger.table_name`)
- **Timing:** Trigger fires after article operation completes
- **Complex Operations:** Article text parsing, advanced category logic, and rich text processing require custom Script actions

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_APPLICATION** knowledge source.
