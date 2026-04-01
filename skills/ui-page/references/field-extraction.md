# ServiceNow Field Extraction Pattern

When using `sysparm_display_value=all` (recommended), ServiceNow reference, choice, and sys_id fields become objects. React cannot render objects directly, so you must extract primitive values.

## The Problem

```tsx
// ServiceNow returns objects for reference fields:
{ assigned_to: { display_value: "John Doe", value: "46b87022a9fe198101" } }

// WRONG - renders object directly (causes React error):
<p>Assigned to: {incident.assigned_to}</p>
```

## Required Utility Functions (MANDATORY)

**ALWAYS use `sysparm_display_value=all` in ALL Table API calls.** This guarantees every field has both `display_value` and `value` properties.

Create these utility functions in EVERY project:

```ts
// src/client/utils/fields.ts
export const display = field => {
  if (typeof field === "string") {
    return field;
  }

  return field?.display_value || "";
};

export const value = field => {
  if (typeof field === "string") {
    return field;
  }

  return field?.value || "";
};
```

## Usage Pattern

```tsx
import { display, value } from "./utils/fields";

// For UI display:
<td>{display(record.short_description)}</td>
<td>{display(record.assigned_to)}</td>

// For operations/keys:
await updateRecord(value(record.sys_id), data);
{records.map(r => <li key={value(r.sys_id)}>)}
```

## Guaranteed Field Structure

With `sysparm_display_value=all`, fields have this structure:

```json
{
  "sys_id": { "display_value": "abc123...", "value": "abc123..." },
  "short_description": {
    "display_value": "Fix login bug",
    "value": "Fix login bug"
  },
  "assigned_to": { "display_value": "John Smith", "value": "user_sys_id" },
  "state": { "display_value": "In Progress", "value": "2" },
  "priority": { "display_value": "High", "value": "1" }
}
```

## Common Mistakes

```tsx
// WRONG - accessing object directly
<span>{record.assigned_to}</span>

// WRONG - assuming string type
<span>{record.assigned_to.toString()}</span>

// CORRECT - using display helper
<span>{display(record.assigned_to)}</span>

// WRONG - using value for display
<span>{value(record.state)}</span>  // Shows "2" instead of "In Progress"

// CORRECT - display for UI, value for operations
<span>{display(record.state)}</span>  // Shows "In Progress"
await api.update(value(record.sys_id), data);  // Uses sys_id value
```
