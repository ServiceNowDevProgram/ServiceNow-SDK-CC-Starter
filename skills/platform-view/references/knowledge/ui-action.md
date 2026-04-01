# UI_ACTION

<!-- Related skill: platform-view -->

## Examples

Creating a basic UI Action:

### 1. A banner UI Action on all test_table forms, refreshes the page when clicked

```typescript
import "@servicenow/sdk/global";
import { UiAction } from "@servicenow/sdk/core";

export const ua1 = UiAction({
  $id: Now.ID["foo-bar"],
  table: "test_table",
  name: "foo-bar",
  actionName: "foo-bars",
  active: true,
  hint: "foobar button to refresh page",
  showUpdate: true,
  showInsert: true,
  form: {
    showButton: true,
    style: "primary"
  },
  script: `window.location.reload();`
});
```

### 2. A banner and bottom UI Action on test_table list, adds info message when clicked

```typescript
import "@servicenow/sdk/global";
import { UiAction } from "@servicenow/sdk/core";

export const ua2 = UiAction({
  $id: Now.ID["foo-bar2"],
  table: "test_table",
  name: "foo-bar2",
  actionName: "foo-bars2",
  active: true,
  hint: "foobar button to add info message",
  showUpdate: true,
  list: {
    showBannerButton: true,
    showButton: true,
    style: "primary"
  },
  script: `gs.addInfoMessage('button pressed');`
});
```

### 3. UI Action on list and form, should only show up if condition is true, user has admin role, and allows multi-row record updates.

```typescript
import "@servicenow/sdk/global";
import { UiAction } from "@servicenow/sdk/core";

export const ua3 = UiAction({
  $id: Now.ID["foobar3"],
  table: "test_table",
  name: "foobar3",
  actionName: "foobar3",
  active: true,
  showUpdate: true,
  showInsert: true,
  showMultipleUpdate: true,
  list: {
    showButton: true,
    style: "primary"
  },
  form: {
    showButton: true,
    style: "primary"
  },
  condition: `current.canWrite();`,
  roles: ["admin"]
});
```

### 4. Client side Workspace-Compatible UI Action only on form

```typescript
import "@servicenow/sdk/global";
import { UiAction } from "@servicenow/sdk/core";

export const workspace_action = UiAction({
  $id: Now.ID["workspace-action4"],
  table: "test_table",
  name: "TEST TEST 3",
  actionName: "TEST_TEST_3",
  active: true,
  showUpdate: true,
  form: {
    showButton: true,
    style: "primary",
    showLink: false,
    showContextMenu: false
  },
  client: {
    isClient: true,
    isUi16Compatible: true
  },
  roles: ["itil", "knowledge"],
  list: {
    showButton: false,
    showLink: false,
    showContextMenu: false,
    showListChoice: false,
    showBannerButton: false,
    showSaveWithFormButton: false
  },
  messages: [],
  showInsert: true,
  includeInViews: [],
  excludeFromViews: [],
  workspace: {
    showFormButtonV2: false,
    showFormMenuButtonV2: false,
    isConfigurableWorkspace: false
  },
  script: Now.include("../../client/test.js")
});
```

## UI Action Properties Reference

| Property             | Type                 | Required | Description                                                                          |
| -------------------- | -------------------- | -------- | ------------------------------------------------------------------------------------ |
| `$id`                | `Now.ID[string]`     | Yes      | Unique identifier for the UI Action                                                  |
| `table`              | `string`             | Yes      | The ServiceNow table this UI Action is associated with                               |
| `name`               | `string`             | Yes      | Display name of the UI Action                                                        |
| `actionName`         | `string`             | Yes      | Unique identifier that can be used in scripts to reference this UI Action            |
| `active`             | `boolean`            | No       | If true, the UI Action is available; if false, it is inactive                        |
| `showInsert`         | `boolean`            | No       | Shows the action on the form in insert mode (before the record is saved)             |
| `showUpdate`         | `boolean`            | No       | Shows the action on the form in update mode (after the record is saved)              |
| `showQuery`          | `boolean`            | No       | Controls whether the UI Action is visible on a list when a filter is applied         |
| `showMultipleUpdate` | `boolean`            | No       | When multiple records are selected, this UI Action can be triggered on those records |
| `condition`          | `string`             | No       | A script or condition that controls the visibility of the UI Action                  |
| `script`             | `string`             | No       | Script to execute when the action is triggered                                       |
| `hint`               | `string`             | No       | Tooltip that appears when hovering over the UI Action button                         |
| `order`              | `number`             | No       | Determines the position of the button/link in the UI                                 |
| `isolateScript`      | `boolean`            | No       | If true, script runs in an isolated script scope                                     |
| `roles`              | `(string or Role)[]` | No       | Roles that can see or execute this UI Action                                         |
| `includeInViews`     | `string[]`           | No       | Views in which the UI Action should appear                                           |
| `excludeFromViews`   | `string[]`           | No       | Views from which the UI Action should be excluded                                    |

### Form Properties

| Property               | Type      | Description                                           |
| ---------------------- | --------- | ----------------------------------------------------- |
| `form.showButton`      | `boolean` | Adds a button to the form view                        |
| `form.showLink`        | `boolean` | Adds a link to the form view                          |
| `form.showContextMenu` | `boolean` | Adds the action to the right-click menu               |
| `form.style`           | `string`  | Button style: 'primary', 'destructive', or 'unstyled' |

### List Properties

| Property                      | Type      | Description                                           |
| ----------------------------- | --------- | ----------------------------------------------------- |
| `list.showButton`             | `boolean` | Adds a button to the list view                        |
| `list.showLink`               | `boolean` | Adds a link to the list view                          |
| `list.showContextMenu`        | `boolean` | Adds the action to the right-click menu               |
| `list.style`                  | `string`  | Button style: 'primary', 'destructive', or 'unstyled' |
| `list.showListChoice`         | `boolean` | Adds the action to dropdown menus of choice fields    |
| `list.showBannerButton`       | `boolean` | Adds a button in the list view banner                 |
| `list.showSaveWithFormButton` | `boolean` | Ensures form is saved before executing the action     |

### Client Properties

| Property                  | Type      | Description                                         |
| ------------------------- | --------- | --------------------------------------------------- |
| `client.isClient`         | `boolean` | If true, script runs on client; if false, on server |
| `client.isUi11compatible` | `boolean` | UI Action is compatible with UI11                   |
| `client.isUi16Compatible` | `boolean` | UI Action is compatible with UI16                   |
| `client.onClick`          | `string`  | JavaScript to run when action is clicked            |

### Workspace Properties

| Property                            | Type      | Description                                  |
| ----------------------------------- | --------- | -------------------------------------------- |
| `workspace.isConfigurableWorkspace` | `boolean` | Enables UI Action for Configurable Workspace |
| `workspace.showFormButtonV2`        | `boolean` | Uses V2 button rendering                     |
| `workspace.showFormMenuButtonV2`    | `boolean` | Uses V2 menu rendering                       |
| `workspace.clientScriptV2`          | `string`  | V2 client script model code                  |
