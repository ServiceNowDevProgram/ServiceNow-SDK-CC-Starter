# LIST

<!-- Related skill: platform-view -->

### List object

Configure lists [sys_ui_list] and their views.

### Properties

| Name         | Type                         | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| table        | String                       | Required. The name of the table for the list.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| view         | Reference                    | Required. The variable identifier of the UI view [sys_ui_view] to apply to the list or default_view. To define a UI view, use the Record API - ServiceNow Fluent. You can also use the default view (default_view) if you import it: import { default_view } from '@servicenow/sdk/core'                                                                                                                                                                                                                                                                                             |
| columns      | array of ListElement objects | Required. See List Element                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| parent       | TableName                    | Name of the parent table on which the related list appears                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| relationship | sys_relationship             | The relationship (`sys_relationship`) to apply to the related list                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| $meta        | Object                       | Metadata for the application metadata. With the installMethod property, you can map the application metadata to an output directory that loads only in specific circumstances. $meta: { installMethod: 'String' } Valid values for installMethod: • demo: Outputs the application metadata to the metadata/unload.demo directory to be installed with the application when the Load demo data option is selected. • first install: Outputs the application metadata to the metadata/unload directory to be installed only the first time an application is installed on an instance. |

### List Element

| Field Name   | Type    | Mandatory | Options | Description                                                     |
| ------------ | ------- | --------- | ------- | --------------------------------------------------------------- |
| element      | string  | true      |         | Name of the field on the table to display, supports dot walking |
| maxValue     | boolean | false     |         |
| minValue     | boolean | false     |         |
| averageValue | boolean | false     |         |
| sum          | boolean | false     |         |
| position     | number  | false     |         | Element position in the display, defaults to array order        |

### Example

```javascript
import { List } from "@servicenow/sdk/core";
const app_task_view_list = List({
  table: "cmdb_ci_server",
  view: app_task_view,
  columns: [
    { element: "name", position: 0 },
    { element: "business_unit", position: 1 },
    { element: "vendor", position: 2 },
    { element: "cpu_type", position: 3 }
  ]
});
```

The UI view definition referenced is defined using the Record object:

```javascript
import { Record } from "@servicenow/sdk/core";

const app_task_view = Record({
  $id: Now.ID["app_task_view"],
  table: "sys_ui_view",
  data: {
    name: "app_task_view",
    title: "app_task_view"
  }
});
```
