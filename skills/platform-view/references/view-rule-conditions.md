# Condition-Based View Rules

Simple, declarative view rules using device type and encoded queries (no scripts).

## Contents

- [Device-Based Switching](#device-based-switching)
- [Condition-Based Switching](#condition-based-switching)
- [Encoded Query Syntax](#encoded-query-syntax-critical)
- [Important Properties for Condition-Based Rules](#important-properties-for-condition-based-rules)
- [Critical Validation](#critical-validation)

## Device-Based Switching

Switch views based on device type (mobile, tablet, browser):

```typescript
import { Record } from "@servicenow/sdk/core";

export const mobileRule = Record({
  $id: Now.ID["mobile-rule"],
  table: "sysrule_view",
  data: {
    name: "Mobile View Rule",
    table: "incident",
    view: "mobile", // Must exist in sys_ui_view
    device_type: "mobile",
    active: true,
    overrides_user_preference: true
  }
});
```

## Condition-Based Switching

Switch views based on field values using encoded queries:

```typescript
// High priority incidents
export const criticalRule = Record({
  $id: Now.ID["critical-rule"],
  table: "sysrule_view",
  data: {
    name: "Critical Priority Rule",
    table: "incident",
    view: "critical",
    condition: "priority=1^ORpriority=2^EQ", // MUST end with ^EQ
    active: true,
    overrides_user_preference: true
  }
});
```

## Encoded Query Syntax (CRITICAL)

**MUST use backend field names and internal values, NOT UI labels.**

### Requirements

1. **Must end with `^EQ`** - All encoded queries must terminate with `^EQ`
2. **Use field names** - Element name from `sys_dictionary`, not field label
3. **Use internal values** - Value from `sys_choice`, not display label

### Common Operators: `=`, `!=`, `^`, `^OR`, `LIKE`, `ISEMPTY`, `ISNOTEMPTY`

### Common Mistakes

| WRONG                  | CORRECT           | Why                              |
| ---------------------- | ----------------- | -------------------------------- |
| `Priority=1^EQ`        | `priority=1^EQ`   | Field name lowercase             |
| `priority=Critical^EQ` | `priority=1^EQ`   | Use internal value (1) not label |
| `state=Closed^EQ`      | `state=7^EQ`      | Use state number not label       |
| `Assigned to=...`      | `assigned_to=...` | Field name has underscore        |
| `priority=1`           | `priority=1^EQ`   | Must end with ^EQ                |

### Find Backend Names & Values

| What             | Where to Find                     | Use                         |
| ---------------- | --------------------------------- | --------------------------- |
| Field names      | `sys_dictionary` → Element column | `priority` (not `Priority`) |
| Choice values    | `sys_choice` → Value column       | `1` (not `Critical`)        |
| Reference fields | Use sys_id                        | `.toString()` for sys_id    |

### Examples

```typescript
// High priority AND in progress
condition: "priority=1^state=2^EQ";

// Hardware OR software category
condition: "category=hardware^ORcategory=software^EQ";

// Assigned to current user
condition: "assigned_to=javascript:gs.getUserID()^EQ";
```

## Important Properties for Condition-Based Rules

- `condition` - Encoded query (MUST end with `^EQ`)
- `device_type` - Device filter: `"browser"`, `"mobile"`, or `"tablet"`
- `active: true` - Enable the rule
- `overrides_user_preference: true` - Override manual selection

## Critical Validation

**Before creating rule:**

1. Verify view exists in `sys_ui_view` (use `runQuery`)
2. Test encoded query in list filter to verify syntax
3. Ensure field names and values are backend names (not labels)
4. Confirm query ends with `^EQ`
