# Catalog Variables Reference

For properties and templates, use `get_knowledge_source` with knowledge source `CATALOG_VARIABLES`.

## Variable Types

### Text Variables

- **SingleLineTextVariable** — Single line text input
- **MultiLineTextVariable** — Multi-line text area
- **WideSingleLineTextVariable** — Full-width single line
- **EmailVariable** — Email address input
- **UrlVariable** — URL input
- **IpAddressVariable** — IPv4/IPv6 input
- **MaskedVariable** — Masked/password input (supports `useEncryption`, `useConfirmation`)

### Choice Variables

- **SelectBoxVariable** — Dropdown choice list. Requires `choices` object with `{ label, sequence }`.
- **MultipleChoiceVariable** — Radio buttons. Supports `choiceDirection: 'down'` or `'across'`.
- **YesNoVariable** — Yes/No choice list.
- **CheckboxVariable** — Checkbox. Use `selectionRequired: true` for mandatory.
- **NumericScaleVariable** — Likert scale radio buttons.

### Lookup Variables

- **LookupSelectBoxVariable** — Dropdown from table data.
- **LookupMultipleChoiceVariable** — Radio buttons from table data.

### Reference Variables

- **ReferenceVariable** — References a record in another table. Key properties: `referenceTable`, `referenceQualCondition`, `useReferenceQualifier`.
- **RequestedForVariable** — Specifies who the request is for.
- **ListCollectorVariable** — Select multiple records from a table.

### Date/Time Variables

- **DateVariable** — Date picker.
- **DateTimeVariable** — Date and time picker.
- **DurationVariable** — Duration input.

### Layout Variables

- **ContainerStartVariable** / **ContainerSplitVariable** / **ContainerEndVariable** — Multi-column layout containers. Must be properly paired.
- **LabelVariable** — Display-only label.
- **BreakVariable** — Horizontal line separator.

### Special Variables

- **AttachmentVariable** — File upload.
- **HtmlVariable** — Rich content display.
- **RichTextLabelVariable** — Formatted label.
- **CustomVariable** / **CustomWithLabelVariable** — UI macro insertion.
- **UIPageVariable** — UI page insertion.

## Important Notes

- Variables without names cannot be accessed by client scripts
- Use `readRoles` and `writeRoles` for sensitive fields
- Container variables must be properly paired (Start/End)
- Don't use the same variable name as a table field name (causes conflicts)
- Don't skip the `order` property (causes unpredictable ordering)

For complete examples and templates, use `get_knowledge_source` with knowledge source `CATALOG_VARIABLES`.
