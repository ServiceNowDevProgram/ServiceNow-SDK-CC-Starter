# User-Specific UI Views

Views restricted to a single user.

> Complete name and title uniqueness check before implementing

## Verify User Exists (MANDATORY)

If user not found: Ask the user to confirm the username or provide:

- Correct username/user_name
- OR create the user first (see "Creating User First" section below)

## Implementation

### Using Existing User

use runQuery to verify user

```typescript
import { Record } from "@servicenow/sdk/core";

export const personalView = Record({
  $id: Now.ID["john-view"],
  table: "sys_ui_view",
  data: {
    name: "incident_john_personal",
    title: "John's Personal View",
    user: //12344 User sys id
  }
});
```

### Creating User First

```typescript
export const newUser = Record({
  $id: Now.ID["user-jane"],
  table: "sys_user",
  data: {
    user_name: "jane.smith",
    first_name: "Jane",
    last_name: "Smith",
    email: "jane.smith@example.com",
    active: true
  }
});

export const personalView = Record({
  $id: Now.ID["jane-view"],
  table: "sys_ui_view",
  data: {
    name: "incident_jane_personal",
    title: "Jane's Personal View",
    user: newUser
  }
});
```

## Requirements

- **User Reference:** Use sys_id from query OR Now.ID reference
- **Verify First:** Always check user exists and is active
