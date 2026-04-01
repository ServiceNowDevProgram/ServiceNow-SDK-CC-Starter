# COLUMN

## Column object

Add Column objects in the schema property of the Table object. Column objects use the format `<Type>Column` where `<Type>` is the field type.

Supported column types: ListColumn, RadioColumn, StringColumn, ChoiceColumn, ScriptColumn, BooleanColumn, ConditionsColumn, DecimalColumn, IntegerColumn, VersionColumn, DomainIdColumn, FieldNameColumn, ReferenceColumn, TableNameColumn, UserRolesColumn, BasicImageColumn, DocumentIdColumn, DomainPathColumn, TranslatedTextColumn, SystemClassNameColumn, TranslatedFieldColumn, GenericColumn, DateColumn, DateTimeColumn, CalendarDateTime, BasicDateTimeColumn, DueDateColumn, CalendarDateTime, IntegerDateColumn, ScheduleDateTimeColumn, and OtherDateColumn.

IMPORTANT: Only the column types explicitly listed above are supported. Do not attempt to use columns not included in this list. Only import the ones you use to avoid build errors.

## Properties

| Name                      | Type            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| label                     | String or Array | A unique label for the column that appears on list headers and form fields. Field labels can be provided as a string or an array of label objects.<br><br>Default: the key used for the column object                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| maxLength                 | Number          | The maximum length of values in the column.<br><br>A length of under 254 appears as a single-line text field. Anything 255 characters or over appears as a multi-line text box.<br><br>**Note:** To avoid data loss, only decrease the length of a string field when you're developing a new application and not when a field contains data.<br><br>Default: 40                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| active                    | Boolean         | Flag that indicates whether to display the field in lists and forms.<br><br>Valid values:<br>• true: Displays the field.<br>• false: Hides the field.<br><br>Default: true                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| mandatory                 | Boolean         | Flag that indicates whether the field must contain a value to save a record.<br><br>Valid values:<br>• true: The field must contain a value.<br>• false: The field isn't required.<br><br>Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| read_only                 | Boolean         | Flag that indicates whether you can edit the field value.<br><br>Valid values:<br>• true: You can't change the value, and the system calculates and displays the data for the field.<br>• false: You can change the field value.<br><br>Default: false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| default                   | Any             | The default value of the field when creating a record. The value must use the correct type based on the column type.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| choices                   | Object          | A list of choices [sys_choice] for a column.<br><br>This property only applies to ChoiceColumn objects and column types that extend choice columns. It can include either an array of primitive values or a series of choice objects.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| attributes                | Object          | Key and value pairs of any supported dictionary attributes [sys_schema_attribute].<br>For example:<br>`attributes: {  update_sync_custom: Boolean, native_recordlock: Boolean}`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| function_definition       | String          | The definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation.<br><br>Each definition begins with `glidefunction:`, followed by the operation to be performed (such as, concat), followed by function parameters. Constants must be enclosed in single quotes.<br><br>For example, the following function definition creates a field that shows the short description, followed by a space, followed by the caller name:<br>`function_definition: 'glidefunction:concat(short_description, ' ', caller_id.name)'`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| dynamic_value_definitions | Object          | Default values that are generated dynamically based on dynamic filters. Provide a combination of a type and a related behavior key to specify dynamic defaults. The following types are supported:<br><br>• dynamic_default: Provide a function from the Dynamic Filter Options [sys_filter_option_dynamic] table. For example:<br>``dynamic_value_definitions: {type: 'dynamic_default', dynamic_default: `gs.info()`},``<br><br>• dependent_field: Provide another column name from the same table. For example:<br>`dynamic_value_definitions: {type: 'dependent_field', column_name: 'status'}`<br><br>• calculated_value: Provide a function for calculating the value. The function can be imported from a JavaScript module or be defined inline. For example:<br>`dynamic_value_definitions: {type: 'calculated_value', calculated_value: function}`<br><br>• choices_from_other_table: Provide choices from a column on another table. For example:<br>`dynamic_value_definitions: {type: 'choices_from_other_table', table: 'sc_cat_item', field: 'display'}` |
| dropdown                  | String          | An option for how a list of choices displays for list and form views of the table. This property only applies to ChoiceColumn objects and column types that extend choice columns.<br><br>Valid values:<br>• none: The choices aren't enforced.<br>• dropdown_without_none: A menu without the -- None -- option. If you select this option, you must configure the default property for the column.<br>• dropdown_with_none: A menu with the -- None -- option. The default value is -- None --.<br>• suggestion: Choices are displayed in a list of suggested values.<br><br>Default: none                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

## Examples

Column names are provided as object keys paired with the column definitions.

```typescript
schema: {
  task: StringColumn({ label: 'Task', maxLength: 120 }),
  deadline: DateColumn({ label: 'Deadline' }), // Default format is 'yyyy-mm-dd' for DateColumn
  state: StringColumn({
    label: 'State',
    choices: {
      ready: { label: 'Ready', sequence: 0 },
      completed: { label: 'Completed', sequence: 1 },
      in_progress: { label: 'In Progress', sequence: 2 },
    },
  }),
  completed_at: DateTimeColumn({ label: 'Completed' }), // Default format is 'yyyy-mm-dd HH:mm:ss' for DateTimeColumn
}
```

If the table name doesn't include the application scope, column names must be prefixed with the application scope instead.

```typescript
schema: {
  x_scope_myColumn: StringColumn({...})
}
```

## Specs for the different types of columns (`sys_dictionary`):

```typescript
StringColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string
  dropdown: 'none', // 'none' | 'dropdown_with_none' | 'suggestion' | 'dropdown_without_none'
  dynamic_value_definitions: {}, // object, see dynamic_value_definition examples
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): StringColumn // returns a StringColumn object

BooleanColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string | boolean
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): BooleanColumn // returns an BooleanColumn object

ChoiceColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string | number
  dropdown: 'none', // 'none' | 'dropdown_with_none' | 'suggestion' | 'dropdown_without_none'
  dynamic_value_definitions: {}, // object, see dynamic_value_definition examples
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): ChoiceColumn // returns a ChoiceColumn object

ReferenceColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  cascadeRule: 'none', // "none" | "cascade" | "delete_no_workflow" | "delete" | "restrict" | "clear"
  default: get_table_name(""), // undefined | string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
  referenceTable?: get_table_name(""), // undefined | string
}): ReferenceColumn // returns a ReferenceColumn object

DateTimeColumn({ // Default format is 'yyyy-mm-dd HH:mm:ss'
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): DateTimeColumn // returns a DateTimeColumn object

DateColumn({ // Default format is 'yyyy-mm-dd'
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): DateColumn // returns a DateColumn object

IntegerColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string
  dropdown: 'none', // 'none' | 'dropdown_with_none' | 'suggestion' | 'dropdown_without_none'
  dynamic_value_definitions: {}, // object, see dynamic_value_definition examples
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  max: 0, // number
  maxLength: 0, // number
  min: 0, // number
  read_only: false // boolean
}): IntegerColumn // returns an IntegerColumn object

DecimalColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string | number
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): DecimalColumn // returns a DecimalColumn object

ListColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: get_table_name(""), // undefined | string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
  referenceTable?: get_table_name(""), // undefined | string
}): ListColumn // returns a ListColumn object

FieldNameColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string
  dropdown: 'none', // 'none' | 'dropdown_with_none' | 'suggestion' | 'dropdown_without_none'
  dynamic_value_definitions: {}, // object, see dynamic_value_definition examples
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): FieldNameColumn // returns a FieldNameColumn object

ScriptColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // undefined | string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
  signature: '', // undefined | string
}): ScriptColumn // returns a ScriptColumn object

UserRolesColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string | Role object see spec
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): UserRolesColumn // returns a UserRolesColumn object

TranslatedTextColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string
  dropdown: 'none', // 'none' | 'dropdown_with_none' | 'suggestion' | 'dropdown_without_none'
  dynamic_value_definitions: {}, // object, see dynamic_value_definition examples
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): TranslatedTextColumn // returns a TranslatedTextColumn object

ConditionsColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: get_table_name(""), // undefined | string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): ConditionsColumn // returns a ConditionsColumn object

TranslatedFieldColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string
  dropdown: 'none', // 'none' | 'dropdown_with_none' | 'suggestion' | 'dropdown_without_none'
  dynamic_value_definitions: {}, // object, see dynamic_value_definition examples
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): TranslatedFieldColumn // returns a TranslatedFieldColumn object

BasicImageColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string
  dropdown: 'none', // 'none' | 'dropdown_with_none' | 'suggestion' | 'dropdown_without_none'
  dynamic_value_definitions: {}, // object, see dynamic_value_definition examples
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): BasicImageColumn // returns a BasicImageColumn object

IntegerDateColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string | number
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): IntegerDateColumn // returns a IntegerDateColumn object

VersionColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): VersionColumn // returns a VersionColumn object

BasicDateTimeColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): BasicDateTimeColumn // returns a BasicDateTimeColumn object

CalendarDateTime({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): CalendarDateTime // returns a CalendarDateTime object

DueDateColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): DueDateColumn // returns a DueDateColumn object

ScheduleDateTimeColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): ScheduleDateTimeColumn // returns a ScheduleDateTimeColumn object

OtherDateColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): OtherDateColumn // returns a OtherDateColumn object

RadioColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): RadioColumn // returns a RadioColumn object

DomainIdColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): DomainIdColumn // returns an DomainIdColumn object

DomainPathColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string
  dropdown: 'none', // 'none' | 'dropdown_with_none' | 'suggestion' | 'dropdown_without_none'
  dynamic_value_definitions: {}, // object, see dynamic_value_definition examples
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): DomainPathColumn // returns a DomainPathColumn object

TableNameColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string | (string & {})
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): TableNameColumn // returns a TableNameColumn object

SystemClassNameColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  choices: {}, // object, snake_case name value pairs, for example { choice_1: { label: 'Choice1' }, choice_2: { label: 'Choice2' } }
  default: '', // string
  dropdown: 'none', // 'none' | 'dropdown_with_none' | 'suggestion' | 'dropdown_without_none'
  dynamic_value_definitions: {}, // object, see dynamic_value_definition examples
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): SystemClassNameColumn // returns a SystemClassNameColumn object

DocumentIdColumn({
  active: false, // boolean
  attributes: {}, // object, snake_case name value pairs, see attribute list
  audit: false, // boolean
  default: '', // string
  dependent: {}, // object TableNameColumn see above spec
  function_definition: `glidefunction:${""}`, // string, definition of a function that the field performs, such as a mathematical operation, field length computation, or day of the week calculation
  label: '', // string or array of Documentation object
  mandatory: false, // boolean
  maxLength: 0, // number
  read_only: false // boolean
}): DocumentIdColumn // returns a DocumentIdColumn object
```
