# CATALOG_ITEM

# Catalog Item - Templates and Examples

This knowledge source provides working code templates for ServiceNow Catalog Items. For guidelines, requirements, and patterns, activate the `service-catalog` skill first.

## Basic Catalog Item with Variables

```typescript
import { CatalogItem } from "@servicenow/sdk/core";

const serviceCatalog = "e0d08b13c3330100c8b837659bba8fb4";
const hardwareCategory = "d258b953c611227a0146101fb1be7c31";
const hardwareTopic = "782413a7c3053010069aec4b7d40ddf1";
const itilUsers = "2f137fb2eb303010e0ef83c45e52287c";
const guestUsers = "76f09af6cb1200108ad442fcf7076dbf";

export const laptopRequest = CatalogItem({
  $id: Now.ID["laptop_request"],
  name: "Laptop Request",
  shortDescription: "Request a new laptop for work",
  description: "Submit a request for a new laptop with configuration options.",

  catalogs: [serviceCatalog],
  categories: [hardwareCategory],
  assignedTopics: [hardwareTopic],
  availableFor: [itilUsers],
  notAvailableFor: [guestUsers],

  // Pricing
  pricingDetails: [{ amount: 1299, currencyType: "USD", field: "price" }],

  // Variables for user input
  variables: {
    laptopType: SelectBoxVariable({
      question: "Laptop Type",
      choices: {
        standard: { label: "Standard Laptop", sequence: 1 },
        developer: { label: "Developer Workstation", sequence: 2 }
      },
      mandatory: true,
      order: 100
    }),
    justification: MultiLineTextVariable({
      question: "Business Justification",
      mandatory: true,
      order: 200
    })
  },

  // Fulfillment
  flow: "523da512c611228900811a37c97c2014",
  fulfillmentAutomationLevel: "semiAutomated",
  deliveryTime: { days: 7, hours: 0 },

  // Access control
  accessType: "restricted",

  // UI settings
  availability: "both",
  requestMethod: "order"
});
```

---

## Catalog Item with Variable Sets

```typescript
export const softwareLicenseRequest = CatalogItem({
  $id: Now.ID["software_license_request"],
  name: "Software License Request",
  shortDescription: "Request a software license",

  catalogs: [serviceCatalog],
  categories: [softwareCategory],

  // Attach reusable variable sets
  variableSets: [
    { variableSet: contactInfoSet, order: 100 },
    { variableSet: approvalInfoSet, order: 200 }
  ],

  // Item-specific variables
  variables: {
    software_name: SingleLineTextVariable({
      question: "Software Name",
      mandatory: true,
      order: 100
    }),
    license_type: SelectBoxVariable({
      question: "License Type",
      choices: {
        individual: { label: "Individual", sequence: 1 },
        team: { label: "Team (5 seats)", sequence: 2 },
        enterprise: { label: "Enterprise (unlimited)", sequence: 3 }
      },
      mandatory: true,
      order: 200
    }),
    number_of_licenses: SingleLineTextVariable({
      question: "Number of Licenses",
      defaultValue: "1",
      order: 300
    }),
    justification: MultiLineTextVariable({
      question: "Business Justification",
      mandatory: true,
      order: 400
    })
  },

  // Pricing with recurring charges
  pricingDetails: [
    { amount: 0, currencyType: "USD", field: "price" },
    { amount: 99, currencyType: "USD", field: "recurring_price" }
  ],
  recurringFrequency: "monthly",

  flow: "523da512c611228900811a37c97c2014",
  deliveryTime: { days: 3, hours: 0 }
});
```

## Circular Dependency Resolution

When a flow needs to use `getCatalogVariables` with the catalog item's variables,
there's a circular dependency:

- Flow needs to import CatalogItem (for template_catalog_item and catalog_variables)
- CatalogItem needs to reference Flow (for fulfillment)

**Solution Pattern:**

1. **Flow** → imports CatalogItem (can use getCatalogVariables with variables)
2. **CatalogItem** → uses `Now.ref()` to reference Flow (NO import)

```typescript
// catalog-item.now.ts - Uses Now.ref(), does NOT import flow
export const myCatalogItem = CatalogItem({
  $id: Now.ID["my_catalog_item"],
  flow: Now.ref("sys_hub_flow", "my_flow"), // No import needed
  variables: { ... }
});

// flow.now.ts - Imports catalog item for getCatalogVariables
import { myCatalogItem } from "../catalog-item.now";

export const myFlow = Flow(
  { $id: Now.ID["my_flow"] },
  wfa.trigger(trigger.application.serviceCatalog, ...),
  _params => {
    const vars = wfa.action(action.core.getCatalogVariables, {
      template_catalog_item: `${myCatalogItem}`,
      catalog_variables: [myCatalogItem.variables.field1, ...]
    });
  }
);
```

---

## Properties

| Name                       | Type                                          | Description                                                               |
| -------------------------- | --------------------------------------------- | ------------------------------------------------------------------------- |
| $id                        | string                                        | **Required.** Unique identifier.                                          |
| name                       | string                                        | **Required.** Name to appear in the catalog.                              |
| shortDescription           | string                                        | Brief summary shown in catalog listings.                                  |
| description                | string                                        | Detailed description shown on the item page.                              |
| catalogs                   | array                                         | sys_id of existing catalogs. Use queries to find.                         |
| categories                 | array                                         | sys_id of existing categories. Use queries to find.                       |
| assignedTopics             | array                                         | sys_id of existing topics. Controls ESC portal visibility.                |
| accessType                 | `'restricted'` \| `'unrestricted'`            | Controls who can request the item. Default: `'restricted'`.               |
| availableFor               | array                                         | sys_id of user criteria for availability.                                 |
| notAvailableFor            | array                                         | sys_id of user criteria for restrictions (overrides availableFor).        |
| roles                      | array                                         | Roles for catalog item access.                                            |
| active                     | boolean                                       | Whether the item is active. Default: `true`.                              |
| availability               | `'desktopOnly'` \| `'both'` \| `'mobileOnly'` | Platform availability. Default: `'desktopOnly'`.                          |
| requestMethod              | `'order'` \| `'request'` \| `'submit'`        | Submission button label. Default: `'order'`.                              |
| flow                       | string                                        | Flow Designer flow for fulfillment (recommended).                         |
| workflow                   | string                                        | Legacy workflow for fulfillment.                                          |
| executionPlan              | string                                        | Delivery plan for fulfillment.                                            |
| fulfillmentAutomationLevel | string                                        | `'unspecified'` \| `'manual'` \| `'semiAutomated'` \| `'fullyAutomated'`. |
| fulfillmentGroup           | string                                        | Group responsible for delivery.                                           |
| deliveryTime               | Duration                                      | Estimated delivery time `{ days, hours }`.                                |
| pricingDetails             | array                                         | Pricing breakdown: `{ amount, currencyType, field }`.                     |
| recurringFrequency         | string                                        | Required when pricingDetails contains `'recurring_price'`.                |
| variables                  | object                                        | Variable definitions for the form.                                        |
| variableSets               | array                                         | Variable set references: `{ variableSet, order }`.                        |

## Fulfillment Configuration

**Flow Designer** (`flow`) — Recommended fulfillment method for catalog items. Use `Now.ref()` to reference a project-defined flow or provide a sys_id string for an existing platform flow.

## Pricing Configuration

Use `pricingDetails` array with `{ amount, currencyType, field }` objects. Supported `field` values: `price`, `recurring_price`. When using `recurring_price`, `recurringFrequency` is required (`monthly`, `yearly`, etc.).

## Access Control

Use User Criteria as the preferred method:

- `availableFor`: Users/groups who can see the item
- `notAvailableFor`: Users/groups restricted (overrides availableFor)
- `accessType`: `'restricted'` or `'unrestricted'`
- `roles`: Role-based restrictions
- `assignedTopics`: For Employee Center visibility

Standard user criteria updates take effect immediately; scripted user criteria requires session relaunch.

## UI Display Options

| Property                  | Description                     |
| ------------------------- | ------------------------------- |
| hideAddToCart             | Hides "Add to Cart" button      |
| hideAttachment            | Hides attachment section        |
| hideDeliveryTime          | Hides delivery time             |
| hideQuantitySelector      | Hides quantity selection        |
| hideSaveAsDraft           | Hides "Save as Draft"           |
| hideSP                    | Hides from Service Portal       |
| hideAddToWishList         | Hides "Add to Wishlist"         |
| ignorePrice               | Ignores price display           |
| omitPrice                 | Omits price entirely            |
| mandatoryAttachment       | Requires attachment             |
| makeItemNonConversational | Prevents virtual agent ordering |
| showVariableHelpOnLoad    | Shows help text by default      |

---

## Common User Requests Mapping

| User Request                   | Agent Implementation                                  |
| ------------------------------ | ----------------------------------------------------- |
| "Order a laptop"               | Catalog Item with variables and Flow Designer         |
| "Request software license"     | Catalog Item with variable sets and recurring pricing |
| "Service with approvals"       | Catalog Item with flow fulfillment                    |
| "Reusable fields across items" | Catalog Item with variable sets                       |

---

## Quick Reference

### ALWAYS

- Assign items to at least one catalog and category
- Use queries to find existing catalog/category/topic sys_ids
- Use Flow Designer as the preferred fulfillment method
- Use `snake_case` for variable names
- Use `order` increments of 100

### NEVER

- Use catalog items for creating task records directly (use Record Producers)
- Skip catalogs or categories assignment
- Hard-code sys_ids without documenting their source
