# SCRIPT_INCLUDE

## Basic Script Include Example

### 1 - Create the ScriptInclude Definition

```typescript
// ./src/fluent/script-includes/math-utils.now.ts

import '@servicenow/sdk/global';
import { ScriptInclude } from `@servicenow/sdk/core`

export const MathUtils = ScriptInclude({
    $id: Now.ID['MathUtils'],
    name: 'MathUtils',
    script: Now.include('../../server/script-includes/math-utils.js') // Always use Now.include to link the JS file. Avoid inline source.
    description: 'some description',
    apiName: 'x_scope.MathUtilsInclude',
    callerAccess: 'tracking',
    clientCallable: true,
    mobileCallable: true,
    sandboxCallable: true,
    accessibleFrom: 'public',
    active: true,
})
```

### 2 - Write the JavaScript Logic

```javascript
// ./src/server/script-includes/math-utils.js

var MathUtils = Class.create();
MathUtils.prototype = {
  initialize: function () {},

  multiply: function (a, b) {
    return a * b;
  },

  type: "MathUtils" // IMPORTANT: must match the class name
};
```

> **Tip** — when you change the file path, update the relative path in `Now.include()` or the build will fail.

## Advanced Script Include Example

#### 1 - Create ScriptInclude Definition

```typescript
// ./src/fluent/script-includes/todo-ajax.now.ts

import '@servicenow/sdk/global';
import { ScriptInclude } from `@servicenow/sdk/core`

export const TodoAjax = ScriptInclude({
    $id: Now.ID['TodoAjax'],
    name: 'TodoAjax',
    script: Now.include('../../server/script-includes/todo-ajax.js') // Use Now.include to link the JS file.
    description: 'some description',
    apiName: 'x_scope.TodoAjaxInclude',
    callerAccess: 'tracking',
    clientCallable: true,
    mobileCallable: true,
    sandboxCallable: true,
    accessibleFrom: 'public',
    active: true,
})
```

### 2 - Write the JavaScript Logic

```typescript
// ./src/server/script-includes/todo-ajax.js
var TodoAjax = Class.create();

TodoAjax.prototype = Object.extendsObject(global.AbstractAjaxProcessor, {
  getTasks: function () {
    var tasks = [];
    var gr = new GlideRecord("x_snc_todo_item");

    gr.query();

    while (gr.next()) {
      tasks.push({
        sys_id: gr.getUniqueValue(),
        task: gr.getValue("task"),
        state: gr.getValue("state")
      });
    }

    return JSON.stringify(tasks);
  },

  addTask: function () {
    var task = this.getParameter("sysparm_task");
    var state = this.getParameter("sysparm_state") || "ready";

    var gr = new GlideRecord("x_snc_todo_item");

    gr.initialize();
    gr.setValue("task", task);
    gr.setValue("state", state);

    var sysId = gr.insert();

    return JSON.stringify({
      success: true,
      task: {
        sys_id: sysId,
        task: task,
        state: state
      }
    });
  },

  updateTask: function () {
    var sysId = this.getParameter("sysparm_sys_id");
    var gr = new GlideRecord("x_snc_todo_item");

    gr.get(sysId);

    var task = this.getParameter("sysparm_task");
    var state = this.getParameter("sysparm_state");

    if (task) gr.setValue("task", task);
    if (state) gr.setValue("state", state);

    gr.update();

    return JSON.stringify({
      success: true,
      task: {
        sys_id: sysId,
        task: gr.getValue("task"),
        state: gr.getValue("state")
      }
    });
  },

  deleteTask: function () {
    var sysId = this.getParameter("sysparm_sys_id");
    var gr = new GlideRecord("x_snc_todo_item");

    gr.get(sysId) && gr.deleteRecord();
  },

  type: "TodoAjax"
});
```

## Parameters

| Property          | Type             | Required | Options                     | Default              | Description                                                                                                                                                                                                              |
| ----------------- | ---------------- | -------- | --------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `$id`             | `Now.ID[string]` | Yes      |                             |                      | Unique identifier for the script include                                                                                                                                                                                 |
| `name`            | `string`         | Yes      |                             |                      | The name of the script include. If you are defining a class, this must match the name of the class, prototype, and type. If you are using a classless (on-demand) script include, the name must match the function name. |
| `apiName`         | `string`         | No       |                             | `{scopeName}.{name}` | The internal name of the Script Include                                                                                                                                                                                  |
| `description`     | `string`         | No       |                             |                      | Documentation explaining the purpose and function of the Script Include                                                                                                                                                  |
| `script`          | `string`         | Yes      |                             |                      | Defines the server side script to run when called                                                                                                                                                                        |
| `sandboxCallable` | `boolean`        | No       |                             | `false`              | The script include is available to scripts invoked from the script sandbox, such as a query condition.                                                                                                                   |
| `clientCallable`  | `boolean`        | No       |                             | `false`              | The script include can be called from client-side scripts using GlideAjax                                                                                                                                                |
| `mobileCallable`  | `boolean`        | No       |                             | `false`              | The script include is available to client scripts called from mobile devices                                                                                                                                             |
| `active`          | `boolean`        | No       |                             | `true`               | Enable or disable the script include                                                                                                                                                                                     |
| `accessibleFrom`  | `string`         | No       | `public`, `package_private` | `package_private`    | Sets which applications can access this script include                                                                                                                                                                   |
| `callerAccess`    | `string`         | No       | `restriction`, `tracking`   |                      | Allow scoped applications to restrict access                                                                                                                                                                             |
