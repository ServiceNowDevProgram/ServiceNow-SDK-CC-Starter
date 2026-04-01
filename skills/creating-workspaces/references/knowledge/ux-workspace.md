# UX_WORKSPACE

# Workspace Fluent Plugin - Knowledge Source

## Purpose

The `Workspace` fluent plugin manages a complete business entity management solution in ServiceNow. It automatically generates landing pages, list views, and detail forms following ServiceNow's standard UX patterns.

## Overview

A Workspace provides:

- **Automatic Page Generation**: Creates landing, list, and detail pages
- **Standardized Navigation**: Consistent user experience across business entities
- **Table Integration**: Connects to one or more ServiceNow tables
- **List Configuration**: Integrates with UxListMenuConfig for navigation
- **Security**: Works with ACLs for route-level access control

## Basic Structure

```typescript
import { Workspace, UxListMenuConfig } from "@servicenow/sdk/core";

// Assumes listConfig is already defined (see UX_LIST_MENU_CONFIG knowledge source)
const workspace = Workspace({
  $id: Now.ID["my_example_workspace"],
  title: "My Example Workspace",
  path: "my-example",
  tables: ["incident", "problem", "user"],
  listConfig: listConfig
});
```

## Properties

| Name          | Type               | Required | Description                                                                                                       |
| ------------- | ------------------ | -------- | ----------------------------------------------------------------------------------------------------------------- |
| `$id`         | `Now.ID[string]`   | Yes      | Unique identifier for the Workspace                                                                               |
| `title`       | `string`           | Yes      | Display name for the Workspace that appears in navigation and headers                                             |
| `path`        | `string`           | Yes      | URL path segment for the Workspace (use kebab-case, e.g., 'incident-management')                                  |
| `landingPath` | `string`           | No       | URL path segment for the landing page. Uses 'home' if not specified, should not set unless specifically requested |
| `active`      | `boolean`          | No       | Whether the Workspace is active and accessible to users, defaults to true if not specified                        |
| `tables`      | `string[]`         | Yes      | Array of ServiceNow table names that this Workspace manages                                                       |
| `listConfig`  | `UxListMenuConfig` | Yes      | Reference to a UxListMenuConfig object defining the navigation structure                                          |

## Workspace URL Structure

When a Workspace is created, the URL follows this pattern:

```
/now/{path}/{landingPath}
```

### URL Components

**Base Path**: `/now/` (fixed prefix for all workspaces)

**Workspace Path**: The `path` property value

**Landing Path**: The `landingPath` property value, defaults to 'home' if not specified

### URL Examples

**Example 1**: Basic incident workspace

```typescript
Workspace({
  path: "incident-management"
  // ...
});
```

**Resulting URL**: `/now/incident-management/home`

**Example 2**: Custom landing path

```typescript
Workspace({
  path: "hr-services",
  landingPath: "main"
  // ...
});
```

**Resulting URL**: `/now/hr-services/main`

## Integration with UxListMenuConfig

The Workspace requires a UxListMenuConfig to define its navigation structure.

```typescript
import {
  Workspace,
  UxListMenuConfig,
  Applicability,
  Role
} from "@servicenow/sdk/core";

// 1. Define roles and applicability
const userRole = Role({
  $id: Now.ID["incident_user_role"],
  name: "x_snc_incident.user",
  containsRoles: ["canvas_user"]
});

const applicability = Applicability({
  $id: Now.ID["incident_applicability"],
  name: "Incident Users",
  roles: [userRole]
});

// 2. Create list configuration
const incidentListConfig = UxListMenuConfig({
  $id: Now.ID["incident_list_config"],
  name: "Incident List Configuration",
  description: "Navigation for Incident Workspace",
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
        }
      ]
    }
  ]
});

// 3. Create workspace with list configuration
const incidentWorkspace = Workspace({
  $id: Now.ID["incident_management_workspace"],
  title: "Incident Management",
  path: "incident-management",
  tables: ["incident", "user"],
  listConfig: incidentListConfig // Reference the list config
});
```

## Integration with ACL

Workspaces require Access Control Lists (ACLs) to secure the workspace routes.

### ACL Pattern

```typescript
import { Acl } from "@servicenow/sdk/core";

Acl({
  $id: Now.ID["workspace_name_ACL"],
  localOrExisting: "Existing",
  type: "ux_route",
  operation: "read",
  roles: ["x_scope.user", "x_scope.admin"], // Match your application's roles
  table: "now",
  field: "<workspace_path>.*" // CRITICAL: Must match the workspace path property
});
```

### ACL Field Property

**CRITICAL**: The ACL `field` property must match the workspace `path` property with a wildcard:

```typescript
// Workspace definition
const workspace = Workspace({
  path: "incident-management"
  // ...
});

// Corresponding ACL
Acl({
  field: "incident-management.*" // Matches workspace path + .*
  // ...
});
```

**Pattern**: `{workspace.path}.*`

### Common ACL Mistake

**❌ Incorrect**:

```typescript
// Workspace
path: "incident-management";

// ACL
field: "incidents.*"; // Does NOT match!
```

**✅ Correct**:

```typescript
// Workspace
path: "incident-management";

// ACL
field: "incident-management.*"; // Matches exactly
```

## Complete Example

This example shows a complete workspace implementation with all required components:

```typescript
import {
  Acl,
  Applicability,
  Role,
  Workspace,
  UxListMenuConfig
} from "@servicenow/sdk/core";

// 1. Define Roles
const userRole = Role({
  $id: Now.ID["asset_workspace_user_role"],
  name: "x_snc_asset.user",
  containsRoles: ["canvas_user"]
});

const adminRole = Role({
  $id: Now.ID["asset_workspace_admin_role"],
  name: "x_snc_asset.admin",
  containsRoles: ["canvas_admin"]
});

// 2. Define Applicability
const assetApplicability = Applicability({
  $id: Now.ID["asset_applicability"],
  name: "Asset Management Users",
  roles: [userRole, adminRole]
});

// 3. Create List Configuration
const assetListConfig = UxListMenuConfig({
  $id: Now.ID["asset_management_list_config"],
  name: "Asset Management List Configuration",
  description: "Navigation for Asset Management Workspace",
  categories: [
    {
      $id: Now.ID["assets_category"],
      title: "Assets",
      order: 10,
      lists: [
        {
          $id: Now.ID["assets_active"],
          title: "Active",
          order: 10,
          condition: "install_status=1",
          table: "alm_asset",
          columns: "asset_tag,display_name,model_category,assigned_to",
          applicabilities: [
            {
              $id: Now.ID["assets_active_applicability"],
              applicability: assetApplicability
            }
          ]
        },
        {
          $id: Now.ID["assets_all"],
          title: "All",
          order: 20,
          condition: "",
          table: "alm_asset",
          columns: "asset_tag,display_name,model_category,assigned_to",
          applicabilities: [
            {
              $id: Now.ID["assets_all_applicability"],
              applicability: assetApplicability
            }
          ]
        }
      ]
    }
  ]
});

// 4. Create Workspace
export const assetWorkspace = Workspace({
  $id: Now.ID["asset_management_workspace"],
  title: "Asset Management",
  path: "asset-management",
  tables: ["alm_asset", "cmdb_ci", "user"],
  listConfig: assetListConfig
});

// 5. Create ACL
Acl({
  $id: Now.ID["asset_management_workspace_ACL"],
  localOrExisting: "Existing",
  type: "ux_route",
  operation: "read",
  roles: ["x_snc_asset.user", "x_snc_asset.admin"],
  table: "now",
  field: "asset-management.*" // Matches workspace path
});
```

## Integration with Dashboard

Dashboards are linked to workspaces through the `visibilities` property:

```typescript
import { Dashboard } from "@servicenow/sdk/core";

Dashboard({
  $id: Now.ID["my_dashboard"],
  name: "My Dashboard",
  tabs: [
    // ... tabs configuration
  ],
  visibilities: [
    {
      $id: Now.ID["my_dashboard_visibility"],
      experience: myWorkspace // Reference to Workspace object
    }
  ],
  permissions: []
});
```

**Important**: The workspace must be defined and accessible (usually exported) before it can be referenced in the Dashboard.

## Common Patterns

### Single Table Workspace

For managing one primary business entity:

```typescript
const requestWorkspace = Workspace({
  $id: Now.ID["request_workspace"],
  title: "Service Requests",
  path: "requests",
  tables: ["sc_request", "sc_req_item", "user"],
  listConfig: requestListConfig
});
```

### Multi-Table Workspace

For managing related business entities:

```typescript
const itsmWorkspace = Workspace({
  $id: Now.ID["itsm_workspace"],
  title: "IT Service Management",
  path: "itsm",
  tables: ["incident", "problem", "change_request", "user", "sys_user_group"],
  listConfig: itsmListConfig
});
```

## Troubleshooting

### Workspace Not Accessible

**Problem**: Users cannot access the workspace.

**Solutions**:

1. Verify if `active: false` is set
2. Check ACL configuration matches workspace path
3. Verify users have required roles in ACL
4. Confirm listConfig is properly defined
5. Confirm user is using the correct URL

### Wrong URL Generated

**Problem**: Workspace URL doesn't match expectations.

**Solutions**:

1. Check `path` property (should be kebab-case)
2. Verify `landingPath` property (should also be kebab-case, and defaults to 'home')
3. Remember URL pattern: `/now/{path}/{landingPath}`

## Related Knowledge Sources

- **WORKSPACE**: High-level workspace architecture and orchestration
- **UX_LIST_MENU_CONFIG**: Detailed list menu configuration
- **PAR_DASHBOARD**: Dashboard configuration for landing pages
