# CATALOG_VARIABLES

# Catalog Variables - Templates and Examples

This knowledge source provides working code templates for ServiceNow Catalog Variables and Variable Sets. For guidelines, requirements, and patterns, activate the `service-catalog` skill first.

## Text Variables

```typescript
// Single line text
SingleLineTextVariable({
  question: "Employee Name",
  order: 100,
  mandatory: true,
  exampleText: "John Smith"
});

// Multi-line text
MultiLineTextVariable({
  question: "Description",
  order: 200,
  mandatory: true,
  width: 100
});

// Email
EmailVariable({
  question: "Email Address",
  order: 300,
  mandatory: true
});

// Masked (password)
MaskedVariable({
  question: "Enter Password",
  order: 400,
  useEncryption: true
});
```

## Choice Variables

```typescript
// Select box (dropdown)
SelectBoxVariable({
  question: "Priority Level",
  order: 100,
  choices: {
    high: { label: "High", sequence: 1 },
    medium: { label: "Medium", sequence: 2 },
    low: { label: "Low", sequence: 3 }
  },
  includeNone: true
});

// Multiple choice (radio buttons)
MultipleChoiceVariable({
  question: "Services Required",
  choiceDirection: "down",
  choices: {
    install: { label: "Installation", sequence: 1 },
    config: { label: "Configuration", sequence: 2 },
    training: { label: "Training", sequence: 3 }
  },
  order: 200
});

// Yes/No
YesNoVariable({
  question: "Manager Approval Required?",
  order: 300,
  mandatory: true,
  includeNone: false
});

// Checkbox
CheckboxVariable({
  question: "I agree to the terms",
  order: 400,
  selectionRequired: true
});
```

## Date/Time Variables

```typescript
DateVariable({ question: "Start Date", order: 100, mandatory: true });
DateTimeVariable({ question: "Meeting Time", order: 200, mandatory: true });
DurationVariable({ question: "Service Duration", order: 300, mandatory: true });
```

## Reference Variables

```typescript
// Reference to sys_user
ReferenceVariable({
  question: "Point of Contact",
  referenceTable: "sys_user",
  referenceQualCondition: "active=true",
  order: 100
});

// Requested for
RequestedForVariable({
  question: "Request For",
  referenceQualCondition: "active=true",
  order: 200
});

// List collector (multi-select)
ListCollectorVariable({
  question: "Team Members",
  listTable: "sys_user",
  referenceQual: "active=true",
  order: 300,
  mandatory: true
});
```

## Container Layout (Multi-Column)

```typescript
variables: {
  contact_container_start: ContainerStartVariable({
    question: "Contact Information",
    layout: "2across",
    displayTitle: true,
    order: 100
  }),

  first_name: SingleLineTextVariable({
    question: "First Name",
    mandatory: true,
    order: 110
  }),

  contact_split: ContainerSplitVariable({
    order: 200
  }),

  email: EmailVariable({
    question: "Email Address",
    mandatory: true,
    order: 210
  }),

  contact_container_end: ContainerEndVariable({
    order: 300
  })
}
```

## Variables with Pricing

```typescript
// Checkbox with conditional pricing
premiumSupport: CheckboxVariable({
  question: "Premium Support (+$150)",
  pricingDetails: [
    { amount: 150, currencyType: "USD", field: "price_if_checked" },
    { amount: 30, currencyType: "USD", field: "rec_price_if_checked" }
  ],
  order: 500
});

// SelectBox with choice pricing
hardwareType: SelectBoxVariable({
  question: "Hardware Type",
  choices: {
    laptop: {
      label: "Business Laptop (Base)",
      sequence: 1,
      pricingDetails: [{ amount: 0, currencyType: "USD", field: "misc" }]
    },
    workstation: {
      label: "Developer Workstation (+$800)",
      sequence: 2,
      pricingDetails: [{ amount: 800, currencyType: "USD", field: "misc" }]
    }
  },
  mandatory: true,
  order: 600
});
```

## Field Mapping (Record Producers)

```typescript
short_description: SingleLineTextVariable({
  question: "Summary",
  mapToField: true,
  field: "short_description",
  mandatory: true,
  order: 100
});
```

---

## Single-Row Variable Set

```typescript
import {
  VariableSet,
  EmailVariable,
  SingleLineTextVariable,
  ReferenceVariable
} from "@servicenow/sdk/core";

export const contactInfoSet = VariableSet({
  $id: Now.ID["contact_info_set"],
  title: "Contact Information",
  description: "Standard contact information fields",
  type: "singleRow",
  layout: "2across",
  order: 100,
  displayTitle: true,
  variables: {
    email: EmailVariable({
      question: "Email Address",
      mandatory: true,
      order: 100
    }),
    phone: SingleLineTextVariable({
      question: "Phone Number",
      mandatory: true,
      order: 200
    }),
    department: ReferenceVariable({
      question: "Department",
      referenceTable: "cmn_department",
      referenceQualCondition: "active=true",
      order: 300
    })
  }
});
```

## Multi-Row Variable Set (MRVS)

```typescript
export const teamMembersSet = VariableSet({
  $id: Now.ID["team_members_set"],
  title: "Team Members",
  description: "Add multiple team members who need access",
  type: "multiRow",
  layout: "2across",
  displayTitle: true,
  setAttributes: "max_rows=10,collapsible=true",

  readRoles: ["admin", "manager"],
  writeRoles: ["admin"],

  variables: {
    user: ReferenceVariable({
      question: "User",
      referenceTable: "sys_user",
      referenceQualCondition: "active=true",
      mandatory: true,
      order: 100
    }),
    accessLevel: SelectBoxVariable({
      question: "Access Level",
      choices: {
        read: { label: "Read Only", sequence: 1 },
        write: { label: "Write", sequence: 2 },
        admin: { label: "Admin", sequence: 3 }
      },
      mandatory: true,
      order: 200
    }),
    startDate: DateVariable({
      question: "Access Start Date",
      mandatory: true,
      order: 300
    })
  }
});
```

## Attaching Variable Sets to Catalog Items

```typescript
export const accessRequest = CatalogItem({
  $id: Now.ID["access_request"],
  name: "Team Access Request",
  shortDescription: "Request access for team members",

  catalogs: [serviceCatalog],
  categories: [itServicesCategory],

  // Attach variable sets with display order
  variableSets: [
    { variableSet: contactInfoSet, order: 100 },
    { variableSet: teamMembersSet, order: 200 }
  ],

  // Additional item-specific variables
  variables: {
    notes: MultiLineTextVariable({
      question: "Additional Notes",
      order: 100
    })
  },

  flow: "523da512c611228900811a37c97c2014"
});
```

---

## Common Variable Properties

| Property     | Type                  | Description                                   |
| ------------ | --------------------- | --------------------------------------------- |
| question     | string                | **Required.** Label text displayed to user.   |
| order        | number                | Display order (use increments of 100).        |
| mandatory    | boolean               | Whether field is required. Default: `false`.  |
| readOnly     | boolean               | Whether field is editable. Default: `false`.  |
| hidden       | boolean               | Whether field is visible. Default: `false`.   |
| tooltip      | string                | Hover help text.                              |
| exampleText  | string                | Placeholder text.                             |
| instructions | string                | Inline help text.                             |
| defaultValue | string                | Pre-filled value.                             |
| width        | 25 \| 50 \| 75 \| 100 | Field width percentage.                       |
| readRoles    | array                 | Roles that can read the variable.             |
| writeRoles   | array                 | Roles that can write to the variable.         |
| mapToField   | boolean               | Map to target table field (Record Producers). |
| field        | string                | Target field name when mapToField is true.    |

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

## Variable Set Properties

| Name          | Type    | Description                                                 |
| ------------- | ------- | ----------------------------------------------------------- |
| $id           | string  | **Required.** Unique identifier.                            |
| title         | string  | **Required.** Display title.                                |
| internalName  | string  | Internal name. Auto-generated from title if not provided.   |
| description   | string  | Description of the variable set.                            |
| type          | string  | `'singleRow'` (default) or `'multiRow'`.                    |
| layout        | string  | `'normal'`, `'2down'`, or `'2across'`. Default: `'normal'`. |
| order         | number  | Display order. Default: `100`.                              |
| displayTitle  | boolean | Show collapsible section header. Default: `false`.          |
| setAttributes | string  | Additional config (e.g., `"max_rows=10,collapsible=true"`). |
| readRoles     | array   | Roles that can view the variable set.                       |
| writeRoles    | array   | Roles that can modify values.                               |
| createRoles   | array   | Roles that can create instances (for multiRow).             |
| variables     | object  | Variable definitions for the set.                           |

## MRVS Constraints

### Unsupported Variable Types in Multi-Row Sets

- AttachmentVariable
- ContainerStartVariable / ContainerEndVariable / ContainerSplitVariable
- HtmlVariable
- CustomVariable / CustomWithLabelVariable
- RichTextLabelVariable
- UIPageVariable

### MRVS Limitations

- "Assign to Field" not supported
- Cannot add variables with read roles
- Set row limits using `max_rows` attribute
- Will not display if added to a container

## Important Notes

- Variables without names cannot be accessed by client scripts
- Use `readRoles` and `writeRoles` for sensitive fields
- Container variables must be properly paired (Start/End)
- Don't use the same variable name as a table field name (causes conflicts)
- Don't skip the `order` property (causes unpredictable ordering)
- Variable sets allow centralized updates across all using items
- Catalog UI Policies and Client Scripts can be scoped to a variable set using `appliesTo: 'set'`

---

## Common User Requests Mapping

| User Request                   | Agent Implementation                       |
| ------------------------------ | ------------------------------------------ |
| "Reusable contact fields"      | Variable Set (singleRow) attached to items |
| "Add multiple team members"    | Multi-Row Variable Set (MRVS)              |
| "Multi-column form layout"     | Container variables (Start/Split/End)      |
| "Conditional pricing on field" | Variable with pricingDetails               |

---

## Quick Reference

### ALWAYS

- Use `snake_case` for variable names
- Use `order` increments of 100
- Pair container variables properly (Start → Split → End)
- Use `mapToField: true` for simple field mappings in Record Producers

### NEVER

- Use the same variable name as a target table field name
- Skip the `order` property on variables
- Use AttachmentVariable, ContainerVariable, HtmlVariable, or CustomVariable in Multi-Row Variable Sets
