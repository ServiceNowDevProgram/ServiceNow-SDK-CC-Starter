# Component Selection and Usage

## Contents

- [🚨 CRITICAL ANTI-PATTERN](#-critical-anti-pattern-)
- [MANDATORY Checkpoints](#mandatory-checkpoints)
- [❌ WRONG vs ✅ CORRECT Examples](#-wrong-vs--correct-examples)
- [Record List Pattern](#record-list-pattern)
- [Single Record Form Pattern](#single-record-form-pattern)
- [UI Components](#ui-components)
- [Decision Matrix](#decision-matrix)
- [Reading Component Documentation](#reading-component-documentation)
- [Final Validation Checklist](#final-validation-checklist)

## 🚨 CRITICAL ANTI-PATTERN 🚨

**NEVER EVER use `.map()` to iterate over ServiceNow records for display.**

This is the #1 most common mistake. Read this carefully:

### ❌ FORBIDDEN PATTERN

```tsx
// ❌ NEVER DO THIS - NO EXCEPTIONS
const [records, setRecords] = useState([]);
useEffect(() => {
  fetch("/api/now/table/incident")
    .then(r => r.json())
    .then(d => setRecords(d.result));
}, []);
return records.map(record => <div>{record.number}</div>);

// ❌ ALSO FORBIDDEN
return (
  <table>
    {tickets.map(t => (
      <tr>
        <td>{t.number}</td>
      </tr>
    ))}
  </table>
);
```

### ✅ REQUIRED PATTERN

```tsx
// ✅ ALWAYS DO THIS
<RecordListConnected
  table="incident"
  fields={["number", "short_description", "priority"]}
  onNewActionClicked={() => navigateToView("create", null, { reload: true })}
/>
```

**If you write `.map()` over ServiceNow records, STOP and use `RecordListConnected` instead.**

## MANDATORY Checkpoints

**BEFORE writing ANY TSX code, you MUST:**

1. ✅ Run `package_docs({ packageName: "@servicenow/react-components" })`
2. ✅ Read documentation for each component you will use
3. ✅ Map every UI element to a ServiceNow component
4. ✅ Verify NO raw HTML elements (`<button>`, `<input>`, `<select>`, `<table>`) in your plan
5. ✅ **Verify NO manual `.map()` loops over ServiceNow records in your plan**
6. ✅ **Confirm ALL record lists use `RecordListConnected`**

### Checkpoint Enforcement

FAILURE TO COMPLETE THESE CHECKPOINTS = REJECT THE IMPLEMENTATION

## ❌ WRONG vs ✅ CORRECT Examples

### Buttons

```tsx
// ❌ WRONG - Raw HTML
<button onClick={handleSubmit}>Submit</button>
<button className="primary-btn">Save</button>

// ✅ CORRECT - ServiceNow Component (check package_docs for exact event prop names)
import { Button } from "@servicenow/react-components/Button";
<Button label="Submit" /* use correct click event from docs */ />
<Button label="Save" variant="primary" />
```

> **⚠️ IMPORTANT:** Do NOT guess event handler prop names (e.g., `onClick` vs `onClicked`). ALWAYS read the component documentation via `package_docs` to get the correct prop names.

### Form Inputs

```tsx
// ❌ WRONG - Raw HTML
<input type="text" value={name} onChange={e => setName(e.target.value)} />
<select value={priority} onChange={e => setPriority(e.target.value)}>
  <option value="1">High</option>
  <option value="2">Medium</option>
</select>

// ✅ CORRECT - ServiceNow Components (check package_docs for exact prop/event names)
import { Input } from "@servicenow/react-components/Input";
import { Select } from "@servicenow/react-components/Select";
<Input value={name} /* use correct input event from docs */ />
<Select
  items={[{label: "High", value: "1"}, {label: "Medium", value: "2"}]}
  selectedItem={priority}
  /* use correct selection event from docs */
/>
```

### Cards/Containers

```tsx
// ❌ WRONG - Raw HTML
<div className="card">
  <h2>Ticket Details</h2>
  <p>Content here</p>
</div>;

// ✅ CORRECT - ServiceNow Component
import { Card } from "@servicenow/react-components/Card";
<Card>
  <h2>Ticket Details</h2>
  <p>Content here</p>
</Card>;
```

## Record List Pattern

**ALWAYS use `RecordListConnected` for displaying ServiceNow records.**

```tsx
// src/client/components/TicketList.tsx
import React from "react";
import { RecordListConnected } from "@servicenow/react-components/RecordListConnected";

export default function TicketList({ onSelectTicket }) {
  return (
    <RecordListConnected
      table="incident"
      query="active=true^priority=1"
      fields={[
        "number",
        "short_description",
        "priority",
        "state",
        "assigned_to"
      ]}
      onRowClick={record => onSelectTicket(record.sys_id)}
      onNewActionClicked={onNewClicked}
      pageSize={25}
    />
  );
}
```

### RecordListConnected Properties

- `table`: ServiceNow table name (e.g., "incident", "task", "cmdb_ci")
- `query`: Encoded query string (e.g., "active=true^priority=1")
- `fields`: Array of field names to display
- `pageSize`: Number of records per page (default: 25)
- **For event handlers** (row click, new action, etc.): ALWAYS check `package_docs` for exact prop names

**⚠️ Never use manual `fetch()` + `.map()` for record lists — see [CRITICAL ANTI-PATTERN](#-critical-anti-pattern-) above.**

## Single Record Form Pattern

**ALWAYS use `RecordProvider` for viewing/editing a single record.**

```tsx
// src/client/components/TicketDetail.tsx
import React from "react";
import { RecordProvider } from "@servicenow/react-components/RecordProvider";
import { RecordField } from "@servicenow/react-components/RecordField";
import { Button } from "@servicenow/react-components/Button";

export default function TicketDetail({ ticketId, onBack }) {
  return (
    <RecordProvider
      table="incident"
      sysId={ticketId}
    >
      <div className="ticket-detail">
        <RecordField
          name="number"
          readOnly
        />
        <RecordField name="short_description" />
        <RecordField name="description" />
        <RecordField name="priority" />
        <RecordField name="state" />
        <RecordField name="assigned_to" />

        <Button
          label="Back to List"
          variant="secondary"
          /* check package_docs for correct click event prop */
        />
      </div>
    </RecordProvider>
  );
}
```

### RecordProvider Usage

- `table`: ServiceNow table name
- `sysId`: Record sys_id to load
- `RecordField`: Automatically renders the correct field type (reference, choice, date, etc.)
- `RecordField.name`: Field name from the table
- `RecordField.readOnly`: Make field read-only

### Anti-Pattern: Manual Record Forms

```tsx
// ❌ WRONG - Never manually fetch and create form fields
export default function TicketDetail({ ticketId }) {
  const [ticket, setTicket] = useState({});

  useEffect(() => {
    fetch(`/api/now/table/incident/${ticketId}`)
      .then(res => res.json())
      .then(data => setTicket(data.result));
  }, [ticketId]);

  return (
    <div>
      <input value={ticket.number} readOnly />
      <input value={ticket.short_description} onChange={...} />
      <select value={ticket.priority} onChange={...}>
        <option value="1">1 - Critical</option>
        <option value="2">2 - High</option>
      </select>
    </div>
  );
}
```

## UI Components

Use ServiceNow components for consistent styling:

```tsx
import { Button } from "@servicenow/react-components/Button";
import { Card } from "@servicenow/react-components/Card";
import { Modal } from "@servicenow/react-components/Modal";

<Button variant="primary" label="Save" /* check docs for click event */ />
<Card><h2>Details</h2><RecordField name="short_description" /></Card>
<Modal /* check docs for open/close props */ title="Confirm">
  <Button label="Delete" /* check docs for click event */ />
</Modal>
```

Other available components: `Tabs`, `Tab`, `Alert`, `Heading`. Use `package_docs` to discover more.

## Decision Matrix

| Use Case                 | Component                        | Never Use                              |
| ------------------------ | -------------------------------- | -------------------------------------- |
| Display list of records  | `RecordListConnected`            | Manual `fetch()` + `map()` + `<table>` |
| View/edit single record  | `RecordProvider` + `RecordField` | Manual `fetch()` + `<input>` fields    |
| Buttons                  | `Button`                         | `<button>`                             |
| Form inputs (non-record) | `Input`, `Textarea`              | `<input>`, `<textarea>`                |
| Dropdowns (non-record)   | `Select`                         | `<select>`                             |
| Cards/panels             | `Card`                           | `<div className="card">`               |
| Modals                   | `Modal`                          | Custom modal implementation            |
| Tabs (UI only)           | `Tabs` + `Tab`                   | Custom tab implementation              |

**IMPORTANT:** For navigation between different views/pages, use URLSearchParams (`?tab=overview`) NOT `Tabs`. Use `Tabs` only for UI organization within a single view that doesn't need separate URLs.

## Reading Component Documentation

Before using any component, read its documentation:

```typescript
// Use this tool to get component docs
package_docs({ packageName: "@servicenow/react-components" });

// Then read specific component files
read_file({ file_path: "/path/to/RecordListConnected.md" });
read_file({ file_path: "/path/to/RecordProvider.md" });
```

## Final Validation Checklist

**After writing components, verify:**

- [ ] **🚨 Zero instances of `.map()` over ServiceNow records - MUST use `RecordListConnected`**
- [ ] **🚨 Zero instances of manual `fetch()` for record lists - MUST use `RecordListConnected`**
- [ ] Zero instances of `<button>` - replaced with `Button`
- [ ] Zero instances of `<input>` - replaced with `Input` or `RecordField`
- [ ] Zero instances of `<select>` - replaced with `Select` or `RecordField`
- [ ] Zero instances of `<textarea>` - replaced with `Textarea` or `RecordField`
- [ ] Zero instances of `<form>` - use `RecordProvider` for record forms
- [ ] Zero instances of `<table>` with manual mapping - replaced with `RecordListConnected`
- [ ] All interactive elements use components from `@servicenow/react-components/*`
- [ ] Event handler prop names verified via `package_docs` (NOT guessed — e.g., `onClick` vs `onClicked`)
