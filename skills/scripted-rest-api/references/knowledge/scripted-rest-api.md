# SCRIPTED_REST_API

The Scripted REST API includes objects that define the endpoints, query parameters, and headers for a scripted REST service [sys_ws_definition].

### RestApi object

Create a scripted REST API [sys_ws_definition] to define web service endpoints.

#### Properties

| Name             | Type             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------- | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $id              | String or Number | Required. A unique ID for the metadata object provided in the following format, where <value> is a string or number. `$id: Now.ID[<value>]` When you build the application, this ID is hashed into a unique sys_ID.                                                                                                                                                                                                                                                                                                                                                                    |
| name             | String           | Required. The name of the API, which is used in the API documentation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| serviceId        | String           | Required. The API identifier used to distinguish this API in URI paths. It must be unique within the API namespace.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| active           | Boolean          | Flag that indicates whether the API can serve requests. Valid values: • true: The API can serve requests. • false: The API can't serve requests. Default: true                                                                                                                                                                                                                                                                                                                                                                                                                         |
| shortDescription | String           | A brief description of the API, which is used in the API documentation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| consumes         | String           | A list of media types that resources of the API can consume. Default: application/json,application/xml,text/xml                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| docLink          | String           | A URL that links to static documentation about the API.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| enforceAcl       | Array            | A list of variable identifiers of ACL objects or sys_ids of ACLs to enforce when accessing resources [sys_security_acl]. For more information, see Access Control List API - ServiceNow Fluent. To not enforce ACLs, set this property to an empty array ([]). enforceAcl: ['sys_id', acl_variable_identifier] Default: Scripted REST External Default                                                                                                                                                                                                                                 |
| produces         | String           | A list of media types that resources of the API can produce. Default: application/json,application/xml,text/xml                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| routes           | Array            | The resources [sys_ws_operation] for the API. For more information, see routes object.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| policy           | String           | The policy for how application files are protected when downloaded or installed. Valid values: • read: Files are viewable only. • protected: Users with password permissions can edit the files.                                                                                                                                                                                                                                                                                                                                                                                       |
| versions         | Array            | A list of versions [sys_ws_version] for the API. For more information, see versions object. Specifying versions allows you to manage different versions of an API and their statuses, such as whether they are active, the default version, or deprecated.                                                                                                                                                                                                                                                                                                                             |
| $meta            | Object           | Metadata for the application metadata. With the installMethod property, you can map the application metadata to an output directory that loads only in specific circumstances. `$meta: { installMethod: 'String' }` Valid values for installMethod: • demo: Outputs the application metadata to the metadata/unload.demo directory to be installed with the application when the Load demo data option is selected. • first install: Outputs the application metadata to the metadata/unload directory to be installed only the first time an application is installed on an instance. |

#### Example

```javascript
import { RestApi } from "@servicenow/sdk/core";
import { process } from "../server/handler.js";
RestApi({
  $id: Now.ID["rest1"],
  name: "customAPI",
  serviceId: "custom_api",
  consumes: "application/json",
  routes: [
    {
      $id: Now.ID["route1"],
      path: "/home/{id}",
      script: process,
      parameters: [{ $id: Now.ID["param1"], name: "n_param" }],
      headers: [{ $id: Now.ID["header1"], name: "n_token" }],
      enforceAcl: [acl],
      version: 1
    }
  ],
  enforceAcl: [acl],
  versions: [
    {
      $id: Now.ID["v1"],
      version: 1,
      isDefault: true
    }
  ]
});
```

The ACL referenced is defined using the ACL object:

```javascript
import { Acl } from "@servicenow/sdk/core";
const acl = Acl({
  name: "My random ACL",
  type: "rest_endpoint",
  script: `answer = (Math.random() > 0.5)`,
  active: true,
  adminOverrides: false,
  operations: ["execute"]
});
```

### routes object

Create a scripted REST resource [sys_ws_operation] to define the HTTP method, the processing script, and to override settings from the parent service. Use the routes object within the RestApi object.

#### Properties

| Name             | Type             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ---------------- | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $id              | String or Number | Required. A unique ID for the metadata object provided in the following format, where <value> is a string or number. `$id: Now.ID[<value>]` When you build the application, this ID is hashed into a unique sys_ID.                                                                                                                                                                                                                                                                                                                                                                    |
| name             | String           | The name of the API resource, which is used in the API documentation. Default: the value of the path property                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| script           | Script           | Required. A function or inline script. The custom script defines how the operation parses and responds to requests. For functions, use the name of a function, function expression, or default function exported from a JavaScript module and imported into the .now.ts file. For information about JavaScript modules, see JavaScript modules and third-party libraries. Inline scripts can use string literals or template literals for scripts with multiple lines. script: `Script`,                                                                                               |
| parameters       | Array            | A list of query parameters [sys_ws_query_parameter] for the route. For more information, see parameters and headers objects.                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| headers          | Array            | A list of headers [sys_ws_header] for the route. For more information, see parameters and headers objects.                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| active           | Boolean          | Flag that indicates whether the resource is used. Valid values: • true: The resource is used. • false: The resource isn't used. Default: true                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| path             | String           | The path of the resource relative to the base API path. The relative URI can contain path parameters such as '/abc/{id}'. Default: /                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| shortDescription | String           | A brief description of the resource, which is used in the API documentation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| consumes         | String           | A list of media types that the resource can consume. This property can be overridden with the PUT, PATCH, or POST methods. Default: The value of the consumes property in the RestApi object                                                                                                                                                                                                                                                                                                                                                                                           |
| enforceAcl       | Array            | A list of variable identifiers of ACL objects or sys_ids of ACLs to enforce when accessing resources [sys_security_acl]. For more information, see Access Control List API - ServiceNow Fluent. To not enforce ACLs, set this property to an empty array ([]). enforceAcl: ['sys_id', acl_variable_identifier] Default: Scripted REST External Default                                                                                                                                                                                                                                 |
| produces         | String           | A list of media types that the resource can produce. Default: The value of the produces property in the RestApi object                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| requestExample   | String           | A valid sample request body payload for the resource, which is used in the API documentation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| method           | String           | The HTTP method that the resource implements. Valid values: GET, POST, PUT, PATCH, DELETE Default: GET                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| authorization    | Boolean          | Flag that indicates whether users must be authenticated to access the resource. Valid values: • true: Users must be authenticated to access the resource. • false: Authentication isn't required to access the resource. Default: true                                                                                                                                                                                                                                                                                                                                                 |
| authentication   | Boolean          | Flag that indicates whether ACLs are enforced when accessing the resource. Valid values: • true: ACLs are enforced when accessing the resource. • false: ACLs aren't enforced when accessing the resource. Default: true                                                                                                                                                                                                                                                                                                                                                               |
| internalRole     | Boolean          | Flag that indicates whether the route requires the snc_internal role. This property is supported only if the Explicit Roles plugin (com.glide.explicit_roles) is enabled. Valid values: • true: The route requires the snc_internal role. • false: The route doesn't require the snc_internal role. Default: true                                                                                                                                                                                                                                                                      |
| policy           | String           | The policy for how application files are protected when downloaded or installed. Valid values: • read: Files are viewable only. • protected: Users with password permissions can edit the files.                                                                                                                                                                                                                                                                                                                                                                                       |
| version          | Number           | The version of the API. This property is required if the versions property is used in the RestApi object. The version specified with this property is used to automatically generate a URI with a version, such as /api/management/v1/table/{tableName}. Version numbers identify the endpoint version that a URI accesses. By specifying a version number, you can test and deploy changes without impacting existing integrations.                                                                                                                                                   |
| $meta            | Object           | Metadata for the application metadata. With the installMethod property, you can map the application metadata to an output directory that loads only in specific circumstances. `$meta: { installMethod: 'String' }` Valid values for installMethod: • demo: Outputs the application metadata to the metadata/unload.demo directory to be installed with the application when the Load demo data option is selected. • first install: Outputs the application metadata to the metadata/unload directory to be installed only the first time an application is installed on an instance. |

#### Example

```javascript
routes: [
   {
      $id: Now.ID['route1'],
      path: '/home/{id}',
      script: process,
      parameters: [{ $id: Now.ID['param1'],  name: 'n_param' }],
      headers: [{ $id: Now.ID['header1'],  name: 'n_token' }],
      enforceAcl: [acl],
      version: 1,
   },
],
```

### parameters and headers objects

Create query parameters [sys_ws_query_parameter] and headers [sys_ws_header] for routes in a scripted REST API. Query parameters control what values a requesting user can pass in the request URI. Headers specify what the API accepts and can respond with. Use the parameters and headers objects within the routes object.

### versions object

Create versions for a scripted REST API [sys_ws_version] to define web service endpoints. Use the `versions` object within the `RestApi` object.

#### Properties

| Name             | Type             | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| $id              | String or Number | Required. A unique ID for the metadata object. When you build the application, this ID is hashed into a unique `sys_id`. <br>Format: `Now.ID['String' or Number]`                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| version          | Number           | Required. A version of the REST API.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| active           | Boolean          | Flag that indicates whether the version of the REST API can serve requests. <br>Valid values: <br>• `true`: The version of the API can serve requests. <br>• `false`: The version of the API can't serve requests. <br>Default: `true`                                                                                                                                                                                                                                                                                                                                                                             |
| deprecated       | Boolean          | Flag that indicates whether the version of the REST API is deprecated. Resources belonging to deprecated versions can still serve requests, but are marked as deprecated in the documentation. <br>Valid values: <br>• `true`: The version of the API is identified as deprecated. <br>• `false`: The version of the API isn't identified as deprecated. <br>Default: `false`                                                                                                                                                                                                                                      |
| shortDescription | String           | A brief description of the version of the REST API, which appears in the API documentation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| isDefault        | Boolean          | Flag that indicates whether the version of the REST API is the default version. Clients can access the default version using either the versioned or non-versioned URI path. <br>Valid values: <br>• `true`: The version of the API is the default version. <br>• `false`: The version of the API isn't the default version. <br>Default: `false`                                                                                                                                                                                                                                                                  |
| $meta            | Object           | Metadata for the application metadata. With the `installMethod` property, you can map the application metadata to an output directory that loads only in specific circumstances. <br>`$meta: { installMethod: 'String' }` <br>Valid values for installMethod: <br>• `demo`: Outputs the application metadata to the `metadata/unload.demo` directory to be installed with the application when the _Load demo data_ option is selected. <br>• `first install`: Outputs the application metadata to the `metadata/unload` directory to be installed only the first time an application is installed on an instance. |

#### Example

```javascript
versions: [
   {
      $id: Now.ID['v1'],
      version: 1,
      active: true,
      isDefault: true,
      shortDescription: 'Initial version of the custom API',
   },
],
```

### URI Path construction

- If version is specified in the rest definition, then use the following format: `/api/{scope_name}/v{version}/{serviceId}/{path}?<query_params>`
- If the version is not specified in the rest definition, then use following format: `/api/{scope_name}/{serviceId}/{path}?<query_params>`

#### Example:

```javascript
//Version is specified in the rest definition
import { RestApi } from "@servicenow/sdk/core";
import { process } from "../server/handler.js";
RestApi({
  $id: Now.ID["rest1"],
  name: "customAPI",
  serviceId: "custom_api",
  consumes: "application/json",
  routes: [
    {
      $id: Now.ID["route1"],
      path: "/home/{id}",
      script: process,
      parameters: [{ $id: Now.ID["param1"], name: "n_param" }],
      headers: [{ $id: Now.ID["header1"], name: "n_token" }],
      enforceAcl: [acl],
      version: 1
    }
  ],
  enforceAcl: [acl],
  versions: [
    {
      $id: Now.ID["v1"],
      version: 1,
      isDefault: true
    }
  ]
});
```

//For above example, the URI path would be:
`/api/custom_api/v1/custom_api/home/{id}?n_param=123`

```javascript
//Version is not specified in the rest definition
import { RestApi } from "@servicenow/sdk/core";
import { process } from "../server/handler.js";
RestApi({
  $id: Now.ID["rest1"],
  name: "customAPI",
  serviceId: "custom_api",
  consumes: "application/json",
  routes: [
    {
      $id: Now.ID["route1"],
      path: "/home/{id}",
      script: process,
      parameters: [{ $id: Now.ID["param1"], name: "n_param" }],
      headers: [{ $id: Now.ID["header1"], name: "n_token" }],
      enforceAcl: [acl]
    }
  ],
  enforceAcl: [acl]
});
```

//For above example, the URI path would be:
`/api/custom_api/custom_api/home/{id}?n_param=123`
