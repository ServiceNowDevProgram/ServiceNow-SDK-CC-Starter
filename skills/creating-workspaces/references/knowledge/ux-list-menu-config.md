# UX_LIST_MENU_CONFIG

# UX List Menu Config Fluent Plugin - Knowledge Source

## Purpose

The `UxListMenuConfig` fluent plugin defines the navigation structure and list views for ServiceNow Workspaces. It organizes data into categories and lists, allowing users to quickly access different views of business data with filtering, column selection, and role-based visibility.

## Overview

UxListMenuConfig creates a hierarchical menu structure:

- **Categories**: Top-level groupings (e.g., "Incidents", "Problems")
- **Lists**: Individual views within categories with specific filters and columns
- **Applicability**: Controls which roles can see each list

This structure appears in the workspace's navigation panel, providing organized access to different data views.

## Basic Structure

```typescript
import { UxListMenuConfig, Applicability, Role } from "@servicenow/sdk/core";

// First, define roles and applicability
const userRole = Role({
  $id: Now.ID["example_user_role"],
  name: "x_snc_example.user",
  containsRoles: ["canvas_user"]
});

const applicability = Applicability({
  $id: Now.ID["example_applicability"],
  name: "Example Users",
  roles: [userRole]
});

// Then create the list configuration
const listConfig = UxListMenuConfig({
  $id: Now.ID["example_workspace_list_config"],
  name: "Example Workspace List Configuration",
  description: "List Menu Configuration for Example Workspace",
  categories: [
    {
      $id: Now.ID["incidents_category"],
      title: "Incidents",
      order: 10,
      lists: [
        {
          $id: Now.ID["incidents_open"],
          title: "Open",
          order: 10,
          condition: "active=true^EQ",
          table: "incident",
          columns: "number,short_description,priority,state",
          applicabilities: [
            {
              $id: Now.ID["incidents_open_applicability"],
              applicability: applicability
            }
          ]
        },
        {
          $id: Now.ID["incidents_all"],
          title: "All",
          order: 20,
          condition: "",
          table: "incident",
          columns: "number,short_description,priority,state",
          applicabilities: [
            {
              $id: Now.ID["incidents_all_applicability"],
              applicability: applicability
            }
          ]
        }
      ]
    }
  ]
});
```

## UxListMenuConfig Properties

| Name          | Type               | Required | Description                                              |
| ------------- | ------------------ | -------- | -------------------------------------------------------- |
| `$id`         | `Now.ID[string]`   | Yes      | Unique identifier for the list configuration             |
| `name`        | `string`           | Yes      | Name of the list configuration                           |
| `description` | `string`           | No       | Description of the list configuration                    |
| `active`      | `boolean`          | No       | Whether the list configuration is active (default: true) |
| `categories`  | `UxListCategory[]` | No       | Array of categories in the list configuration            |

## UxListCategory Properties

Categories group related lists together in the workspace navigation.

| Name          | Type             | Required | Description                                             |
| ------------- | ---------------- | -------- | ------------------------------------------------------- |
| `$id`         | `Now.ID[string]` | Yes      | Unique identifier for the category                      |
| `title`       | `string`         | Yes      | Display title for the category in the navigation menu   |
| `order`       | `number`         | No       | Sort order of the category (lower numbers appear first) |
| `active`      | `boolean`        | No       | Whether the category is active and visible              |
| `description` | `string`         | No       | Description of the category (for documentation)         |
| `lists`       | `UxList[]`       | Yes      | Array of lists in the category                          |

## UxList Properties

Lists define specific views of table data with filtering and column configuration.

| Name              | Type                  | Required | Description                                                                                       |
| ----------------- | --------------------- | -------- | ------------------------------------------------------------------------------------------------- |
| `$id`             | `Now.ID[string]`      | Yes      | Unique identifier for the list                                                                    |
| `title`           | `string`              | Yes      | Display title for the list in the navigation menu                                                 |
| `table`           | `string`              | Yes      | Name of the ServiceNow table this list displays records from                                      |
| `columns`         | `string`              | No       | Comma-separated list of column names to display (e.g., 'number,short_description,priority,state') |
| `condition`       | `string`              | No       | Encoded query condition to filter records (ServiceNow encoded query format)                       |
| `order`           | `number`              | No       | Sort order of the list within its category (lower numbers appear first)                           |
| `active`          | `boolean`             | No       | Whether the list is active and visible to users                                                   |
| `applicabilities` | `ListApplicability[]` | No       | Controls which roles can see this list                                                            |

## List Applicability

Each list can have applicability rules that control role-based visibility.

```typescript
applicabilities: [
  {
    $id: Now.ID["list_applicability_id"],
    applicability: applicability // Reference to an Applicability object
  }
];
```

**Important**: The `applicability` field references an `Applicability` object that defines which roles have access. This allows for fine-grained control over list visibility.

## Encoded Query Conditions

The `condition` property uses ServiceNow's encoded query format to filter records.

### Common Patterns

**Active records only:**

```typescript
condition: "active=true^EQ";
```

**All records (no filter):**

```typescript
condition: "";
```

**Assigned to current user:**

```typescript
condition: "assigned_toDYNAMIC90d1921e5f510100a9ad2572f2b477fe^EQ";
```

**High priority only:**

```typescript
condition: "priority=1^OR priority=2";
```

**Multiple conditions:**

```typescript
condition: "active=true^priority<=3^assigned_to!=NULL";
```

## Complete Example: Multi-Table Workspace

This example shows a workspace managing incidents and problems with multiple categories and filtered lists:

```typescript
import { UxListMenuConfig, Applicability, Role } from "@servicenow/sdk/core";

// Define roles
const userRole = Role({
  $id: Now.ID["itsm_workspace_user_role"],
  name: "x_snc_itsm.user",
  containsRoles: ["canvas_user"]
});

const adminRole = Role({
  $id: Now.ID["itsm_workspace_admin_role"],
  name: "x_snc_itsm.admin",
  containsRoles: ["canvas_admin"]
});

// Define applicability
const userApplicability = Applicability({
  $id: Now.ID["itsm_user_applicability"],
  name: "ITSM Users",
  roles: [userRole, adminRole]
});

// Create list configuration
const itsmListConfig = UxListMenuConfig({
  $id: Now.ID["itsm_workspace_list_config"],
  name: "ITSM Workspace List Configuration",
  description: "List Menu Configuration for ITSM Workspace",
  categories: [
    {
      $id: Now.ID["incidents_category"],
      title: "Incidents",
      order: 10,
      lists: [
        {
          $id: Now.ID["incidents_open"],
          title: "Open",
          order: 10,
          condition: "active=true^EQ",
          table: "incident",
          columns: "number,short_description,priority,state",
          applicabilities: [
            {
              $id: Now.ID["incidents_open_applicability"],
              applicability: userApplicability
            }
          ]
        },
        {
          $id: Now.ID["incidents_all"],
          title: "All",
          order: 20,
          condition: "",
          table: "incident",
          columns: "number,short_description,priority,state",
          applicabilities: [
            {
              $id: Now.ID["incidents_all_applicability"],
              applicability: userApplicability
            }
          ]
        },
        {
          $id: Now.ID["incidents_assigned_to_you"],
          title: "Assigned to you",
          order: 30,
          condition: "assigned_toDYNAMIC90d1921e5f510100a9ad2572f2b477fe^EQ",
          table: "incident",
          columns: "number,short_description,priority,state",
          applicabilities: [
            {
              $id: Now.ID["incidents_assigned_to_you_applicability"],
              applicability: userApplicability
            }
          ]
        }
      ]
    },
    {
      $id: Now.ID["problems_category"],
      title: "Problems",
      order: 20,
      lists: [
        {
          $id: Now.ID["problems_open"],
          title: "Open",
          order: 10,
          condition: "active=true^EQ",
          table: "problem",
          columns:
            "number,short_description,state,assignment_group,assigned_to",
          applicabilities: [
            {
              $id: Now.ID["problems_open_applicability"],
              applicability: userApplicability
            }
          ]
        },
        {
          $id: Now.ID["problems_all"],
          title: "All",
          order: 20,
          condition: "",
          table: "problem",
          columns:
            "number,short_description,state,assignment_group,assigned_to",
          applicabilities: [
            {
              $id: Now.ID["problems_all_applicability"],
              applicability: userApplicability
            }
          ]
        },
        {
          $id: Now.ID["problems_assigned_to_you"],
          title: "Assigned to you",
          order: 30,
          condition: "assigned_toDYNAMIC90d1921e5f510100a9ad2572f2b477fe^EQ",
          table: "problem",
          columns:
            "number,short_description,state,assignment_group,assigned_to",
          applicabilities: [
            {
              $id: Now.ID["problems_assigned_to_you_applicability"],
              applicability: userApplicability
            }
          ]
        }
      ]
    }
  ]
});
```

## Best Practices

### 1. Organize by Business Function

Group lists into categories that match how users think about their work. Categories should represent logical business groupings, not technical table names.

**Good:**

```typescript
categories: [
    { title: 'Incidents', ... },
    { title: 'Problems', ... },
    { title: 'Change Requests', ... },
]
```

**Avoid:**

```typescript
categories: [
    { title: 'Task Table Records', ... },
    { title: 'CMDB Items', ... },
]
```

### 2. Prioritize Important Views

Place the most commonly used lists at the top of each category using the `order` property. Lower order numbers appear first.

```typescript
lists: [
    { title: 'Open', order: 10, ... },           // Most used - first
    { title: 'Assigned to you', order: 20, ... }, // Personal view - second
    { title: 'All', order: 30, ... },            // Comprehensive - last
]
```

### 3. Use Clear Naming

Choose descriptive titles that clearly indicate what records will be shown. Avoid technical jargon.

**Good:**

- "Open"
- "Assigned to you"
- "High Priority"
- "Closed This Month"

**Avoid:**

- "Query_1"
- "active=true"
- "My View"

### 4. Optimize Column Selection

Include only the most relevant columns to avoid overwhelming users. Typical column counts: 4-6 columns.

**Good:**

```typescript
columns: "number,short_description,priority,state";
```

**Avoid (too many columns):**

```typescript
columns: "number,short_description,priority,state,assigned_to,assignment_group,opened_at,sys_created_by,sys_updated_by,category,subcategory";
```

### 5. Use Consistent Filtering Patterns

Apply similar filtering patterns across related lists for predictability.

**Example Pattern:**

```typescript
// Pattern: Open, All, Personal across all categories
categories: [
    {
        title: 'Incidents',
        lists: [
            { title: 'Open', condition: 'active=true^EQ', ... },
            { title: 'All', condition: '', ... },
            { title: 'Assigned to you', condition: 'assigned_toDYNAMIC...', ... },
        ],
    },
    {
        title: 'Problems',
        lists: [
            { title: 'Open', condition: 'active=true^EQ', ... },
            { title: 'All', condition: '', ... },
            { title: 'Assigned to you', condition: 'assigned_toDYNAMIC...', ... },
        ],
    },
]
```

### 6. Use Applicability for Role-Based Views

Apply applicability rules when different roles need different lists.

```typescript
// Create role-specific applicability
const managerApplicability = Applicability({
    $id: Now.ID['manager_applicability'],
    name: 'Managers Only',
    roles: [managerRole],
})

// Apply to sensitive lists
{
    $id: Now.ID['all_incidents_unfiltered'],
    title: 'All Incidents (Unfiltered)',
    table: 'incident',
    applicabilities: [
        {
            $id: Now.ID['all_incidents_unfiltered_applicability'],
            applicability: managerApplicability,
        },
    ],
}
```

## Common List View Patterns

### Standard CRUD Pattern

```typescript
lists: [
  { title: "Open", condition: "active=true^EQ", order: 10 },
  { title: "All", condition: "", order: 20 },
  {
    title: "Assigned to you",
    condition: "assigned_toDYNAMIC90d1921e5f510100a9ad2572f2b477fe^EQ",
    order: 30
  }
];
```

### Priority-Based Views

```typescript
lists: [
  { title: "Critical", condition: "priority=1", order: 10 },
  { title: "High", condition: "priority=2", order: 20 },
  { title: "All", condition: "", order: 30 }
];
```

### Time-Based Views

```typescript
lists: [
  {
    title: "Today",
    condition:
      "sys_created_onONToday@javascript:gs.beginningOfToday()@javascript:gs.endOfToday()",
    order: 10
  },
  {
    title: "This Week",
    condition:
      "sys_created_onONThis week@javascript:gs.beginningOfThisWeek()@javascript:gs.endOfThisWeek()",
    order: 20
  },
  { title: "All", condition: "", order: 30 }
];
```

## Integration with Workspace

The UxListMenuConfig must be referenced in the Workspace definition:

```typescript
const workspace = Workspace({
  $id: Now.ID["my_workspace"],
  title: "My Workspace",
  path: "my-workspace",
  tables: ["incident", "problem"],
  listConfig: listConfig // Reference the UxListMenuConfig here
});
```

## Troubleshooting

### Lists Not Appearing

**Problem**: Lists don't show up in the workspace navigation.

**Solutions**:

1. Check that `active` is true (or not set) for both the category and list
2. Verify the applicability includes the current user's roles
3. Ensure the listConfig is properly referenced in the Workspace

### Incorrect Filtering

**Problem**: Lists show wrong records or no records.

**Solutions**:

1. Verify the encoded query condition syntax
2. Test the condition in ServiceNow's list view directly
3. Check that the table name is correct

### Column Not Displaying

**Problem**: Specified columns don't appear in the list.

**Solutions**:

1. Verify column names match the table's field names exactly
2. Check that columns exist in the specified table
3. Ensure column names are comma-separated with no spaces

## Related Knowledge Sources

- **WORKSPACE**: High-level workspace architecture and integration
- **UX_WORKSPACE**: Workspace plugin configuration details
- **PAR_DASHBOARD**: Dashboard configuration for the landing page
