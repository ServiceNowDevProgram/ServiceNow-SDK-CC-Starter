# PROPERTY

The Property API includes objects that define system properties [sys_properties].

## Property object

Add system properties [sys_properties] for configuring aspects of an application.

Properties

| Field Name    | Type             | Mandatory | Options                                                                                                                                                             | Default | Comments                                                                                                    |
| ------------- | ---------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ----------------------------------------------------------------------------------------------------------- |
| `$id`         | string or number | true      |                                                                                                                                                                     |         | Use `Now.ID` notation to provide a unique id for the metadata                                               |
| `name`        | string           | true      |                                                                                                                                                                     |         | must start with app scope. For example if scope is `x_snc_example`, name should start with `x_snc_example.` |
| `type`        | string           | false     | `string`, `integer`, `boolean`, `choicelist`, `color`, `date_format`, `image`, `password`, `password2`, `short_string`, `time_format`, `timezone`, `uploaded_image` |
| `value`       | any              | false     |                                                                                                                                                                     |         | type of `value` depends on the `type` selected above                                                        |
| `description` | string           | false     |
| `ignoreCache` | boolean          | false     |                                                                                                                                                                     | `false` |
| `isPrivate`   | boolean          | false     |                                                                                                                                                                     | `false` |
| `choices`     | array of string  | false     |
| `roles`       | Object           | false     |                                                                                                                                                                     |         | values of the object are of type array of string or `sys_user_role` records                                 |

## Example

```typescript
import { Property } from '@servicenow/sdk/core'

Property({
    $id: Now.ID['<unique string or number>']
    name: 'some.new.prop',
    type: 'string',
    value: 'hello',
    description: 'Brand new prop',
    roles: {
        read: ['admin'],
        write: [activity_admin],
    },
    ignoreCache: false,
    isPrivate: false,
})
```

```typescript
import { Property, Role } from "@servicenow/sdk/core";

Property({
  $id: Now.ID["1234"],
  name: "x_snc_app.some.new.prop",
  type: "string",
  value: "hello",
  description: "A new property",
  roles: {
    read: ["admin"],
    write: [adminRole, managerRole]
  },
  ignoreCache: false,
  isPrivate: false
});

// The roles referenced are defined using the Role object:
const managerRole = Role({
  $id: Now.ID["manager_role"],
  name: "x_snc_example.manager"
});

const adminRole = Role({
  $id: Now.ID["admin_role"],
  name: "x_snc_example.admin",
  containsRoles: [managerRole]
});
```
