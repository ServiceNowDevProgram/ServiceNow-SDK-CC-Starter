# TABLE

## Table object

## Properties

| Name                     | Type            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------ | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name                     | String          | Required. A name for the table beginning with the application scope and in all lowercase letters in the following format: `<scopeName>_<name>`. The scopeName is available in CURRENT DIRECTORY CONTEXT as `scopeName`. For example, if the scopeName is "x_acme", the table name should be "x_acme_my_table". The variable identifier of the Table object MUST match this name exactly for proper functionality. **Note:** To add columns to an existing table in a different application scope, you can provide the name of the table without the application scope followed by `as any`. The column names must begin with the application scope instead. Maximum length: 80                                                                                                                                                                                    |
| schema                   | Array           | Required. A list of Column objects.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| extends                  | String          | The name of any other table on which the table is based. Extending a base table incorporates all the fields of the original table and creates system fields for the new table. If they are in the same scope or if they can be configured from other scopes, you can extend tables that are marked as extensible.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| label                    | String or Array | A unique label for the table in list and form views. Field labels can be provided as a string or an array of label objects. Maximum length: 80 Default: the value of the name property                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| licensing_config         | Object          | The licensing configuration [ua_table_licensing_config] for a table.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| display                  | String          | The default display column. Use a column name from the schema property.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| extensible               | Boolean         | Flag that indicates whether other tables can extend the table. Valid values: • true: Other tables can extend the table. • false: Other tables can't extend the table. Changing this property from true to false prevents the creation of additional child tables but existing child tables remain unchanged. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| live_feed                | Boolean         | Flag that indicates if live feeds are available for records in the table. Valid values: • true: Live feeds are provided for records in the table. This option adds the Show Live Feed option in the form header. • false: Live feeds aren't provided for records in the table. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| accessible_from          | String          | The application scopes that can access the table. Valid values: • public: The table is accessible from all application scopes. • package_private: The table is accessible from only the application scope it's in. Default: public                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| caller_access            | String          | The access level for cross-scope requests. Valid values: • restricted: Calls to the resource must be manually approved. Access requests are tracked in the Restricted Caller Access table with a status of Requested. • tracking: Calls to the resource are automatically approved. Calls are tracked in the Restricted Caller Access table with a status of Allowed. • none: Cross-scope calls to the resource are approved or denied based on the value of the accessible_from property. Default: none                                                                                                                                                                                                                                                                                                                                                          |
| actions                  | Array           | A list of access options. Valid values: • read: Allow script objects from other application scopes to read records stored in this table. For example, a script in another application can query data on this table. Read access is required to grant any other API record operations. • create: Allow script objects from other application scopes to create records in this table. For example, a script in another application can insert a new record in this table. • update: Allow script objects from other application scopes to modify records stored in this table. For example, a script in another application can modify a field value on this table. • delete: Allow script objects from other application scopes to delete records from this table. For example, a script in another application can remove a record from this table. Default: read |
| allow_web_service_access | Boolean         | Flag that indicates whether web services can make calls to the table. Valid values: • true: Web services can make calls to the table. • false: Web services can't make calls to the table. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| allow_new_fields         | Boolean         | Flag that indicates whether to allow design time configuration of new fields on the table from other application scopes. Valid values: • true: Allow design time configuration of new fields on the table from other application scopes. • false: Don't allow design time configuration of new fields on the table from other application scopes. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| allow_ui_actions         | Boolean         | Flag that indicates whether to allow design time configuration of UI actions on the table from other application scopes. Valid values: • true: Allow design time configuration of UI actions on the table from other application scopes. • false: Don't allow design time configuration of UI actions on the table from other application scopes. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| allow_client_scripts     | Boolean         | Flag that indicates whether to allow design time configuration of client scripts on the table from other application scopes. Valid values: • true: Allow design time configuration of client scripts on the table from other application scopes. • false: Don't allow design time configuration of client scripts on the table from other application scopes. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| audit                    | Boolean         | Flag that indicates whether to track the creation, update, and deletion of all records in the table. Valid values: • true: Track the creation, update, and deletion of all records in the table • false: Don't track the creation, update, and deletion of all records in the table. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| read_only                | Boolean         | Flag that indicates whether users can edit fields in the table. Valid values: • true: Users can't edit fields in the table. • false: Users can edit fields in the table. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| text_index               | Boolean         | Flag that indicates whether search engines index the text in a table. Valid values: • true: The table's text is indexed. • false: The table's text isn't indexed. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| attributes               | Object          | Key and value pairs of any supported dictionary attributes [sys_schema_attribute]. For example: `attributes: { update_sync_custom: Boolean, native_recordlock: Boolean}`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| index                    | Array           | A list of column references to generate indexes in the metadata XML of the table. The value of the element property should match the object key used with the Column object. A database index increases the speed of accessing data from the table with the expense of using additional storage. `index: [{name: 'String', element: 'String', unique: Boolean}]`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| auto_number              | Object          | The auto-numbering configuration [sys_number] for a table.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| scriptable_table         | Boolean         | Flag that indicates whether the table is a remote table that uses data retrieved from an external source. Valid values: • true: The table is a remote table. • false: The table isn't a remote table. Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |

## Example

For typeahead support for columns, assign the Table object to an exported variable with the same name as the name property.

```typescript
import { Table, StringColumn } from "@servicenow/sdk/core";
import { myFunction } from "../server/myFunction.js";

// IMPORTANT: The exported constant name MUST match the name property value
// If your scopeName from CURRENT DIRECTORY CONTEXT is "x_snc_example", name your to_do table "x_snc_example_to_do"
export const x_snc_example_to_do = Table({
  name: "x_snc_example_to_do",
  label: "My To Do Table",
  extends: "task",
  schema: {
    status: StringColumn({ label: "Status" }),
    deadline: StringColumn({
      label: "Deadline",
      active: true,
      mandatory: false,
      read_only: false,
      maxLength: 40,
      dropdown: "none",
      attributes: {
        update_sync: false
      },
      default: "today",
      dynamic_value_definitions: {
        type: "calculated_value",
        calculated_value: ""
      },
      choices: {
        choice1: {
          label: "Choice1 Label",
          sequence: 0,
          inactive_on_update: false,
          dependent_value: "5",
          hint: "hint",
          inactive: false,
          language: "en"
        },
        choice2: { label: "Choice2 Label", sequence: 1 }
      }
    }),
    dynamic1: StringColumn({
      dynamic_value_definitions: {
        type: "calculated_value",
        calculated_value: myFunction
      }
    }),
    dynamic2: StringColumn({
      dynamic_value_definitions: {
        type: "dynamic_default",
        dynamic_default: `gs.info()`
      }
    }),
    dynamic3: StringColumn({
      dynamic_value_definitions: {
        type: "dependent_field",
        column_name: "status"
      }
    }),
    dynamic4: StringColumn({
      dynamic_value_definitions: {
        type: "choices_from_other_table",
        table: "sc_cat_item",
        field: "display"
      }
    })
  },
  actions: ["create", "read", "update", "delete"],
  display: "deadline",
  accessible_from: "package_private",
  allow_client_scripts: true,
  allow_new_fields: true,
  allow_ui_actions: true,
  allow_web_service_access: true,
  extensible: true,
  live_feed: true,
  caller_access: "none",
  auto_number: {
    number: 10,
    number_of_digits: 2,
    prefix: "abc"
  },
  audit: true,
  read_only: true,
  text_index: true,
  attributes: {
    update_sync: true
  },
  index: [
    {
      name: "idx",
      element: "status",
      unique: true
    }
  ]
});
```

## choices object

Configure choices [sys_choice] for a column in a table.

The choices object is a property within the Column object. Use the choices object with supported column types in the schema property of a Table object. Only certain column types extend the choice column type (`ChoiceColumn`) and can include choices.

## Properties

| Name            | Type   | Description                                                                                                 |
| --------------- | ------ | ----------------------------------------------------------------------------------------------------------- |
| label           | String | Required. The text to display for the choice in the list.                                                   |
| dependent_value | String | A value that you map to the dependent_field in the dynamic_value_definitions property of the Column object. |
| hint            | String | A short description of the choice that displays as tooltip when hovering over it.                           |
| language        | String | The BCP 47 code of the language for the translated choice.                                                  |

Default: en |
| sequence | Integer | The order in the list of choices that a choice occurs. |
| inactive | Boolean | Flag that indicates whether to show the choice in the list.

Valid values:  
• true: The choice is hidden from the list.  
• false: The choice appears in the list.

Default: false |

## Example

The choices object includes a series of choice objects, where the names of the choices are provided as object keys paired with the choices definitions.

```typescript
choices: {
  choice1: {
    label: 'Choice1 Label',
    sequence: 0,
    inactive_on_update: false,
    dependent_value: '5',
    hint: 'hint',
    inactive: false,
    language: 'en',
  },
  choice2: { label: 'Choice2 Label', sequence: 1 },
}
```

## label object

Configure a field label [sys_documentation] for a table or column.

The label object is a property within the `Table` and `Column` objects.

## Properties

| Name       | Type   | Description                                                                                                                                                |
| ---------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| language   | String | The BCP 47 code of the language for the field label. A language can have only one label, so each language must be unique within an array of label objects. |
| label      | String | The text of the field label in the specified language.                                                                                                     |
| hint       | String | A short description that displays as a tooltip when hovering over the field label.                                                                         |
| help       | String | Additional information about the field. Help text isn't displayed in form or list views of the table.                                                      |
| plural     | String | The plural form of the field label.                                                                                                                        |
| url        | String | A URL for a web page that provides information about the field. When a URL is provided, the label displays as a hyperlink.                                 |
| url_target | String | Not used (deprecated).                                                                                                                                     |

## Example

```typescript
label: [
  {
    label: "English description",
    language: "en",
    hint: "Provide a short description"
  },
  {
    label: "Description de español",
    language: "es"
  }
];
```

## licensing_config object

Create a licensing configuration [ua_table_licensing_config] to track subscription counts for a table.

The licensing_config object is a property within the Table object.

If this property isn't specified, a default licensing configuration with license_model set to none is generated for the table on the instance.

**Note:** Specifying a licensing model is not applicable for ServiceNow customers who build custom applications for their own use. Licensing models are used only by partners who sell and monitor the usage of resellable applications on the ServiceNow Store.

## Properties

| Name          | Type   | Description                                              |
| ------------- | ------ | -------------------------------------------------------- |
| license_model | String | The model for tracking subscription usage. Valid values: |

• none: Licensing isn't used for the table.  
• fulfiller: Fulfiller/requester operations are tracked. This model applies to applications in which users open requests and fulfillers address them. Fulfillment is determined by insert, update, and delete operations on records in one or more key tables in the application under a set of specified conditions.  
• producer: Producer operations are tracked. This model applies to applications in which users can perform insert, update, and delete operations on a table without identifying requesters and fulfillers.

Default: none |
| license_roles | Array | A list of roles for which any operations on records in the table count toward the subscription. |
| op_delete | Boolean | Flag that indicates whether a subscription is required to delete records for tables with the producer model.

Valid values:  
• true: A subscription is required to delete records in the table.  
• false: A subscription isn't required to delete records in the table.

Default: true |
| op_insert | Boolean | Flag that indicates whether a subscription is required to insert records for tables with the producer model.

Valid values:  
• true: A subscription is required to insert records in the table.  
• false: A subscription isn't required to insert records in the table.

Default: true |
| op_update | Boolean | Flag that indicates whether a subscription is required to update records for tables with the producer model.

Valid values:  
• true: A subscription is required to update records in the table.  
• false: A subscription isn't required to update records in the table.

Default: true |
| license_condition | String | A filter query that determines conditions for counting operations toward a subscription.

For the fulfiller model, specify the set of conditions that determine whether the logged-in user is the fulfiller of the record.

For the producer model, specify the set of conditions that determine whether records count toward the subscription. |
| owner_condition | String | A filter query that determines whether a user owns a record for the fulfiller model. |
| is_fulfillment | Boolean | Not used (deprecated). Flag that indicates whether to disallow updates by users who aren't subscribed to the application.

Valid values:  
• true: Users who aren't subscribed to the application can't make updates to the table.  
• false: Users who aren't subscribed to the application can make updates to the table.

Default: false |

## Example

```typescript
licensing_config: {
  license_model: 'fulfiller',
  op_insert: false,
  license_roles: ['admin'],
}
```

## auto_number object

Configure auto-numbering [sys_number] for a table.

The auto_number object is a property within the `Table` object.

## Properties

| Name   | Type   | Description                                                                   |
| ------ | ------ | ----------------------------------------------------------------------------- |
| prefix | String | A prefix for every record number in the table. For example, INC for Incident. |

Default: pre |
| number | Integer | The base record number for this table. Record numbers are automatically incremented, and the next number is maintained in the Counter [sys_number_counter] table.

If you set the base number to a value higher than the current counter, the next record number uses the new base number. Otherwise the next record number uses the current counter. The counter doesn't reset to a base number lower than itself.

Default: 1000 |
| number_of_digits | Integer | The minimum number of digits to use after the prefix.

Leading zeros are added to auto-numbers, if necessary. For example, INC0001001 contains three leading zeros. The number of digits can exceed the minimum length. For example, if number_of_digits is 2 and more than 99 records are created on the table, the numbers continue past 100 (such as INC101).

**Warning:** Changing this field can update all number values for existing records on a table. Take care when changing this field on a production instance.

Default: 7 |

## Example

```typescript
auto_number: {
  prefix: 'TODO',
  number: 2000,
  digits: 9,
}
```

To use the number in a table, you need to create a number column that uses the number as the default value. For example:

```typescript
number: IntegerColumn({
  default: "javascript:global.getNextObjNumberPadded();"
});
```
