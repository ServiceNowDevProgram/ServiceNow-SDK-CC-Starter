# Group-Based UI Views

Views restricted to members of a specific group.

> Complete name and title uniqueness check before implementing

## Verify Group Exists (MANDATORY)

Use `runQuery` to verify the group exists and is active before creating the view.

## Implementation

### Using Existing Group

```typescript
import { Record } from "@servicenow/sdk/core";

export const supportView = Record({
  $id: Now.ID["support-view"],
  table: "sys_ui_view",
  data: {
    name: "incident_support_team",
    title: "Support Team View",
    group: "<group_sys_id>" // sys_id of the group
  }
});
```

### Creating Group First

```typescript
export const supportGroup = Record({
  $id: Now.ID["support-group"],
  table: "sys_user_group",
  data: {
    name: "support_team",
    description: "Support Team",
    active: true
  }
});

export const supportView = Record({
  $id: Now.ID["support-view"],
  table: "sys_ui_view",
  data: {
    name: "incident_support_team",
    title: "Support Team View",
    group: supportGroup
  }
});
```

## Requirements

- **Group Reference:** Use sys_id from query OR Now.ID reference
- **Verify First:** Always check group exists and is active
- **Access:** Only group members can see/select view
