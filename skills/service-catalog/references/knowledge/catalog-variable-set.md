# CATALOG_VARIABLE_SET

# Catalog Variable Set - Templates and Examples

This knowledge source provides working code templates for ServiceNow Catalog Variable Sets. For guidelines, requirements, and patterns, activate the `service-catalog` skill first.

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

---

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

---

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

## Properties

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

## Role-Based Access

- `readRoles`: Roles that can view the variable set
- `writeRoles`: Roles that can modify values
- `createRoles`: Roles that can create instances (multiRow)

**Important**: Set-level permissions override variable-level permissions when access is denied at the set level.

## Important Notes

- Variable sets allow centralized updates across all using items
- Catalog UI Policies and Client Scripts can be scoped to a variable set using `appliesTo: 'set'`
- Use `displayTitle: true` to show collapsible section headers

---

## Common User Requests Mapping

| User Request                 | Agent Implementation                       |
| ---------------------------- | ------------------------------------------ |
| "Reusable contact fields"    | Variable Set (singleRow) attached to items |
| "Add multiple team members"  | Multi-Row Variable Set (MRVS)              |
| "Shared fields across items" | Variable Set with displayTitle             |
| "Grid/table data entry"      | Multi-Row Variable Set with setAttributes  |

---

## Quick Reference

### ALWAYS

- Use `displayTitle: true` to show collapsible section headers
- Use `setAttributes` for MRVS configuration (max_rows, collapsible)
- Attach variable sets via `variableSets: [{ variableSet, order }]`
- Use `appliesTo: 'set'` for UI Policies and Client Scripts scoped to a variable set

### NEVER

- Use AttachmentVariable, ContainerVariable, HtmlVariable, or CustomVariable in Multi-Row Variable Sets
- Add variables with read roles to Multi-Row Variable Sets
- Add Multi-Row Variable Sets inside containers
