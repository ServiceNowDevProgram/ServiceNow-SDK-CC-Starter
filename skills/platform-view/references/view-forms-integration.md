# Forms Integration with UI Views

Guide for creating forms, sections, and elements with UI Views using the Record API.

## Contents

- [Understanding Forms and UI Views](#understanding-forms-and-ui-views)
- [CRITICAL: Form-Section Linking](#critical-form-section-linking)
- [Complete Example: Form with Multiple Sections](#complete-example-form-with-multiple-sections)
- [Form Element Types](#form-element-types)
- [Best Practices](#best-practices)

## Understanding Forms and UI Views

**Hierarchy**: UI View → Form → Sections → Elements (fields/formatters/related lists)

**Key Concept**: A UI View determines which form layout to display. Multiple forms can exist for the same table, each associated with a different view.

**How it works**: View Rules select the view → ServiceNow loads the form for that view → Form sections and elements render

## CRITICAL: Form-Section Linking

**ServiceNow requires explicit linking between forms and sections** using the `sys_ui_form_section` table:

- `sys_ui_form` = The form container
- `sys_ui_section` = Section definitions (what sections exist)
- `sys_ui_form_section` = **The link/join table** (which sections appear on which forms, in what order)

**Without `sys_ui_form_section` records, your form will appear EMPTY even if sections and fields exist.**

Think of it like a book:

- Form = The book
- Section = Chapter definitions
- Form-Section Link = Table of contents (which chapters, in what order)
- Elements = Content in each chapter

## Complete Example: Form with Multiple Sections

```typescript
import { Record } from "@servicenow/sdk/core";
import { managerView } from "./views";

// 1. Create Form
export const managerForm = Record({
  $id: Now.ID["manager-form"],
  table: "sys_ui_form",
  data: { name: "incident", view: managerView, active: true }
});

// 2. Create Multiple Sections
export const detailsSection = Record({
  $id: Now.ID["details-section"],
  table: "sys_ui_section",
  data: {
    name: "incident",
    view: managerView,
    caption: "Case Details",
    position: 0
  }
});

export const assignmentSection = Record({
  $id: Now.ID["assignment-section"],
  table: "sys_ui_section",
  data: {
    name: "incident",
    view: managerView,
    caption: "Assignment",
    position: 1
  }
});

export const notesSection = Record({
  $id: Now.ID["notes-section"],
  table: "sys_ui_section",
  data: {
    name: "incident",
    view: managerView,
    caption: "Notes",
    position: 2
  }
});

// 3. CRITICAL: Link ALL Sections to Form
export const formDetailsLink = Record({
  $id: Now.ID["form-details-link"],
  table: "sys_ui_form_section",
  data: {
    sys_ui_form: managerForm,
    sys_ui_section: detailsSection,
    position: 0
  }
});

export const formAssignmentLink = Record({
  $id: Now.ID["form-assignment-link"],
  table: "sys_ui_form_section",
  data: {
    sys_ui_form: managerForm,
    sys_ui_section: assignmentSection,
    position: 1
  }
});

export const formNotesLink = Record({
  $id: Now.ID["form-notes-link"],
  table: "sys_ui_form_section",
  data: {
    sys_ui_form: managerForm,
    sys_ui_section: notesSection,
    position: 2
  }
});

// 4. Add Fields to Sections
export const numberField = Record({
  $id: Now.ID["number-field"],
  table: "sys_ui_element",
  data: {
    element: "number",
    sys_ui_section: detailsSection,
    position: 0,
    type: "element"
  }
});

export const priorityField = Record({
  $id: Now.ID["priority-field"],
  table: "sys_ui_element",
  data: {
    element: "priority",
    sys_ui_section: detailsSection,
    position: 10,
    type: "element"
  }
});

export const assignedToField = Record({
  $id: Now.ID["assigned-to-field"],
  table: "sys_ui_element",
  data: {
    element: "assigned_to",
    sys_ui_section: assignmentSection,
    position: 0,
    type: "element"
  }
});

export const assignmentGroupField = Record({
  $id: Now.ID["assignment-group-field"],
  table: "sys_ui_element",
  data: {
    element: "assignment_group",
    sys_ui_section: assignmentSection,
    position: 10,
    type: "element"
  }
});

// 5. Add Formatters (optional)
export const approvalFormatter = Record({
  $id: Now.ID["approval-formatter"],
  table: "sys_ui_element",
  data: {
    element: "approval_history",
    sys_ui_section: notesSection,
    position: 0,
    type: "formatter",
    sys_ui_formatter: "approval_summarizer"
  }
});

// 6. Add Related Lists (optional)
export const tasksRelatedList = Record({
  $id: Now.ID["tasks-rel-list"],
  table: "sys_ui_element",
  data: {
    element: "tasks",
    sys_ui_section: notesSection,
    position: 10,
    type: "list"
  }
});
```

## Form Element Types

| Type           | Description           |
| -------------- | --------------------- |
| `element`      | Standard field        |
| `formatter`    | Custom formatter      |
| `list`         | Related list          |
| `.split`       | Layout: split columns |
| `.begin_split` | Layout: begin split   |
| `.end_split`   | Layout: end split     |
| `.space`       | Layout: empty space   |

## Form Layout Architecture

### Column Layout with Splits

Use splits to arrange fields in 2 columns. Pattern:

```
.begin_split → opens 2-column area
  [left column fields]
.split       → column divider
  [right column fields]
.end_split   → closes 2-column area
[full-width content: text areas, related lists]
```

**Split element example:**

```typescript
export const beginSplit = Record({
  $id: Now.ID["section-begin-split"],
  table: "sys_ui_element",
  data: {
    element: ".begin_split",
    sys_ui_section: detailsSection,
    position: 0,
    type: ".begin_split"
  }
});
// left column fields at positions 10, 11, 12...
// .split at position 20
// right column fields at positions 30, 31, 32...
// .end_split at position 40
// full-width fields (text areas) at positions 50+
```

**Split rules:**

- **Use splits for:** short fields — state, priority, category, caller, assigned_to, impact, urgency
- **Never split:** text areas (description, work_notes) — always full width after `.end_split`
- **Never split:** related lists — always full width in their own dedicated section
- **Balance columns:** keep left and right field counts equal or within 1

### Standard Section Layout

| Section       | Position | Layout      | Content                                             |
| ------------- | -------- | ----------- | --------------------------------------------------- |
| Header        | 0        | 2-col split | number+state, priority+category, caller+assigned_to |
| Details       | 1        | 2-col split | impact+urgency, CI, dates                           |
| Notes         | 2        | Full width  | description, work_notes (text areas — no split)     |
| Resolution    | 3        | Full width  | close_code, resolution_notes                        |
| Related Lists | 4        | Full width  | Dedicated section — never mix with fields           |

## Best Practices

**Naming**: Use consistent prefixes across views, forms, sections, elements

**Positioning**: Use increments of 10 (0, 10, 20...) to allow easy insertion later

**Section Organization**:

- Details: Core record info
- Assignment: Ownership fields
- Notes: Comments/work notes
- Related: Related lists in own dedicated section at bottom

**View-Specific Guidelines**:

- **Default**: All fields, sections, formatters, related lists
- **Mobile**: 5-8 essential fields only, 1-2 sections, no formatters/related lists/splits
- **Manager**: Detailed info, approval formatters, related lists, metrics
