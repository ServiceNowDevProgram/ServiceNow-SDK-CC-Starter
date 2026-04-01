# APPLICATION_MENU

### ApplicationMenu object

Create a top-level menu for an application [sys_app_application] in the Application Navigator.

#### Properties

| Name        | Type             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| $id         | String or Number | Required. A unique ID for the metadata object provided in the following format, where <value> is a string or number. $id: Now.ID[<value>] When you build the application, this ID is hashed into a unique sys_ID.                                                                                                                                                                                                                                                                                                                                                                    |
| title       | String           | Required. The label for the menu in the application navigator.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| active      | Boolean          | Flag that indicates whether the menu appears in the application navigator. Valid values: • true: The menu appears. • false: The menu is hidden. Default: true                                                                                                                                                                                                                                                                                                                                                                                                                        |
| roles       | Array            | A list of variable identifiers of Role objects or names of roles that can access the menu. For more information, see Role API - ServiceNow Fluent.                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| category    | Reference        | The variable identifier of a menu category [sys_app_category] that defines the navigation menu style. To define a menu category, use the Record API - ServiceNow Fluent. For general information about menu categories, see Customize menu categories.                                                                                                                                                                                                                                                                                                                               |
| hint        | String           | The tooltip text that appears when a user hovers over the menu.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| description | String           | Additional information about what the application does.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| name        | String           | An internal name to differentiate between applications with the same title.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| order       | Number           | The relative position of the application menu in the application navigator. Default: 100                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| $meta       | Object           | Metadata for the application metadata. With the installMethod property, you can map the application metadata to an output directory that loads only in specific circumstances. $meta: { installMethod: 'String' } Valid values for installMethod: • demo: Outputs the application metadata to the metadata/unload.demo directory to be installed with the application when the Load demo data option is selected. • first install: Outputs the application metadata to the metadata/unload directory to be installed only the first time an application is installed on an instance. |

### Application Module object

Create a child-level sub-menu for an application [sys_app_application] in the Application Navigator.

#### Properties

| Name                 | Type    | Description                                                                                                                                                                                       |
| -------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| title                | String  | The label for the module. Maximum length is 80 characters.                                                                                                                                        |
| active               | Boolean | Flag that indicates whether the module appears in the application navigator. Default: false                                                                                                       |
| application          | Object  | The applicationMenu object this module belongs to.                                                                                                                                                |
| assessment           | String  | sys_id reference to a record in 'asmt_metric_type' table.                                                                                                                                         |
| device_type          | String  | Options: '', 'any', 'browser', or 'mobile'.                                                                                                                                                       |
| filter               | String  | Conditions applied to list view for visibility in application navigator. Uses ServiceNow encoded query format.                                                                                    |
| hint                 | String  | The tooltip text that appears when a user hovers over the module. Maximum length is 255 characters.                                                                                               |
| link_type            | String  | Type of link. Options: '', 'SEPARATOR', 'TIMELINE', 'DETAIL', 'HTML', 'ASSESSMENT', 'LIST', 'FILTER', 'SCRIPT', 'CONTENT_PAGE', 'SEARCH', 'SURVEY', 'DOC_LINK', 'NEW', 'MAP', 'REPORT', 'DIRECT'. |
| map_page             | String  | sys_id reference to a record in 'cmn_map_page' table.                                                                                                                                             |
| mobile_title         | String  | Title displayed on mobile devices. Maximum length is 80 characters.                                                                                                                               |
| mobile_view_name     | String  | View name for mobile devices. Maximum length is 40 characters.                                                                                                                                    |
| name                 | String  | System name of table to link                                                                                                                                                                      |
| order                | Number  | The relative position of the module in the application navigator. Default: 100                                                                                                                    |
| override_menu_roles  | Boolean | Override menu roles. Default: false                                                                                                                                                               |
| query                | String  | Link type arguments. Maximum length is 3500 characters.                                                                                                                                           |
| require_confirmation | Boolean | Whether confirmation is required. Default: true                                                                                                                                                   |
| roles                | String  | A comma separated list of the names of roles that can access the menu.                                                                                                                            |
| uncancelable         | Boolean | Whether the action is uncancelable. Default: false                                                                                                                                                |
| view_name            | String  | Name of the view to use.                                                                                                                                                                          |
| window_name          | String  | Name of the window to use.                                                                                                                                                                        |
| report               | String  | Report identifier.                                                                                                                                                                                |
| timeline_page        | String  | sys_id reference to a record in 'cmn_timeline_page' table.                                                                                                                                        |

#### Example

```typescript
// Create a new top-level Application Menu (`sys_app_application`)
import { ApplicationMenu, Record } from "@servicenow/sdk/core";
const applicationMenu = ApplicationMenu({
  $id: Now.ID["my_app_menu"],
  title: "My App Menu",
  hint: "This is a hint",
  description: "This is a description",
  category: appCategory,
  roles: ["admin"],
  active: true
});

// Using Record API, create a new module (`sys_app_module`) which links to a table
const tableSubMenu = Record({
  $id: Now.ID["my_app_module_1"],
  table: "sys_app_module",
  data: {
    title: "My Table",
    application: applicationMenu.$id,
    link_type: "LIST", // For a list of records
    name: "x_snc_example_to_do", // Links to the x_snc_example_to_do table
    hint: "Link to my table",
    description: "This is a description",
    roles: "admin,itil",
    active: true,
    order: 100
  }
});

// Using Record API, create a new module (`sys_app_module`) which acts as a sub-folder, or separator
const separatorSubMenu = Record({
  $id: Now.ID["my_app_module_separator"],
  table: "sys_app_module",
  data: {
    title: "UI Pages",
    application: applicationMenu.$id,
    link_type: "SEPARATOR", // To create a sub-folder that will hold subsequent modules
    roles: "admin,itil",
    active: true,
    order: 200
  }
});

// Using Record API, create a new module (`sys_app_module`) which links to a UI Page
const uiPageSubMenu = Record({
  $id: Now.ID["my_app_module_2"],
  table: "sys_app_module",
  data: {
    title: "My UI Page",
    application: applicationMenu.$id,
    link_type: "DIRECT", // For URLs
    query: "my_ui_page.do", // relative link to the page
    hint: "Link to my UI Page",
    description: "This is a description",
    roles: "admin,itil",
    active: true,
    order: 300
  }
});

// The Application Menu category referenced is defined using the Record object:
import { Record } from "@servicenow/sdk/core";
export const appCategory = Record({
  table: "sys_app_category",
  $id: Now.ID[9],
  data: {
    name: "example",
    style: "border-color: #a7cded; background-color: #e3f3ff;"
  }
});
```
