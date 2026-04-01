# PAR_DASHBOARD

# Dashboard Fluent Plugin - Knowledge Source

## Purpose

The `Dashboard` fluent plugin defines the landing page for ServiceNow Workspaces. Dashboards are organized into tabs, and each tab contains widgets that display data visualizations, headings, rich text, and other widgets. Dashboards provide users with at-a-glance insights into their business data.

## Overview

A Dashboard consists of:

- **Tabs**: Organize widgets into logical groupings
- **Widgets**: Individual components (charts, headings, rich texts, filters etc.)
- **Visibilities**: Link the dashboard to one or more workspaces
- **Permissions**: Control who can view or edit the dashboard

## Basic Structure

```typescript
import { Dashboard, Workspace } from "@servicenow/sdk/core";

// Assumes workspace is already defined
Dashboard({
  $id: Now.ID["my_dashboard"],
  name: "My Workspace Dashboard",
  tabs: [
    {
      $id: Now.ID["my_dashboard_tab1"],
      name: "Overview",
      widgets: [
        {
          $id: Now.ID["my_dashboard_widget_1"],
          component: "single-score",
          componentProps: {
            dataSources: [
              {
                label: "Incident",
                sourceType: "table",
                tableOrViewName: "incident",
                filterQuery: "",
                reportSourceSysId: "d94d6f62d7632100b96d45a3ce6103d2",
                id: "dGFibGVpbmNpZGVudDE3NjMzOTI0OTY5ODE="
              }
            ],
            headerTitle: "Open Incidents",
            metrics: [
              {
                dataSource: "dGFibGVpbmNpZGVudDE3NjMzOTI0OTY5ODE=",
                id: "ZEdGaWJHVnBibU5wWkdWdWRERTNOak16T1RJME9UWTVPREU9MTc2MzM5MjQ5Nzc5OQ==",
                aggregateFunction: "COUNT",
                axisId: "primary"
              }
            ],
            sortBy: "value"
          },
          height: 14,
          width: 14,
          position: { x: 0, y: 0 }
        }
      ]
    }
  ],
  visibilities: [
    {
      $id: Now.ID["my_dashboard_visibility1"],
      experience: myWorkspace // Reference to Workspace object
    }
  ],
  permissions: []
});
```

## Dashboard Properties

| Name           | Type                    | Required | Description                                           |
| -------------- | ----------------------- | -------- | ----------------------------------------------------- |
| `$id`          | `Now.ID[string]`        | Yes      | Unique identifier for the dashboard                   |
| `name`         | `string`                | Yes      | Display name for the dashboard                        |
| `tabs`         | `DashboardTab[]`        | Yes      | Array of tabs in the dashboard                        |
| `visibilities` | `DashboardVisibility[]` | Yes      | Controls which workspaces display this dashboard      |
| `permissions`  | `DashboardPermission[]` | Yes      | Access control for the dashboard (can be empty array) |

## Dashboard Tab Properties

| Name      | Type             | Required | Description                                       |
| --------- | ---------------- | -------- | ------------------------------------------------- |
| `$id`     | `Now.ID[string]` | Yes      | Unique identifier for the tab                     |
| `name`    | `string`         | Yes      | Display name for the tab                          |
| `widgets` | `Widget[]`       | Yes      | Array of widgets in this tab (can be empty array) |

## Widget Properties

| Name             | Type             | Required | Description                                                                             |
| ---------------- | ---------------- | -------- | --------------------------------------------------------------------------------------- |
| `$id`            | `Now.ID[string]` | Yes      | Unique identifier for the widget                                                        |
| `component`      | `string`         | Yes      | Type of visualization component (e.g., 'vertical-bar', 'single-score')                  |
| `componentProps` | `object`         | Yes      | Configuration object specific to the component type                                     |
| `height`         | `number`         | Yes      | Height of the widget on the grid (grid units)                                           |
| `width`          | `number`         | Yes      | Width of the widget on the grid (grid units)                                            |
| `position`       | `object`         | Yes      | Position of the widget in the grid with `x` and `y` properties (e.g., `{ x: 0, y: 0 }`) |

## Grid Layout System

Dashboards use a **48-point grid system** for positioning widgets.

### Grid Properties

- **Total Grid Width**: 48 units
- **Position Origin**: Top-left corner (x=0, y=0)
- **Units**: Abstract grid units (not pixels)

### Widget Positioning

```typescript
{
    height: 14,           // Height: 14 grid units
    width: 16,            // Width: 16 grid units
    position: { x: 0, y: 0 }  // Top-left corner position
}
```

### Common Layout Patterns

**Full Width Widget**:

```typescript
{ width: 48, height: 14, position: { x: 0, y: 0 } }
```

**Half Width (Side by Side)**:

```typescript
// Left widget
{ width: 24, height: 14, position: { x: 0, y: 0 } }

// Right widget
{ width: 24, height: 14, position: { x: 24, y: 0 } }
```

**Three Equal Columns**:

```typescript
// Left widget
{ width: 16, height: 14, position: { x: 0, y: 0 } }

// Center widget
{ width: 16, height: 14, position: { x: 16, y: 0 } }

// Right widget
{ width: 16, height: 14, position: { x: 32, y: 0 } }
```

**Stacked Widgets (Vertical)**:

```typescript
// Top widget
{ width: 48, height: 14, position: { x: 0, y: 0 } }

// Bottom widget
{ width: 48, height: 14, position: { x: 0, y: 14 } }  // y = previous y + previous height
```

## Widget Component Types

Widgets are generally split into two types:

### Visualizations

| Component Name      | Description                        | Data type |
| ------------------- | ---------------------------------- | --------- |
| area                | Area chart                         | Trend     |
| boxplot             | Boxplot chart                      | Group     |
| bubble              | Bubble chart                       | Group     |
| calendar-report     | Calendar report widget             |           |
| column              | Column chart                       | Trend     |
| dial                | Dial gauge                         | Simple    |
| donut               | Donut chart                        | Group     |
| gauge               | Gauge widget                       | Simple    |
| geomap              | Geographic map                     | Group     |
| heatmap             | Heatmap chart                      | Group     |
| horizontal-bar      | Horizontal bar chart               | Group     |
| indicator-scorecard | Indicator scorecard                |           |
| line                | Line chart                         | Trend     |
| list                | List widget for displaying records | List      |
| pareto              | Pareto chart                       | Group     |
| pie                 | Pie chart                          | Group     |
| pivot-table         | Pivot table                        | Group     |
| scatter             | Scatter plot                       | Trend     |
| semi-donut          | Semi donut chart                   | Group     |
| single-score        | Single score/metric display        | Simple    |
| spline              | Spline chart                       | Trend     |
| step                | Step chart                         | Trend     |
| vertical-bar        | Vertical bar chart                 | Group     |

Each visualization has specific data requirements, which are driven by the data type:

#### 1. Trend data type

Trend data type visualizations require `dataSources`, `metrics`, `trendBy` properties. `groupBy` property is optional.

##### Example

**Best For**: Showing how a metric trends over time

**Configuration**:

```typescript
{
    component: 'line',
    componentProps: {
        dataSources: [
            {
                label: 'Incident',
                sourceType: 'table',
                tableOrViewName: 'incident',
                filterQuery: '',
                id: 'data_source_1',
            },
        ],
        headerTitle: 'Incidents by State',
        metrics: [
            {
                dataSource: 'data_source_1',
                id: 'metric_1',
                aggregateFunction: 'COUNT',
                axisId: 'primary',
            },
        ],
        trendBy: {
            trendByFrequency: "year",
            trendByFields: [
                {
                    field: "sys_created_on",
                    metric: "metric_1"
                }
            ]
        },
    },
    height: 14,
    width: 17,
    position: { x: 0, y: 0 },
}
```

#### 2.Group data type

Group data type visualizations require `dataSources`, `metrics` and `grouBy`. `trendBy` is NOT supported in these visualizations.

##### Example

Displays a vertical bar chart.

**Best For**: Category comparisons, state distributions, priority breakdowns

**Configuration**:

```typescript
{
    component: 'vertical-bar',
    componentProps: {
        dataSources: [
            {
                label: 'Incident',
                sourceType: 'table',
                tableOrViewName: 'incident',
                filterQuery: '',
                id: 'data_source_1',
            },
        ],
        headerTitle: 'Incidents by State',
        metrics: [
            {
                dataSource: 'data_source_1',
                id: 'metric_1',
                aggregateFunction: 'COUNT',
                axisId: 'primary',
            },
        ],
        groupBy: [
            {
                groupBy: [
                    {
                        dataSource: 'data_source_1',
                        groupByField: 'state',
                    },
                ],
                maxNumberOfGroups: 10,
                showOthers: false,
            },
        ],
        sortBy: 'value',
    },
    height: 14,
    width: 17,
    position: { x: 0, y: 0 },
}
```

#### Simple data type

Simple data type visualizations require `dataSources` and `metrics` properties. `groupBy` and `trendBy` are NOT supported in these visualizations.

##### Example

Displays a single metric or count.

**Best For**: Key performance indicators, total counts

**Configuration**:

```typescript
{
    component: 'single-score',
    componentProps: {
        dataSources: [
            {
                label: 'Incident',
                sourceType: 'table',
                tableOrViewName: 'incident',
                filterQuery: '',
                id: 'data_source_1',
            },
        ],
        headerTitle: 'Open Incidents',
        metrics: [
            {
                dataSource: 'data_source_1',
                id: 'metric_1',
                aggregateFunction: 'COUNT',
                axisId: 'primary',
            },
        ]
    },
    height: 14,
    width: 14,
    position: { x: 0, y: 0 },
}
```

### Supporting widgets

| Component name | Description         |
| -------------- | ------------------- |
| heading        | Heading/header text |
| image          | Image widget        |
| rich-text      | Rich text widget    |

#### Heading

**Best for:** Displaying simple text on the dashboard (headings)

**Example:**

```typescript
{
    component: 'heading',
    componentProps: {
        label: 'Title',  // Text to be displayed
        level: '1',      // Level of H tag to use
        align: 'start'   // Horizontal alignment
    },
    height: 14,
    width: 14,
    position: { x: 0, y: 0 },
}
```

#### Rich text

**Best for:** Displaying html in a widget.

**Example:**

```typescript
{
    component: 'rich-text',
    componentProps: {
        html: '<p>The world works with ServiceNow</p>' // HTML to be rendered in the component (scripts are not allowed)
    },
    height: 14,
    width: 14,
    position: { x: 0, y: 0 },
}
```

## Component Props Structure

### Common Properties

All widget `componentProps` share these common elements:

#### dataSources

Array defining where the widget gets its data.

```typescript
dataSources: [
  {
    label: "Incident", // Human-readable label
    sourceType: "table", // Type of data source
    tableOrViewName: "incident", // ServiceNow table name
    filterQuery: "", // Optional encoded query filter
    id: "data_source_1" // Unique data source ID
  }
];
```

#### headerTitle

Display title for the widget.

```typescript
headerTitle: "Open Incidents";
```

#### metrics

Array defining what to measure from the data source.

```typescript
metrics: [
    {
        dataSource: 'data_source_1',  // Must match dataSource id
        id: 'metric_1',  // Unique metric ID
        aggregateFunction: 'AVG',                          // COUNT, SUM, AVG, MIN, MAX, COUNT_DISTINCT
        aggregateField: 'business_duration'  // Field to be used for aggregation.  Not required for COUNT aggregation
        axisId: 'primary',                                   // Which axis to display the series
    },
]
```

#### groupBy

Defines how to group and organize data.

```typescript
groupBy: [
  {
    groupBy: [
      {
        dataSource: "data_source_1", // Must match dataSource id
        groupByField: "state" // Field to group by
      }
    ],
    maxNumberOfGroups: 10, // Maximum categories to show
    showOthers: false // Show "Others" category
  }
];
```

#### trendBy

Defines trend configuration for trend charts

```typescript
{
  "trendByFrequency": "year", // Frequency of the trend (date, week, month, year)
  "trendByFields": [
    {
      "field": "sys_created_on", // Field to trend on (field from the table of the dataSource)
      "metric": "metric_1" // ID of the metric
    }
  ]
}
```

#### sortBy

How to sort the data.

```typescript
sortBy: "value"; // or 'label', 'field'
```

## Dashboard Visibility

Visibilities link the dashboard to one or more workspaces.

```typescript
visibilities: [
  {
    $id: Now.ID["dashboard_visibility_1"],
    experience: myWorkspace // Reference to Workspace object
  }
];
```

**Important**:

- The workspace must be defined before the dashboard
- The workspace should be exported if defined in a different file
- Multiple workspaces can share the same dashboard by adding multiple visibility entries

## Complete Dashboard Example

This example shows a complete dashboard with multiple widget types:

```typescript
import { Dashboard, Workspace } from "@servicenow/sdk/core";

// Assumes incidentWorkspace is defined and exported
Dashboard({
  $id: Now.ID["incident_management_dashboard"],
  name: "Incident Management Dashboard",
  tabs: [
    {
      $id: Now.ID["incident_overview_tab"],
      name: "Overview",
      widgets: [
        // Single Score: Total Open Incidents
        {
          $id: Now.ID["total_open_incidents_widget"],
          component: "single-score",
          componentProps: {
            dataSources: [
              {
                label: "Incident",
                sourceType: "table",
                tableOrViewName: "incident",
                filterQuery: "",
                reportSourceSysId: "d94d6f62d7632100b96d45a3ce6103d2",
                id: "dGFibGVpbmNpZGVudDE3NjMzOTI0OTY5ODE="
              }
            ],
            headerTitle: "Open Incidents",
            metrics: [
              {
                dataSource: "dGFibGVpbmNpZGVudDE3NjMzOTI0OTY5ODE=",
                id: "ZEdGaWJHVnBibU5wWkdWdWRERTNOak16T1RJME9UWTVPREU9MTc2MzM5MjQ5Nzc5OQ==",
                aggregateFunction: "COUNT",
                axisId: "primary"
              }
            ],
            sortBy: "value"
          },
          height: 14,
          width: 14,
          position: { x: 0, y: 0 }
        },
        // Vertical Bar: Incidents by Priority
        {
          $id: Now.ID["incidents_by_priority_widget"],
          component: "vertical-bar",
          componentProps: {
            dataSources: [
              {
                label: "Incident",
                sourceType: "table",
                tableOrViewName: "incident",
                filterQuery: "",
                reportSourceSysId: "d94d6f62d7632100b96d45a3ce6103d2",
                id: "dGFibGVpbmNpZGVudDE3NjI3OTU4OTg3NDA="
              }
            ],
            headerTitle: "Incidents by Priority",
            metrics: [
              {
                dataSource: "dGFibGVpbmNpZGVudDE3NjI3OTU4OTg3NDA=",
                id: "ZEdGaWJHVnBibU5wWkdWdWRERTNOakkzT1RVNE9UZzNOREE9MTc2Mjc5NTg5OTM5OA==",
                aggregateFunction: "COUNT",
                axisId: "primary"
              }
            ],
            groupBy: [
              {
                groupBy: [
                  {
                    dataSource: "dGFibGVpbmNpZGVudDE3NjI3OTU4OTg3NDA=",
                    groupByField: "priority"
                  }
                ],
                maxNumberOfGroups: 10,
                numberOfGroupsBasedOn: "NO_OF_GROUP_BASED_ON_PER_METRIC",
                showOthers: false,
                disableRanges: false
              }
            ],
            sortBy: "value"
          },
          height: 14,
          width: 17,
          position: { x: 14, y: 0 }
        },
        // Vertical Bar: Incidents by State
        {
          $id: Now.ID["incidents_by_state_widget"],
          component: "vertical-bar",
          componentProps: {
            dataSources: [
              {
                label: "Incident",
                sourceType: "table",
                tableOrViewName: "incident",
                filterQuery: "",
                reportSourceSysId: "d94d6f62d7632100b96d45a3ce6103d2",
                id: "dGFibGVpbmNpZGVudDE3NjI3OTU4OTg3NDA="
              }
            ],
            headerTitle: "Incidents by State",
            metrics: [
              {
                dataSource: "dGFibGVpbmNpZGVudDE3NjI3OTU4OTg3NDA=",
                id: "ZEdGaWJHVnBibU5wWkdWdWRERTNOakkzT1RVNE9UZzNOREE9MTc2Mjc5NTg5OTM5OA==",
                aggregateFunction: "COUNT",
                axisId: "primary"
              }
            ],
            groupBy: [
              {
                groupBy: [
                  {
                    dataSource: "dGFibGVpbmNpZGVudDE3NjI3OTU4OTg3NDA=",
                    groupByField: "state"
                  }
                ],
                maxNumberOfGroups: 10,
                numberOfGroupsBasedOn: "NO_OF_GROUP_BASED_ON_PER_METRIC",
                showOthers: false,
                disableRanges: false
              }
            ],
            sortBy: "value"
          },
          height: 14,
          width: 17,
          position: { x: 31, y: 0 }
        }
      ]
    },
    {
      $id: Now.ID["incident_trends_tab"],
      name: "Trends",
      widgets: [] // Can be populated later or left empty
    }
  ],
  visibilities: [
    {
      $id: Now.ID["incident_dashboard_visibility"],
      experience: incidentWorkspace
    }
  ],
  permissions: []
});
```

## Best Practices

### 1. Follow the Examples Closely

The dashboard configuration is complex. **Use the provided examples as templates** and modify only the necessary parts:

- Change `headerTitle` to match your data
- Update `tableOrViewName` to your table
- Modify `groupByField` to your desired field
- Modify `trendByFields` and `trendByFrequency` to your desired options
- Adjust layout positions (position, width, height) as needed

### 2. Keep IDs Consistent

Generate unique IDs but maintain consistent patterns:

- Id's for `dataSources` and `metrics` should be following the pattern `data_source_1` and `metric_1`
- Create unique `$id` values for your widgets using `Now.ID['descriptive_name']`

### 3. Standard Dashboard Layout

For most workspaces, use a three-widget layout:

- **Left**: Single-score widget (width: 14)
- **Center**: Vertical-bar widget (width: 17)
- **Right**: Vertical-bar widget (width: 17)

```typescript
widgets: [
    { /* single-score */, width: 14, position: { x: 0, y: 0 } },
    { /* vertical-bar */, width: 17, position: { x: 14, y: 0 } },
    { /* vertical-bar */, width: 17, position: { x: 31, y: 0 } },
]
```

### 4. Common Visualization Patterns

**Total Count + Two Breakdowns**:

```typescript
// Widget 1: Total count (single-score)
// Widget 2: By priority (vertical-bar)
// Widget 3: By state (vertical-bar)
```

**Multiple Metrics**:

```typescript
// Widget 1: Open items (single-score)
// Widget 2: Closed items (single-score)
// Widget 3: Combined view (vertical-bar)
```

### 5. Multiple Tabs for Organization

Use multiple tabs when you have different perspectives on the data:

```typescript
tabs: [
  {
    name: "Overview",
    widgets: [
      /* summary widgets */
    ]
  },
  {
    name: "Trends",
    widgets: [
      /* time-based widgets */
    ]
  },
  {
    name: "Details",
    widgets: [
      /* detailed breakdowns */
    ]
  }
];
```

### 6. Aggregate Functions

Choose appropriate aggregate functions:

- **COUNT**: Total number of records
- **SUM**: Sum of numeric field values
- **AVG**: Average of numeric field values
- **MIN**: Minimum value
- **MAX**: Maximum value
- **COUNT_DISTINCT**: Total number of distinct values

## Common Widget Configurations

### Count of Records

```typescript
{
    component: 'single-score',
    componentProps: {
        dataSources: [{ /* table config */ }],
        headerTitle: 'Total Records',
        metrics: [
            {
                aggregateFunction: 'COUNT',  // Count all records
                // ...
            },
        ],
    },
}
```

### Group by Field

```typescript
{
    component: 'vertical-bar',
    componentProps: {
        dataSources: [{ /* table config */ }],
        headerTitle: 'Records by Category',
        groupBy: [
            {
                groupBy: [
                    {
                        groupByField: 'category',  // Replace with your field
                    },
                ],
            },
        ],
    },
}
```

### Filter Data Source

```typescript
dataSources: [
  {
    tableOrViewName: "incident",
    filterQuery: "active=true^priority=1" // Only high-priority active incidents
    // ...
  }
];
```

## Integration with Workspace

The complete integration pattern:

```typescript
// 1. Define and export workspace
export const myWorkspace = Workspace({
  $id: Now.ID["my_workspace"],
  title: "My Workspace",
  path: "my-workspace",
  tables: ["incident"],
  listConfig: myListConfig
});

// 2. Create dashboard referencing workspace
Dashboard({
  $id: Now.ID["my_dashboard"],
  name: "My Dashboard",
  tabs: [
    {
      $id: Now.ID["my_dashboard_tab1"],
      name: "Overview",
      widgets: [
        // ... widgets
      ]
    }
  ],
  visibilities: [
    {
      $id: Now.ID["my_dashboard_visibility"],
      experience: myWorkspace // Reference the workspace
    }
  ],
  permissions: []
});
```

## Troubleshooting

### Dashboard Not Appearing

**Problem**: Dashboard doesn't show on workspace landing page.

**Solutions**:

1. Verify `visibilities` references the correct workspace
2. Confirm the URL pattern `/now/{path}/{landingPath}` for the workspace is correct

### Widget Not Displaying Data

**Problem**: Widget shows no data or error.

**Solutions**:

1. Verify `tableOrViewName` matches an existing table
2. Check `dataSource.id` is consistent across dataSources, metrics, and groupBy
3. Ensure `filterQuery` syntax is correct

### Layout Issues

**Problem**: Widgets overlap or don't fit correctly.

**Solutions**:

1. Verify total width doesn't exceed 48 units
2. Check position.x + width ≤ 48 for each widget
3. Ensure position.y coordinates don't overlap for widgets at same position.x
4. Use the standard layout patterns provided

### ID Mismatch Errors

**Problem**: Errors about mismatched IDs.

**Solutions**:

1. Ensure `dataSource.id` in dataSources matches references in metrics and groupBy
2. Only change `$id` values for dashboard, tabs, and widgets

## Related Knowledge Sources

- **WORKSPACE**: High-level workspace architecture and integration
- **UX_WORKSPACE**: Workspace plugin configuration details
- **UX_LIST_MENU_CONFIG**: List menu configuration for workspace navigation
