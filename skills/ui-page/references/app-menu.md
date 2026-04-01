# Application Menu and Navigation

After creating UI Pages, make them accessible through the ServiceNow application menu.

## Creating Application Menu and App Module

1. **Create Application Menu** (`sys_app_application`): Define the main application entry in the ServiceNow menu
2. **Create App Module** (`sys_app_module`): For each UI Page, create a corresponding module that links to it

## Example

```typescript
// Application Menu
ApplicationMenu({
  $id: Now.ID["my-app-menu"],
  title: "My Application",
  hint: "Custom application for managing incidents",
  order: 100,
  roles: ["admin"]
});

// App Module for UI Page
AppModule({
  $id: Now.ID["my-page-module"],
  title: "Incident Dashboard",
  hint: "View incident metrics and details",
  order: 10,
  application: Now.ID["my-app-menu"],
  link_type: "DIRECT",
  uri: "x_app_page.do" // Match the endpoint from UiPage definition
});
```

## Key Points

- Application Menu groups related modules together
- Each UI Page should have its own App Module
- Use `link_type: "DIRECT"` for UI Pages
- The `uri` must match the `endpoint` in your UiPage definition
- Set appropriate `roles` for access control
