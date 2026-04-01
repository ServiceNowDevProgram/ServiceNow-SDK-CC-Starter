# CATALOG_UI_POLICY

# Catalog UI Policy - Templates and Examples

This knowledge source provides working code templates for ServiceNow Catalog UI Policies. For guidelines, requirements, and patterns, activate the `service-catalog` skill first.

## Show/Hide Variable Based on Condition

```typescript
import { CatalogUiPolicy } from "@servicenow/sdk/core";
import { hardwareRequestItem } from "./catalog-items/HardwareRequest.now";

export const managerApprovalPolicy = CatalogUiPolicy({
  $id: Now.ID["manager_approval_policy"],
  shortDescription: "Show manager approval when high priority selected",
  catalogItem: hardwareRequestItem,
  catalogCondition: `${hardwareRequestItem.variables.priority}=high^EQ`,
  actions: [
    {
      variableName: hardwareRequestItem.variables.manager_approval,
      visible: true,
      mandatory: true
    }
  ]
});
```

---

## Make Variable Read-Only with Value

```typescript
export const readOnlyPolicy = CatalogUiPolicy({
  $id: Now.ID["readonly_license_policy"],
  shortDescription: "Make license type read-only for standard software",
  catalogItem: softwareRequestItem,
  catalogCondition: `${softwareRequestItem.variables.software_type}=standard^EQ`,
  actions: [
    {
      variableName: softwareRequestItem.variables.license_type,
      readOnly: true,
      value: "standard_license",
      valueAction: "setValue"
    }
  ]
});
```

---

## Policy with Client Scripts

```typescript
CatalogUiPolicy({
  $id: Now.ID["vm_prod_controls_policy"],
  shortDescription: "VM: Prod/BizCritical Controls",
  catalogItem: cloudVmRequest,
  catalogCondition: `${cloudVmRequest.variables.environment}=prod^OR${cloudVmRequest.variables.business_critical}=true^EQ`,
  active: true,
  onLoad: true,
  reverseIfFalse: true,
  runScripts: true,
  runScriptsInUiType: "all",

  actions: [
    {
      variableName: cloudVmRequest.variables.backup_required,
      value: "true",
      valueAction: "setValue",
      readOnly: true,
      order: 100
    },
    {
      variableName: cloudVmRequest.variables.cost_center,
      mandatory: true,
      order: 200
    }
  ],

  executeIfTrue: Now.include("../../scripts/vm-production-controls.js"),
  executeIfFalse: Now.include("../../scripts/vm-development-controls.js")
});
```

**vm-production-controls.js:**

```javascript
function onCondition() {
  var PROD_REGIONS = [
    ["AP-South-1", "AP-South-1 (Mumbai)"],
    ["EU-West-1", "EU-West-1 (Ireland)"]
  ];

  // Restrict region options
  g_form.clearOptions("region");
  PROD_REGIONS.forEach(function (pair) {
    g_form.addOption("region", pair[0], pair[1]);
  });

  // Show guidance message
  g_form.showFieldMsg(
    "environment",
    "Production VMs enforce backup and require cost center.",
    "info"
  );
}
```

---

## Policy Applied to Variable Set

```typescript
export const internationalShippingPolicy = CatalogUiPolicy({
  $id: Now.ID["international_shipping_policy"],
  shortDescription: "Show customs fields for international shipping",
  variableSet: shippingVariableSet,
  appliesTo: "set",
  catalogCondition: `${shippingVariableSet.variables.shipping_country}!=US^EQ`,
  appliesOnCatalogItemView: true,
  appliesOnRequestedItems: true,
  actions: [
    {
      variableName: shippingVariableSet.variables.customs_declaration,
      visible: true,
      mandatory: true,
      variableMessage: "Required for international shipping",
      variableMessageType: "warning"
    }
  ]
});
```

---

## Multiple Actions on Multiple Variables

```typescript
export const urgentRequestPolicy = CatalogUiPolicy({
  $id: Now.ID["urgent_request_policy"],
  shortDescription: "Show additional fields for urgent requests",
  catalogItem: laptopRequest,
  catalogCondition: `${laptopRequest.variables.urgency}=1^EQ`,
  onLoad: true,
  reverseIfFalse: true,
  actions: [
    {
      variableName: laptopRequest.variables.justification,
      mandatory: true,
      variableMessage: "Justification required for urgent requests",
      variableMessageType: "info",
      order: 100
    },
    {
      variableName: laptopRequest.variables.manager_approval,
      visible: true,
      mandatory: true,
      order: 200
    },
    {
      variableName: laptopRequest.variables.delivery_date,
      visible: true,
      order: 300
    }
  ]
});
```

---

## Properties

| Property                 | Type           | Description                                                            |
| ------------------------ | -------------- | ---------------------------------------------------------------------- |
| $id                      | Now.ID[string] | **Required.** Unique identifier.                                       |
| shortDescription         | string         | **Required.** Description of what the policy does.                     |
| catalogItem              | string         | **Required** if not using variableSet.                                 |
| variableSet              | string         | **Required** if not using catalogItem.                                 |
| appliesTo                | string         | Required if using variableSet. `'item'` or `'set'`. Default: `'item'`. |
| active                   | boolean        | Whether the policy is active. Default: `true`.                         |
| onLoad                   | boolean        | Run on form load. Default: `true`.                                     |
| reverseIfFalse           | boolean        | Reverse actions when condition is false. Default: `true`.              |
| catalogCondition         | string         | Condition using encoded query syntax.                                  |
| appliesOnCatalogItemView | boolean        | Applies to catalog item view. Default: `true`.                         |
| appliesOnTargetRecord    | boolean        | Applies to target record. Default: `false`.                            |
| appliesOnCatalogTasks    | boolean        | Applies to catalog tasks. Default: `false`.                            |
| appliesOnRequestedItems  | boolean        | Applies to requested items. Default: `false`.                          |
| runScripts               | boolean        | Execute client scripts. Default: `false`.                              |
| executeIfTrue            | string         | Script when condition is true.                                         |
| executeIfFalse           | string         | Script when condition is false.                                        |
| runScriptsInUiType       | string         | `'desktop'`, `'mobileOrServicePortal'`, `'all'`. Default: `'desktop'`. |
| actions                  | array          | List of variable actions.                                              |

## Action Properties

| Property            | Type    | Description                       |
| ------------------- | ------- | --------------------------------- |
| variableName        | object  | **Required.** Variable reference. |
| visible             | boolean | Show/hide the variable.           |
| mandatory           | boolean | Make variable required.           |
| readOnly            | boolean | Make variable read-only.          |
| value               | string  | Value to set.                     |
| valueAction         | string  | `'clearValue'` or `'setValue'`.   |
| order               | number  | Execution order. Default: `100`.  |
| variableMessage     | string  | Message to display on the field.  |
| variableMessageType | string  | `'info'`, `'warning'`, `'error'`. |

## Condition Syntax

```typescript
// Simple condition
catalogCondition: `${catalogItem.variables.priority}=high^EQ`;

// Multiple conditions with AND
catalogCondition: `${catalogItem.variables.env}=prod^${catalogItem.variables.critical}=true^EQ`;

// Multiple conditions with OR
catalogCondition: `${catalogItem.variables.env}=prod^OR${catalogItem.variables.critical}=true^EQ`;

// Not empty check
catalogCondition: `${catalogItem.variables.reference}ISNOTEMPTY^EQ`;
```

## Priority Rules

1. **Mandatory** has highest priority
2. If a variable is mandatory and has no value, readonly/hide actions **do not work**
3. If a variable set/container has a mandatory variable without value, the entire set **cannot be hidden**
4. "Clear value" action does not work on variable sets and containers

## Variable Type Limitations

| Policy Type | Not Applicable To                                                  |
| ----------- | ------------------------------------------------------------------ |
| Mandatory   | Fraction, Container Split, Container End, UI Macro, Label, UI Page |
| Read-only   | Fraction, Container Split, Container End, UI Macro, Label, UI Page |
| Visibility  | Fraction, Container Split, Container End                           |

## When to Use UI Policy vs Client Script

| Use Case                 | UI Policy     | Client Script |
| ------------------------ | ------------- | ------------- |
| Show/hide variables      | **Preferred** | Supported     |
| Make variables mandatory | **Preferred** | Supported     |
| Make variables read-only | **Preferred** | Supported     |
| Set variable values      | Supported     | Supported     |
| Complex validation       | Limited       | **Preferred** |
| Dynamic calculations     | Limited       | **Preferred** |
| API calls / async        | Not supported | Supported     |
| Form submission control  | Not supported | Supported     |

---

## Common User Requests Mapping

| User Request                         | Agent Implementation                              |
| ------------------------------------ | ------------------------------------------------- |
| "Show field when priority is high"   | UI Policy with condition and visible action       |
| "Make field read-only conditionally" | UI Policy with readOnly and setValue actions      |
| "Run script when condition matches"  | UI Policy with runScripts and executeIfTrue/False |
| "Apply policy to variable set"       | UI Policy with variableSet and appliesTo: "set"   |

---

## Quick Reference

### ALWAYS

- Use UI Policies for simple show/hide/mandatory/read-only logic
- Use object references in properties (e.g., `catalogItem.variables.urgency`)
- End conditions with `^EQ`
- Use `reverseIfFalse: true` to auto-reverse when condition no longer matches

### NEVER

- Use UI Policies for complex validation (use Client Scripts)
- Use UI Policies for async/API calls (use Client Scripts)
- Hide mandatory variables that have no value (action will not work)
