---
name: wfa-flow
description: Guide for creating/editing ServiceNow Workflow Automation Flows (Flow Designer) to automate business processes and orchestrate actions. Use when user mentions flows, workflow automation, WFA, triggers, actions, or event-driven automation. Covers all components - trigger (when), action (what), flow logic (how), and flow structure (organization).Only supported in SDK 4.3.0 or higher.
user-invocable: true
---
> [!NOTE]
> This skill was authored for an agent runtime that provides two tools not
> available here. When you encounter references to these tools, resolve them
> as follows:
>
> **`get_knowledge_source`** — When instructed to use this tool with a
> knowledge source name like `BUSINESS_RULE`, read the file at
> `references/knowledge/<name>.md` where `<name>` is the lowercase,
> hyphenated version (e.g. `BUSINESS_RULE` →
> `references/knowledge/business-rule.md`).
>
> **`load_skill_resource`** — When instructed to use this tool with a file
> path like `references/column.md`, read that file directly from the
> `references/` directory within this skill.


# Workflow Automation Flow

## When to use this skill

- When creating/editing workflow automation flows
- When implementing event-driven automation
- When the user explicitly asks about WFA, flows, or workflow automation
- When users mention triggers, actions, flow logic, or any WFA components
- When implementing approval workflows or notification processes

**Important:** Flows consist of four integrated components - Flow Configuration (metadata), Trigger (when), Actions (what), and Flow Logic (how).

## Instructions

1. **Minimal Implementation:** Build only what's explicitly requested. Do not add logging, error handling, notifications, approvals, or additional logic unless specified in requirements.

2. **Action-to-Verb Mapping:** Each action verb in requirements (create, update, send, notify, etc.) maps to exactly ONE action. Multiple verbs require multiple actions.

3. **Flow Logic Only When Explicit:** Use flow logic constructs (if, forEach, etc.) only when explicitly mentioned in requirements. Do not add conditional checks, default branches, or decision paths unless stated.

4. **Post-Creation Summary:** After successfully creating a flow, always include in your summary that the user needs to **activate the flow from Flow Designer** before it will execute. Provide clear instructions that flows are created in an draft state by default and must be manually activated in the ServiceNow Flow Designer interface.

## Core Principles (CRITICAL)

1. **API Documentation First**: **ALWAYS** use `get_knowledge_source` tool to retrieve knowledge sources BEFORE creating flows:
   - **WFA_FLOW_TRIGGER_RECORD** - Record triggers (created, updated, createdOrUpdated)
   - **WFA_FLOW_TRIGGER_SCHEDULED** - Scheduled triggers (daily, weekly, monthly, repeat, runOnce)
   - **WFA_FLOW_TRIGGER_APPLICATION** - Application triggers (inboundEmail, slaTask, serviceCatalog, knowledgeManagement, remoteTableQuery)
   - **WFA_FLOW_ACTIONS_TABLE** - Table actions (create, update, delete, lookup operations)
   - **WFA_FLOW_ACTIONS_COMMUNICATION** - Communication actions (sendEmail, sendSms, sendNotification)
   - **WFA_FLOW_ACTIONS_CONTROL** - Control actions (log, waitForCondition)
   - **WFA_FLOW_ACTIONS_APPROVAL** - Approval actions (askForApproval)
   - **WFA_FLOW_ACTIONS_TASK** - Task actions (createTask)
   - **WFA_FLOW_ACTIONS_SERVICE_CATALOG** - Service Catalog actions (submitCatalogItemRequest, getCatalogVariables, createCatalogTask)
   - **WFA_FLOW_ACTIONS_SLA** - SLA actions (slaPercentageTimer)
   - **WFA_FLOW_LOGICS** - Flow control structures (if/elseIf/else, forEach, exitLoop, skipIteration, endFlow)

2. **Load Reference Guidance**: Use `load_skill_resource` with skill name "wfa-flow" to access detailed patterns and examples from the `references/` folder.

3. **Verify API Syntax**: Knowledge sources contain authoritative API signatures. References provide context and patterns but point back to knowledge sources for exact syntax.

## Temporal Requirements Analysis

**Critical:** Distinguish between one-time events and ongoing monitoring:

| User Says                                                  | Trigger Type          | Pattern                            | Example                            |
| ---------------------------------------------------------- | --------------------- | ---------------------------------- | ---------------------------------- |
| "when created", "when updated", "when X happens"           | **Record Trigger**    | Event-driven, fires once per event | created, updated, createdOrUpdated |
| "while active", "as long as", "monitor", "check regularly" | **Scheduled Trigger** | Time-driven, repeats periodically  | daily, weekly, repeat              |
| "every day/hour/week", "periodically", "continuously"      | **Scheduled Trigger** | Time-driven, repeats at intervals  | daily, repeat                      |
| "at 9 AM", "on Monday", "first of month"                   | **Scheduled Trigger** | Time-driven, runs on schedule      | daily, weekly, monthly             |

**Example Distinction:**

- ✅ "When incident is created, notify manager" → Record trigger (created) + sendEmail
- ✅ "While incident is active, log status every day" → Scheduled trigger (daily) + lookUpRecords + forEach + log

## Planning Your Flow

When translating requirements into a flow, follow this structured approach:

### Step 1: Identify the Trigger

- **What event should activate the flow?**
  - Record events: created, updated, createdOrUpdated (one-time, event-driven)
  - Scheduled events: daily, weekly, monthly, repeat, runOnce (time-driven, repeating)
  - Application events: inboundEmail, slaTask, serviceCatalog, knowledgeManagement, remoteTableQuery (app-specific)

### Step 2: Define the Logic

- **What conditions determine different paths?**
  - Use `if/elseIf/else` for branching (priority-based routing, conditional actions)
  - Use `forEach` for bulk processing (process multiple records)
  - Use `exitLoop` to break loops when condition met (early exit, search)
  - Use `skipIteration` to filter during loops (validation, conditional skip)
  - Use `endFlow` to terminate flow early (error handling, completion)

### Step 3: Select Actions

- **What operations are needed?**
  - **Table operations**: createRecord, updateRecord, deleteRecord, lookUpRecord, lookUpRecords, updateMultipleRecords, createOrUpdateRecord
  - **Communications**: sendEmail, sendNotification, sendSms
  - **Approvals**: askForApproval
  - **Task management**: createTask
  - **Control**: log (for debugging/tracking)

## Flow Construction Checklist

Before implementing a flow, verify these requirements:

- ✅ **Identify trigger event type** - What event activates the flow?
- ✅ **Define trigger conditions/filters** - Which records should trigger the flow?
- ✅ **Plan conditional logic paths** - What branching or loops are needed?
- ✅ **Select required actions** - What operations must the flow perform?
- ✅ **Define data flow between actions** - How do action outputs feed into subsequent actions?

**Critical Guidelines:**

- ⚠️ **If you are unaware of any details, ask the user to provide required information** - Never assume or infer requirements
- ⚠️ **If you are confident to do any action which the user didn't mention, you MUST take confirmation from the user before including it into the flow** - Only build what was explicitly requested
- ⚠️ **Do not add**: logging (unless for debugging), error handling, notifications, approvals, or additional logic unless explicitly specified

## Flow Configuration

Configure flow metadata and execution context:

| Property       | Values                            | Use When                                                                     |
| -------------- | --------------------------------- | ---------------------------------------------------------------------------- |
| `runAs`        | 'user' (default), 'system'        | 'system' for admin tasks, bypassing ACLs; 'user' to respect user permissions |
| `runWithRoles` | Array of role sys_ids             | Need temporary elevated privileges without full system access                |
| `flowPriority` | 'LOW', 'MEDIUM' (default), 'HIGH' | 'HIGH' for critical/time-sensitive; 'LOW' for background tasks               |

## Component Quick Reference

### Triggers (When)

| Type            | Key Triggers                          | Use For          |
| --------------- | ------------------------------------- | ---------------- |
| **Record**      | created, updated, createdOrUpdated    | Data changes     |
| **Scheduled**   | daily, weekly, repeat                 | Time-based tasks |
| **Application** | inboundEmail, slaTask, serviceCatalog | App events       |

**Learn more:** Use `load_skill_resource` with skill "wfa-flow" and path:

**Record Triggers:**

- `references/triggers/record/created-trigger.md` - Fires when new record is created
- `references/triggers/record/updated-trigger.md` - Fires when record is modified
- `references/triggers/record/created-or-updated-trigger.md` - Fires on create or update

**Scheduled Triggers:**

- `references/triggers/scheduled/daily-trigger.md` - Executes once per day
- `references/triggers/scheduled/weekly-trigger.md` - Executes once per week
- `references/triggers/scheduled/monthly-trigger.md` - Executes once per month
- `references/triggers/scheduled/repeat-trigger.md` - Executes at regular intervals
- `references/triggers/scheduled/run-once-trigger.md` - Executes once at specific date/time

**Application Triggers:**

- `references/triggers/application/inbound-email-trigger.md` - Email received
- `references/triggers/application/sla-task-trigger.md` - SLA events
- `references/triggers/application/service-catalog-trigger.md` - Service catalog request items
- `references/triggers/application/knowledge-management-trigger.md` - Knowledge article events
- `references/triggers/application/remote-table-query-trigger.md` - Remote table queries

### Actions (What)

| Category              | Key Actions                                 | Use For              |
| --------------------- | ------------------------------------------- | -------------------- |
| **Record Operations** | createRecord, updateRecord, lookUpRecords   | CRUD operations      |
| **Communication**     | sendEmail, sendNotification, sendSms        | Messaging            |
| **Approvals**         | askForApproval                              | Approval workflows   |
| **Service Catalog**   | submitCatalogItemRequest, createCatalogTask | Catalog provisioning |
| **Attachments**       | moveAttachment, copyAttachment              | File handling        |

**Table Operations:**

- `references/actions/table/create-record.md` - Create new records
- `references/actions/table/update-record.md` - Update existing records
- `references/actions/table/delete-record.md` - Delete records
- `references/actions/table/lookup-record.md` - Find single record
- `references/actions/table/lookup-records.md` - Find multiple records
- `references/actions/table/update-multiple-records.md` - Bulk update records
- `references/actions/table/create-or-update-record.md` - Upsert operations

**Communication:**

- `references/actions/communication/send-email.md` - Send custom emails
- `references/actions/communication/send-notification.md` - Send notifications template
- `references/actions/communication/send-sms.md` - Send SMS messages

**Control:**

- `references/actions/control/log.md` - Write messages to execution log

**SLA:**

- `references/actions/sla/sla-percentage-timer.md` - Pause until SLA percentage reached

**Service Catalog:**

- `references/actions/service-catalog/submit-catalog-item-request.md` - Submit catalog item requests programmatically
- `references/actions/service-catalog/get-catalog-variables.md` - Retrieve variables from template catalog items
- `references/actions/service-catalog/create-catalog-task.md` - Create fulfillment tasks on request items

### Flow Logic (How)

| Type            | Constructs                       | Use For          |
| --------------- | -------------------------------- | ---------------- |
| **Conditional** | if, elseIf, else                 | Branching        |
| **Loops**       | forEach, skipIteration, exitLoop | Iteration        |
| **Control**     | endFlow                          | Flow termination |

**Condition syntax:** Encoded query format - use `=` not `==`, `^` for AND, `^OR` for OR

- `references/flow-logics/conditional-if.md` - if/elseIf/else branching logic
- `references/flow-logics/loop-foreach.md` - forEach iteration over arrays and records
- `references/flow-logics/exit-loop.md` - Break out of loops early (search, limits)
- `references/flow-logics/skip-iteration.md` - Skip current iteration (filtering, validation)
- `references/flow-logics/end-flow.md` - endFlow to terminate flow execution

## Component Selection Guide

Use these tables to select the right component based on your use case:

### Triggers by Use Case

| Use Case                | Trigger Type                     | Example Scenario                          |
| ----------------------- | -------------------------------- | ----------------------------------------- |
| New record automation   | trigger.record.created           | Auto-assign incidents when created        |
| Record change detection | trigger.record.updated           | Escalate priority when incident updated   |
| Either create or update | trigger.record.createdOrUpdated  | Audit logging for new or modified records |
| Daily maintenance tasks | trigger.scheduled.daily          | Cleanup old records every morning         |
| Regular interval checks | trigger.scheduled.repeat         | Check system status every 15 minutes      |
| Email-based workflows   | trigger.application.inboundEmail | Parse support emails and create tickets   |

### Actions by Operation Type

| Operation                   | Action                   | Use When                                      |
| --------------------------- | ------------------------ | --------------------------------------------- |
| Create new record           | createRecord             | Creating child records, using templates       |
| Modify existing record      | updateRecord             | Updating field values on records              |
| Remove record               | deleteRecord             | Cleanup tasks, archival processes             |
| Find one specific record    | lookUpRecord             | Getting a specific user, group, or record     |
| Find multiple records       | lookUpRecords            | Bulk processing, finding all matching records |
| Update many records         | updateMultipleRecords    | Mass field changes across records             |
| Create or update (upsert)   | createOrUpdateRecord     | Import operations, sync from external systems |
| Send custom email           | sendEmail                | User notifications with static content        |
| Send templated notification | sendNotification         | Dynamic notifications using templates         |
| Send SMS alert              | sendSms                  | Urgent alerts to mobile devices               |
| Request approval            | askForApproval           | Multi-tier approval workflows                 |
| Create task                 | createTask               | Follow-up tasks, work assignment              |
| Submit catalog request      | submitCatalogItemRequest | Programmatic catalog ordering, provisioning   |
| Get catalog variables       | getCatalogVariables      | Template-based request variable population    |
| Create catalog task         | createCatalogTask        | Fulfillment tasks on catalog request items    |

### Flow Logic by Pattern

| Pattern            | Flow Logic Construct | Use When                              |
| ------------------ | -------------------- | ------------------------------------- |
| Branching paths    | if/elseIf/else       | Different actions based on conditions |
| Bulk processing    | forEach              | Process each record in a collection   |
| Early termination  | endFlow              | Stop flow on error or completion      |
| Skip items in loop | skipIteration        | Filter items during iteration         |
| Break out of loop  | exitLoop             | Stop loop when condition is met       |

## Flow Patterns (Load When Building Complete Flows)

Real-world flow examples showing trigger + actions + flow logic integration. Load these to understand complete flow structure and syntax.

**Load pattern references using `load_skill_resource` with skill "wfa-flow" and path:**

**Foundational Patterns:**

1. **Simple Automation** - `references/patterns/simple-automation.md`
   - **Structure:** Trigger → Single Action
   - **Example:** Auto-assign high-priority incidents
   - **Load when:** Requirements have 1 trigger + 1 action with no branching

2. **Sequential Steps** - `references/patterns/sequential-steps.md`
   - **Structure:** Trigger → Action 1 → Action 2 → Action 3
   - **Example:** Create incident → Find group → Assign → Notify
   - **Load when:** Requirements need multiple actions in specific order (step 1, then 2, then 3)

3. **Conditional Routing** - `references/patterns/conditional-routing.md`
   - **Structure:** Trigger → if/elseIf/else → Different Actions
   - **Example:** Route by priority (P1=team1, P2=team2, other=team3)
   - **Load when:** Requirements include "if", "based on", "depending on", or different actions for different conditions

**Advanced Patterns:**

4. **Record Iteration** - `references/patterns/record-iteration.md`
   - **Structure:** Trigger → lookUpRecords → forEach → Action per Record
   - **Example:** Find unassigned incidents → Assign each to round-robin group
   - **Load when:** Requirements mention "all", "each", "every", processing multiple records, or bulk operations

5. **Approval Workflow** - `references/patterns/approval-workflow.md`
   - **Structure:** Trigger → lookUpRecords (approvers) → askForApproval → Route by Result
   - **Example:** Expense approval with manager/director tiers
   - **Load when:** Requirements include approval, manager review, or multi-level authorization

6. **Notification Flow** - `references/patterns/notification-flow.md`
   - **Structure:** Trigger → Email + SMS + Notification (parallel)
   - **Example:** Critical incident alerts via multiple channels
   - **Load when:** Requirements need multiple notifications, email + SMS, or notifying groups/teams

7. **Monitoring Pattern** - `references/patterns/monitoring-pattern.md`
   - **Structure:** Scheduled Trigger → lookUpRecords → forEach → Action per Record
   - **Example:** Every day, find active incidents → Log status for each
   - **Load when:** Requirements include "while", "monitor", "periodically", "every day/hour", ongoing checks, or temporal continuity (scheduled triggers)

**Service Catalog Patterns:**

**⚠️ Trigger Selection for Catalog Workflows:**

- **Use `trigger.application.serviceCatalog`** when automating service catalog request fulfillment (approvals, tasks, notifications on existing requests)

**Best Practice:** Prefer `serviceCatalog` trigger for catalog request processing. It provides direct access to request item context and executes in the proper catalog workflow lifecycle.

8. **Catalog Approval and Fulfillment** - `references/patterns/catalog-approval-fulfillment.md`
   - **Structure:** Service Catalog Trigger → askForApproval → createCatalogTask → Notifications
   - **Example:** Catalog request created → Manager approves → Create fulfillment task → Notify user
   - **Load when:** Requirements include catalog approvals, fulfillment workflows, or multi-step provisioning

**When to Load Patterns:**

- User asks for complete flow examples
- Requirements are complex and need full context
- Learning correct syntax for trigger + action + flow logic integration
- Understanding data flow between components

## Flow API and Examples

**ALWAYS retrieve knowledge sources for complete API documentation, syntax, parameters, and working examples:**

Use `get_knowledge_source` tool to load:

**For Triggers:**

- **WFA_FLOW_TRIGGER_RECORD** - Full API for created, updated, createdOrUpdated triggers with parameters, outputs, and examples
- **WFA_FLOW_TRIGGER_SCHEDULED** - Full API for daily, weekly, monthly, repeat, runOnce triggers with time/duration specifications
- **WFA_FLOW_TRIGGER_APPLICATION** - Full API for inboundEmail, slaTask, serviceCatalog, knowledgeManagement, remoteTableQuery triggers

**For Actions:**

- **WFA_FLOW_ACTIONS_TABLE** - Full API for createRecord, updateRecord, deleteRecord, lookUpRecord, lookUpRecords, updateMultipleRecords, createOrUpdateRecord
- **WFA_FLOW_ACTIONS_COMMUNICATION** - Full API for sendEmail, sendNotification, sendSms with all parameters and examples
- **WFA_FLOW_ACTIONS_CONTROL** - Full API for log, waitForCondition actions
- **WFA_FLOW_ACTIONS_APPROVAL** - Full API for askForApproval with approval rules and helper functions
- **WFA_FLOW_ACTIONS_TASK** - Full API for createTask, createCatalogTask actions
- **WFA_FLOW_ACTIONS_SLA** - Full API for slaPercentageTimer action

**For Flow Logic:**

- **WFA_FLOW_LOGICS** - Full API for if/elseIf/else, forEach, exitLoop, skipIteration, endFlow with condition syntax and examples

**Each knowledge source contains:**

- Complete API signatures with all parameters
- Input/output specifications
- Data pill types and usage
- Working code examples
- Integration patterns
- Best practices

## Important Notes

- All flows must be created under the `fluent/flows` folder
- Always wrap complex field values in TemplateValue() for createRecord, updateRecord actions
- TemplateValue, Time, and Duration are available globally (don't import)
- Every flow requires exactly one trigger
- Background execution (run_flow_in: 'background') is recommended for most flows
- In conditions, always use template literals: `` `${wfa.dataPill(...)}=value` ``

## Critical Syntax Requirements

**MANDATORY patterns - violations cause runtime errors:**

1. **TemplateValue**: Use `TemplateValue({ })` not `wfa.TemplateValue({ })`

2. **Template literal wrapping**: Required for `recipients` in sendSms, `record` in deleteRecord within forEach
   - Example: `recipients: \`${wfa.dataPill(user.phone, "string")}\``

3. **Email conditions**: Use `LIKE` not `CONTAINS` in inboundEmail `email_conditions`

4. **Trigger paths** (trigger-type specific):
   - **Record triggers** (created/updated): `_params.trigger.current.field` | record ref: `_params.trigger.current`
   - **Application triggers** (inboundEmail/slaTask/serviceCatalog): `_params.trigger.field` | record ref varies by trigger type

5. **Parameter names** (action-specific):
   - createRecord/updateRecord → `values:` | createTask/updateMultipleRecords → `field_values:`
   - lookUpRecords → `table:`, `conditions:` | createTask → `task_table:`

6. **Time functions**: `Time.addDays()`, `Time.nowDateTime()` don't exist. Use dataPills: `wfa.dataPill(_params.trigger.due_date, "glide_date_time")`

7. **Non-existent actions**: `deleteMultipleRecords` doesn't exist. Use: lookUpRecords → forEach → deleteRecord

## When to Load Knowledge Sources

**Use `get_knowledge_source` tool when you need detailed API specifications:**

### WFA_DATAPILLS Knowledge Source

Load this when working with data pills or need to understand data pill types:

```
Use get_knowledge_source tool to get the WFA_DATAPILLS knowledge source
```

**When to load:**

- Working with trigger outputs and need to know field types
- Accessing action outputs and need to understand available data types
- Uncertain about which FlowDataType to use ('string', 'reference', 'integer', etc.)
- Need examples of data pill dot-walking (e.g., `_params.trigger.current.assigned_to.email`)
- Working with complex data types ('array.object', 'array.string', 'records')

**Knowledge source provides:**

- Complete `wfa.dataPill()` API signature
- All FlowDataType enum values (50+ types)
- Data pill usage patterns for triggers, actions, and subflows
- Type mapping for ServiceNow fields (string, reference, glide_date_time, etc.)
- Examples of multi-level field access

### Other Knowledge Sources

- **WFA_FLOW_ACTIONS_TABLE**: Table action APIs (createRecord, updateRecord, lookUpRecord, etc.)
- **WFA_FLOW_ACTIONS_COMMUNICATION**: Communication actions (sendEmail, sendSms, sendNotification)
- **WFA_FLOW_ACTIONS_APPROVAL**: Approval actions and rules
- **WFA_FLOW_LOGICS**: Flow logic constructs (if, forEach, exitLoop, etc.)
- **WFA*FLOW_TRIGGERS*\***: Trigger specifications (record, scheduled, application)

**Rule:** Load knowledge sources BEFORE writing complex code to ensure correct API usage.

## Avoidance

- Don't add logging unless explicitly requested for debugging
- Don't infer requirements or add "helpful" extra behavior
- Don't create subflows (not supported - use flows only)
- Don't hardcode sys_ids - use run_query tool to fetch them
- **⚠️ CRITICAL - Don't assign data pills to const variables:** WFA flows are declarative. Always use `wfa.dataPill()` directly in action parameters, never store in variables.

**❌ WRONG - Assigning data pills to const:**

```typescript
const recordId = wfa.dataPill(result.record, 'reference');
const status = wfa.dataPill(result.status, 'string');
wfa.action(action.core.updateRecord, {...}, {
  record: recordId,  // WRONG!
  values: TemplateValue({ state: status })  // WRONG!
});
```

**✅ CORRECT - Use data pills directly:**

```typescript
wfa.action(action.core.updateRecord, {...}, {
  record: wfa.dataPill(result.record, 'reference'),  // CORRECT!
  values: TemplateValue({
    state: wfa.dataPill(result.status, 'string')  // CORRECT!
  })
});
```

- **⚠️ CRITICAL - Template literal limitations:** Template literals with data pills work ONLY in specific fields: `ah_subject`, `log_message`. They do NOT work in `ah_body`, `TemplateValue` fields, or `message` (SMS).

**❌ WRONG - Template literals in unsupported fields:**

```typescript
wfa.action(action.core.sendEmail, {...}, {
  ah_subject: `Incident ${wfa.dataPill(number, 'string')}`,  // ✅ Works in ah_subject
  ah_body: `Details: ${wfa.dataPill(desc, 'string')}`  // ❌ Does NOT work in ah_body!
});
values: TemplateValue({
  notes: `Status: ${wfa.dataPill(status, 'string')}`  // ❌ Does NOT work in TemplateValue!
});
```

**✅ CORRECT - Static strings in unsupported fields:**

```typescript
wfa.action(action.core.sendEmail, {...}, {
  ah_subject: `Incident ${wfa.dataPill(number, 'string')}`,  // ✅ Works in ah_subject
  ah_body: 'Please check your assignment for details.'  // ✅ Static string
});
values: TemplateValue({
  work_notes: 'Record updated by workflow'  // ✅ Static string
});
```

- **⚠️ CRITICAL - Don't mix JavaScript and DSL paradigms:** WFA flows are declarative configurations, not procedural code. Don't try to abstract, optimize, or make flows more "programmatic."

**❌ WRONG - Trying to optimize with JavaScript abstractions:**

```typescript
// Trying to make it "reusable" - WRONG!
const getFieldValue = field =>
  wfa.dataPill(_params.trigger.current[field], "string");
const buildCondition = (field, value) => `${field}=${value}`;
// Using abstractions - WRONG!
conditions: buildCondition("priority", getFieldValue("priority"));
```

**✅ CORRECT - Direct, declarative specification:**

```typescript
// Follow documented patterns exactly - CORRECT!
conditions: `priority=${wfa.dataPill(_params.trigger.current.priority, "string")}^active=true`;
```

**Key principle:** WFA flows describe "what should happen" (declarative), not "how to make it happen" (imperative). Copy documented patterns exactly - don't try to improve them.
