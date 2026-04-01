# GENERAL

<!-- No dedicated skill needed — referenced via get_knowledge_source from all fluent skills -->

ServiceNow Fluent (version 4.x)

Define application metadata in source code using the ServiceNow Fluent domain-specific programming language.

## Overview of ServiceNow Fluent

ServiceNow Fluent is a domain-specific language (DSL) based on TypeScript for defining the metadata files [sys_metadata] that make up applications and includes APIs for the different types of metadata, such as tables, roles, ACLs, business rules, and Automated Test Framework tests.

Developers define this metadata in a few lines of code instead of through a form or builder tool user interface. Applications created or converted with the ServiceNow IDE or ServiceNow SDK support developing in ServiceNow Fluent.

ServiceNow Fluent supports two-way synchronization, which allows changes to metadata to be synced from other Now Platform user interfaces into source code and changes to source code to be synced back to metadata across the instance.

## APIs

ServiceNow Fluent includes APIs for the following types of metadata.

- Access control lists (ACLs)
- Application menus
- Automated Test Framework tests
- Business rules
- Cross-scope privileges
- Import Sets - Data integration for importing data from external sources into ServiceNow tables
- Lists
- Properties
- Relationships and Related Lists - Used to define and configure relationships between tables and display related records on forms
- Records
- Roles
- Scripted REST APIs
- Script Includes
- Tables
- UI Policy - Dynamically controls the visibility, mandatory state, and read-only behavior of form fields based on defined conditions
- UI Action
- UI Pages - Used to create any visual component that can't be created using forms or lists, such as a custom UI or dashboard. **CRITICAL: UI PAGE cannot be used to create a component or element on the form.** .
- UI Formatters - Used to add components TO AN EXISTING UI FORM that render non-field content and is visually distinct from the fields on the form
-

## Core Difference between UI Pages and UI Formatters

UI Formatter → Used inside forms
UI Page → Used outside forms (standalone pages)

**Rule of thumb:** If the request mentions "on the form" or "to the form" or other similar keywords (activities, process flow, stages, timeline, progress, attached knowledge, checklist, breadcrumb, CI relationships, contextual search, variable editor, formatters), use UI FORMATTERS. If it's a standalone page/application, refer to UI Pages documentation.

## Usage

In files with the .now.ts extension, use objects in the ServiceNow Fluent APIs to define metadata in the application. You must also include the required imports for the APIs from @servicenow/sdk/core. For objects with server-side scripts, such as the BusinessRule object, you can import and use code from JavaScript modules.

You can use content from a separated JavaScript or HTML file in ServiceNow Fluent APIs by referring to the file from a property using the syntax `Now.include('path/to/file')`. The path is relative to the current file.

The following example includes the definitions of a table and business rule in the application. The business rule uses a function from a shared JavaScript file. The suggested file structure is shown below.

```
src/
    fluent/
        business-rules/
            log-state-change.now.ts
        tables/
            to-do.now.ts
    server/
        show-state-update.js
now.config.json
package.json
```

Here is the table definition:

```typescript
// src/fluent/tables/to-do.now.ts

import "@servicenow/sdk/global";
import {
  BooleanColumn,
  DateColumn,
  IntegerColumn,
  StringColumn,
  Table
} from "@servicenow/sdk/core";

//creates todo table, with three columns (deadline, status and task)
export const x_snc_example_to_do = Table({
  name: "x_snc_example_to_do",
  schema: {
    deadline: DateColumn({ label: "deadline" }),
    state: StringColumn({
      label: "State",
      choices: {
        ready: { label: "Ready", sequence: 0 },
        completed: { label: "Completed", sequence: 1 },
        in_progress: { label: "In Progress", sequence: 2 }
      }
    }),
    task: StringColumn({ label: "Task", maxLength: 120 }),
    priority: IntegerColumn({ label: "Priority", maxLength: 40 }),
    active: BooleanColumn({ label: "Active" })
  }
});
```

The business rule definition is as follows:

```typescript
// src/fluent/business-rules/log-state-change.now.ts

import "@servicenow/sdk/global";
import { BusinessRule } from "@servicenow/sdk/core";
import { showStateUpdate } from "../../server/show-state-update.js";

// creates a business rule that pops up state change message whenever a todo record is updated
BusinessRule({
  $id: Now.ID["br0"],
  action: ["update"],
  table: "x_snc_example_to_do",
  script: showStateUpdate,
  name: "LogStateChange",
  order: 100,
  when: "after",
  active: true
});
```

The shared JavaScript file contains the function that is called when the business rule runs:

```javascript
// src/server/show-state-update.js

import { gs } from "@servicenow/glide";

export function showStateUpdate(current, previous) {
  const currentState = current.getValue("state");
  const previousState = previous.getValue("state");

  gs.addInfoMessage(
    `state updated from "\${previousState}" to "\${currentState}"`
  );
}
```

For sample data, use the Record API. It is critical to understand the schema of the table before using the Record API.

```typescript
import "@servicenow/sdk/global";
import { Record } from "@servicenow/sdk/core";

Record({
  $id: Now.ID["example_id0"],
  table: "x_snc_example_to_do",
  data: {
    deadline: "2025-07-30", // Default format is 'yyyy-mm-dd' for DateColumn
    state: "ready",
    task: "Create a ServiceNow Fluent application"
  },
  priority: 1,
  active: true
});
```

## AI Integration with LLM

When developing applications that require AI-powered capabilities such as sentiment analysis, text generation, summarization, or other intelligent features, avoid simulating behavior with hard-coded keywords or mock responses. Instead, integrate with ServiceNow's `sn_generative_ai.LLMClient` API **strictly for textual analysis use cases\*** (e.g., sentiment detection, classification, summarization, text-based reasoning) to provide real AI functionality.

This applies to script-based metadata types that support server-side scripting. However, the approach varies by script type:

**Server-side scripts (can directly use LLM API):**

- Script Includes
- Scripted REST APIs
- Business Rules
- Background Scripts
- UI Actions (server-side)

**Client-side scripts (require server-side integration):**

- Client Scripts
- UI Actions (client-side)

### Implementation Guidelines

1. **Recommended Pattern - Use Script Includes**: Even though server-side scripts can directly use the LLM API, creating dedicated Script Includes is the recommended approach for better code organization, reusability, and maintainability.

2. **Client-side Integration**: For client scripts and client-side UI Actions, create a Script Include to handle LLM operations, then call it via GlideAjax or similar client-server communication methods.

3. **Proper API Usage**: Use ServiceNow's `sn_generative_ai.LLMClient` API for AI capabilities:

   ```js
   var llmClient = new sn_generative_ai.LLMClient(); // Accessible from any scope. Doesn't need to be imported.
   var prompt = "Your specific AI task prompt here";

   try {
     var result = llmClient.call({ prompt: prompt });
     if (result.status === "Success") {
       var response = result.response.trim();
       // Process the response appropriately
     } else {
       gs.error(result.response);
     }
   } catch (e) {
     gs.error(e.message);
   }
   ```

4. **Response Handling**: The result object contains:
   - `status`: "Success" or "Failure"
   - `response`: The model's output (String) or error message

   Always trim the response and safely detect if it's JSON or plain text. Parse JSON responses safely without throwing errors for invalid JSON.

5. **Error Handling**: Always implement robust error handling with try-catch blocks and use `gs.error()` for logging errors.

6. **Fallback Mechanisms**: Implement appropriate fallback handlers for cases where LLM API calls fail.

### Integration Pattern

Instead of creating mock or simulated AI behavior, follow this pattern:

1. Identify the AI requirement in the user's request
2. Create or reference a Script Include that handles the LLM interaction
3. Call the Script Include from your script-based metadata (Business Rule, Scripted REST API, etc.)
4. Process and return the actual AI-generated results

This ensures that applications provide genuine AI capabilities rather than simulated responses, delivering real value to users.
