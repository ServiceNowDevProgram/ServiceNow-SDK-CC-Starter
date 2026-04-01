# Hidden UI Views

Views hidden from the **platform view selector dropdown** but usable by **Service Portal, APIs, Mobile, Workspace, and scripts**.

---

## Overview

The `hidden` flag in the `sys_ui_view` table controls whether a UI view appears in the **standard ServiceNow UI view selector**.

- `hidden = true` → View is hidden from normal users in the platform UI dropdown.
- Admin users can still see and select hidden views.
- Hidden views remain fully usable by **Service Portal, APIs, scripts, Mobile, and Workspace**.

> The `hidden` flag is a **UI visibility control only** — it does **not provide security**.  
> Always use **roles, ACLs, and widget permissions** for access control.

---

> Complete name and title uniqueness check before implementing

## Implementation

### Basic Hidden View (Portal / API / System Use)

```typescript
import { Record } from "@servicenow/sdk/core";

export const portalView = Record({
  $id: Now.ID["portal-view"],
  table: "sys_ui_view",
  data: {
    name: "sp_incident_customer",
    title: "Customer Portal View",
    hidden: true // Hides from standard UI view selector (admins can still see)
  }
});
```

## Requirements

- **Hidden Flag:** `hidden: true` (hides from platform dropdown)
- **Can Combine:** Hidden + roles/user/group for additional access control

---
