# Column Type Selection

## Contents

- Type selection guide
- Cross-scope columns
- Import rules

## Type selection guide

Choose the column type based on the data being stored:

| Data                  | Column type                                                  | Notes                                                 |
| --------------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| Short text            | `StringColumn`                                               | `maxLength` < 254 renders as single-line              |
| Long text             | `StringColumn`                                               | `maxLength` >= 255 renders as multi-line              |
| Dropdown / choices    | `ChoiceColumn` or `StringColumn` with `choices` + `dropdown` | Use `StringColumn` when you also need free-text input |
| True/false            | `BooleanColumn`                                              |                                                       |
| Whole numbers         | `IntegerColumn`                                              | Supports `min` and `max` constraints                  |
| Decimal numbers       | `DecimalColumn`                                              |                                                       |
| Foreign key           | `ReferenceColumn`                                            | Set `referenceTable` to the target table name         |
| Multi-value reference | `ListColumn`                                                 | Set `referenceTable` to the target table name         |
| Date only             | `DateColumn`                                                 | Format: `yyyy-mm-dd`                                  |
| Date and time         | `DateTimeColumn`                                             | Format: `yyyy-mm-dd HH:mm:ss`                         |
| Server script         | `ScriptColumn`                                               |                                                       |
| Radio buttons         | `RadioColumn`                                                | Requires `choices`                                    |

For the full list of supported types and their properties, retrieve the COLUMN knowledge source via `get_knowledge_source`.

## Cross-scope columns

When adding columns to a table in a different application scope:

1. Provide the table name without the scope prefix, followed by `as any`
2. Prefix the column names with your application scope instead

```typescript
schema: {
  x_scope_myColumn: StringColumn({...})
}
```

## Import rules

Only import the column types you actually use from `@servicenow/sdk/core`. Unused imports cause build errors.

```typescript
// Good: import only what you use
import { Table, StringColumn, ReferenceColumn } from "@servicenow/sdk/core";

// Bad: importing types you don't use
import {
  Table,
  StringColumn,
  IntegerColumn,
  BooleanColumn,
  ReferenceColumn,
  DateColumn
} from "@servicenow/sdk/core";
```

## Next Steps

- For full column type specs and properties → retrieve COLUMN knowledge source via `get_knowledge_source`.
