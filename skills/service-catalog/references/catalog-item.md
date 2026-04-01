# Catalog Item Reference

For properties and templates, use `get_knowledge_source` with knowledge source `CATALOG_ITEM`.

## Fulfillment Configuration

Catalog items use a flow-based fulfillment mechanism:

**Flow Designer** (`flow`) — Automates request fulfillment using Flow Designer. Triggered on submission, it handles approvals, notifications, and task creation — retrieving catalog variables for dynamic processing and tracking progress through defined stages.

Reference a flow defined in the project using `Now.ref()`. This avoids importing the flow file directly, which prevents circular dependencies when the flow also imports the catalog item (e.g., for `getCatalogVariables`):

```typescript
flow: Now.ref("sys_hub_flow", "my_fulfillment_flow");
```

Or reference an existing flow by sys_id:

```typescript
flow: "e0d08b13c3330100c8b837659bba8fb4";
```

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

## Best Practices

- Create separate catalogs for different business units
- Use logical category hierarchy (max 3 levels deep)
- Each item should represent one orderable good or service
- Use Flow Designer as the preferred fulfillment method
- Set `active: false` for items under development
- Use `meta` tags for improved search discovery
- Always use queries to find existing catalog/category/topic sys_ids
- Use Service Catalog trigger for flow fulfillment

Activate the `wfa-flow` skill to automate Service Catalog request fulfillment using Flow Designer.
For complete examples and templates, use `get_knowledge_source` with knowledge source `CATALOG_ITEM`.
