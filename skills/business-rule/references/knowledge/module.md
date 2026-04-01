# MODULE

## Exporting modules

In a module, identify code for reuse with `export` statements. You can use named exports or default exports. Named exports can be for variables, constants, functions, or classes whereas default exports can be for functions or classes only. The following example is one way of adding a named export for multiple features (a function and a variable) in a module:

```javascript
export { myFunction, myVariable };
```

## Importing modules

To import the module code you want to reuse, use `import` statements in other modules or `require` statements in server-side scripts.

The following example is one way that you could import an exported feature in a module:

```javascript
import { feature } from "path/to/module";
```

The following example is one way that you could import an exported feature in a script:

```javascript
const { feature } = require("path/to/module");
```

To use shorthand to import module code, you can use subpaths in the `imports` field of the application's package.json file. For example:

```json
{
  "name": "math",
  "version": "1.0.0",
  "exports": {
    "./functions/*.js": "./src/functions/*.js",
    "./functions/private-functions/*": null
  },
  "imports": {
    "#calc": "calculus",
    "#derivative": "calculus/derivative"
  },
  "dependencies": {
    "calculus": "1.0.0"
  }
}
```

Based on that example, instead of writing out the relative path to `derivative.js` every time you want to import it in the math application, you can use the `#derivative` shorthand instead. Subpaths can also be used in the imports field to use shorthand for dependencies, such as `#calc`.

```javascript
import { derivative } from "#derivative";
import * as calculus from "#calc";
```

## Adding dependencies

Applications must declare dependencies on third-party libraries to use their module code. In an application's package.json file, include the package name and version for any dependencies. For example, to use modules from the "math" library in the "test" application, add the "math" package as a dependency:

```json
{
  "name": "test",
  "version": "1.0.0",
  "dependencies": {
    "math": "1.0.0"
  }
}
```

**IMPORTANT:** When adding new dependencies to package.json, NEVER modify the versions of existing dependencies unless explicitly requested. Only add the new dependency with its required version while preserving all existing dependency versions exactly as they are.

## Importing server APIs

To import server APIs and use them in a module, use `import` statements. Glide APIs can be imported from the `@servicenow/glide` package or their namespace in the package.

For example:

```javascript
import { API } from "@servicenow/glide";
import { API } from "@servicenow/glide/<namespace>";
```

In the following example, the gs (GlideSystem) and GlideRecord APIs are imported in a module:

```javascript
import { gs } from "@servicenow/glide";
import { GlideRecord } from "@servicenow/glide";
```

In the following example, the `RESTAPIRequest` and `RESTAPIResponse` APIs are imported from the sn_ws_int namespace in a module because they run in that namespace:

```javascript
import { RESTAPIRequest, RESTAPIResponse } from "@servicenow/glide/sn_ws_int";
```

## Script Include classes in modules

Import Script Include classes from `@servicenow/glide/<scopeName>`:

```javascript
import { RecordUtils } from "@servicenow/glide/x_my_scope";

export function onRecordInsert(current, previous) {
  var recordUtils = new RecordUtils();
}
```

Script Include classes in module files do NOT need Glide API imports — `gs`, `GlideRecord`, `GlideDateTime`, etc. are automatically available in the Script Include execution context. Only import other Script Include classes using `@servicenow/glide/<scopeName>`.
