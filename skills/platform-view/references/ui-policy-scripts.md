# UI Policy Scripts

This document contains detailed information about using scripts in UI Policies.

## Contents

- Overview
- When to Use Scripts
- Script Properties
- Script Execution Context
- Best Practices
- Examples
- Important Notes
- Next Steps

## Overview

UI Policy scripts provide advanced client-side control over form behavior that goes beyond simple field actions. Scripts execute in the browser and have access to the ServiceNow client-side APIs, particularly the g_form API for form manipulation.

## When to Use Scripts

Use scripts in UI Policies when you need to:

- Implement complex conditional logic that can't be expressed through field actions alone
- Perform dynamic calculations or field value manipulations
- Display custom user messages based on complex conditions
- Integrate with g_form API methods for advanced form controls
- Execute custom validation logic on the client-side
- Manipulate multiple fields in coordinated ways
- Access client-side data that isn't available through simple conditions

**Note:** For simple field visibility, mandatory status, or read-only behavior, use field actions instead of scripts. Scripts add complexity and should only be used when necessary.

## Script Properties

To enable scripts in a UI Policy, you must set `runScripts: true`. When scripts are enabled, the following properties become available:

### scriptTrue

The `scriptTrue` property contains JavaScript code that executes when the policy conditions evaluate to true.

- **Type:** string
- **Format:** Must be wrapped in `function onCondition() { ... }` format
- **Default:** `'function onCondition() {\n\n}'`
- **Access:** Full access to client-side ServiceNow APIs (g_form, g_user, etc.)

### scriptFalse

The `scriptFalse` property contains JavaScript code that executes when the policy conditions evaluate to false.

- **Type:** string
- **Format:** Must be wrapped in `function onCondition() { ... }` format
- **Default:** `'function onCondition() {\n\n}'`
- **Access:** Full access to client-side ServiceNow APIs (g_form, g_user, etc.)

### uiType

The `uiType` property determines where the scripts execute:

- `'desktop'` (default): Scripts run only on desktop interface
- `'mobile-service-portal'`: Scripts run on mobile and Service Portal interfaces
- `'all'`: Scripts run on all interfaces

The plugin automatically converts between TypeScript string literals and ServiceNow numeric values.

### isolateScript

The `isolateScript` property controls script isolation:

- `false` (default): Scripts run in the global scope
- `true`: Scripts run in an isolated scope for better security

## Script Execution Context

UI Policy scripts execute on the client-side in the following contexts:

1. **Form Load:** When `onLoad: true`, scripts execute when the form first loads
2. **Condition Change:** Scripts re-execute whenever fields referenced in the conditions change
3. **Client-Side Only:** Scripts have no access to server-side data or APIs

### Available APIs

Scripts have access to these client-side ServiceNow APIs:

- **g_form:** Primary form manipulation API
- **g_user:** Current user information
- **g_scratchpad:** Data passed from server to client
- **g_modal:** Modal dialog controls (if applicable)
- **g_service_catalog:** Service catalog methods (if applicable)

### Common g_form Methods

```javascript
// Field visibility
g_form.setVisible("field_name", true);
g_form.setVisible("field_name", false);

// Mandatory fields
g_form.setMandatory("field_name", true);
g_form.setMandatory("field_name", false);

// Read-only fields
g_form.setReadOnly("field_name", true);
g_form.setReadOnly("field_name", false);

// Field values
g_form.setValue("field_name", "value");
g_form.getValue("field_name");
g_form.clearValue("field_name");

// Messages
g_form.addInfoMessage("Information message");
g_form.addWarningMessage("Warning message");
g_form.addErrorMessage("Error message");
g_form.clearMessages();

// Field messages
g_form.showFieldMsg("field_name", "Message text", "info");
g_form.showFieldMsg("field_name", "Message text", "error");
g_form.hideFieldMsg("field_name");
```

## Best Practices

1. **Keep Scripts Simple:** Use field actions for simple behaviors; reserve scripts for complex logic
2. **Wrap in Functions:** Always wrap script code in `function onCondition() { ... }` format
3. **Handle Both States:** Provide both `scriptTrue` and `scriptFalse` to properly reverse behaviors
4. **Test Across UI Types:** If using `uiType: 'all'`, test on desktop, mobile, and service portal
5. **Use Isolated Scope:** Set `isolateScript: true` for better security and to avoid variable conflicts
6. **Clear Messages:** When adding messages, also clear them in `scriptFalse` to avoid stale messages
7. **Performance:** Avoid heavy computations or excessive DOM manipulations
8. **Error Handling:** Consider edge cases and null values when accessing field data

## Examples

### Basic Script with Messages

```javascript
export const taskAssignmentPolicy = UiPolicy({
  $id: Now.ID["task_assignment_policy"],
  table: "task",
  shortDescription: "Task assignment validation",
  onLoad: true,
  conditions: "assigned_to=",
  runScripts: true,
  scriptTrue: `function onCondition() {
    g_form.addInfoMessage('Please assign this task to a user.');
    g_form.setMandatory('assigned_to', true);
  }`,
  scriptFalse: `function onCondition() {
    g_form.clearMessages();
  }`,
  uiType: "desktop"
});
```

### Complex Validation

```javascript
export const incidentValidationPolicy = UiPolicy({
  $id: Now.ID["incident_validation_policy"],
  table: "incident",
  shortDescription: "Validate incident fields based on priority",
  onLoad: true,
  conditions: "priority=1",
  runScripts: true,
  scriptTrue: `function onCondition() {
    // High priority requires additional fields
    g_form.setMandatory('justification', true);
    g_form.setMandatory('business_service', true);
    g_form.showFieldMsg('priority', 'High priority incidents require justification and business service', 'info');
    
    // Auto-set urgency and impact
    if (g_form.getValue('urgency') == '') {
      g_form.setValue('urgency', '1');
    }
    if (g_form.getValue('impact') == '') {
      g_form.setValue('impact', '1');
    }
  }`,
  scriptFalse: `function onCondition() {
    // Clear mandatory requirements
    g_form.setMandatory('justification', false);
    g_form.setMandatory('business_service', false);
    g_form.hideFieldMsg('priority');
  }`,
  uiType: "all",
  isolateScript: true
});
```

### User-Based Logic

```javascript
export const approvalPolicy = UiPolicy({
  $id: Now.ID["approval_policy"],
  table: "change_request",
  shortDescription: "Show approval notes for approvers only",
  onLoad: true,
  conditions: "approval=requested",
  runScripts: true,
  scriptTrue: `function onCondition() {
    // Check if user has approver role
    if (g_user.hasRole('approver_user')) {
      g_form.setVisible('approval_notes', true);
      g_form.setMandatory('approval_notes', true);
      g_form.showFieldMsg('approval_notes', 'Please provide approval notes', 'info');
    } else {
      g_form.setVisible('approval_notes', false);
      g_form.addInfoMessage('This change is pending approval.');
    }
  }`,
  scriptFalse: `function onCondition() {
    g_form.setVisible('approval_notes', false);
    g_form.setMandatory('approval_notes', false);
    g_form.hideFieldMsg('approval_notes');
    g_form.clearMessages();
  }`,
  uiType: "all",
  isolateScript: true
});
```

## Important Notes

- **Client-Side Only:** Scripts execute on the client-side and cannot access server-side data directly
- **Function Wrapper:** All scripts must be wrapped in `function onCondition() { ... }` format
- **Enable Scripts:** You must set `runScripts: true` to enable script execution
- **Condition Re-evaluation:** Scripts re-execute whenever condition fields change
- **Performance Impact:** Complex scripts can impact form load and interaction performance
- **Combine with Actions:** You can use both scripts and field actions in the same policy
- **Script vs Actions:** Field actions are processed first, then scripts execute
- **UI Type Awareness:** Scripts may behave differently across UI types; test accordingly
- **Isolated Scope Recommended:** Use `isolateScript: true` to avoid variable conflicts with other scripts

## Next Steps

- For field action details, refer to `ui-policy-guide.md` loaded via `load_skill_resource`
- For related list action details, load `references/ui-policy-related-lists.md` via `load_skill_resource` with skill name "ui-layout-control"
- For fluent API and more examples, use `get_knowledge_source` tool to get the UI Policy knowledge source
