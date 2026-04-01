# Role-Based UI Views

Views restricted to users with specific roles.

> Complete name and title uniqueness check before implementing

## Verify Roles Exist (MANDATORY)

Use runQuery to verify roles exist before creating views.

## Implementation

### Single Role

```typescript
import { Record } from "@servicenow/sdk/core";

export const adminView = Record({
  $id: Now.ID["admin-view"],
  table: "sys_ui_view",
  data: {
    name: "incident_admin",
    title: "Admin View",
    roles: ["admin"]
  }
});
```

### Multiple Roles

```typescript
export const managerView = Record({
  $id: Now.ID["manager-view"],
  table: "sys_ui_view",
  data: {
    name: "incident_manager",
    title: "Manager View",
    roles: ["sn_facilities_6.business_user", "itil"]
  }
});
```

## Requirements

- **Role Format:** Array of role name strings
  - Single role: `["admin"]`
  - Multiple roles: `["sn_facilities_6.business_user", "itil"]` — each role as a separate string element
- **Verify First:** Always check roles exist in sys_user_role using runQuery before creating view
- **NO ACLs:** Use `roles` field only - never create ACLs for view access control
