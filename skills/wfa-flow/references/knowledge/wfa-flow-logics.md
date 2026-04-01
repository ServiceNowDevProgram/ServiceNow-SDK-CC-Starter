# WFA_FLOW_LOGICS

The Flow Logic API reference provides complete technical specifications for flow control constructs. This document contains API signatures, parameter definitions, types, and syntax examples for conditional logic, loops, loop control, and flow termination. For conceptual guidance, design patterns, best practices, and when-to-use information, refer to skill references.

## Flow Logic Usage Pattern

All flow logic constructs follow the same usage pattern:

```typescript
wfa.flowLogic.<construct>(
  { $id: Now.ID['logic_id'], ...params },
  callback?
)
```

---

## Conditional Logic

### wfa.flowLogic.if

Evaluates a condition and executes the callback if true.

**Signature:**

```typescript
wfa.flowLogic.if(
  params: {
    $id: string,
    condition: string,
    label?: string,
    annotation?: string
  },
  body: () => void
)
```

**Parameters:**

| Parameter  | Type   | Default | Mandatory | Description                          |
| ---------- | ------ | ------- | --------- | ------------------------------------ |
| $id        | string | -       | Yes       | Unique identifier (Now.ID['value'])  |
| condition  | string | -       | Yes       | Encoded query expression to evaluate |
| label      | string | -       | No        | Display label for this branch        |
| annotation | string | -       | No        | Description/comment                  |

**Example:**

```typescript
wfa.flowLogic.if(
  {
    $id: Now.ID["check_priority"],
    condition: `${wfa.dataPill(_params.trigger.current.priority, "string")}=1`,
    annotation: "Check if priority is critical"
  },
  () => {
    // Actions for priority 1 incidents
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["escalate"] },
      {
        table_name: "incident",
        record: wfa.dataPill(_params.trigger.current, "reference"),
        values: TemplateValue({ state: 2 })
      }
    );
  }
);
```

---

### wfa.flowLogic.elseIf

Evaluates a condition if previous conditions were false. Must follow `if` or `elseIf`.

**Signature:**

```typescript
wfa.flowLogic.elseIf(
  params: {
    $id: string,
    condition: string,
    label?: string,
    annotation?: string
  },
  body: () => void
)
```

**Parameters:**

| Parameter  | Type   | Default | Mandatory | Description                          |
| ---------- | ------ | ------- | --------- | ------------------------------------ |
| $id        | string | -       | Yes       | Unique identifier (Now.ID['value'])  |
| condition  | string | -       | Yes       | Encoded query expression to evaluate |
| label      | string | -       | No        | Display label for this branch        |
| annotation | string | -       | No        | Description/comment                  |

**Example:**

```typescript
wfa.flowLogic.elseIf(
  {
    $id: Now.ID["check_priority_2"],
    condition: `${wfa.dataPill(_params.trigger.current.priority, "string")}=2`,
    annotation: "Check if priority is high"
  },
  () => {
    // Actions for priority 2 incidents
  }
);
```

---

### wfa.flowLogic.else

Executes callback if all previous conditions were false. Must follow `if` or `elseIf`.

**Signature:**

```typescript
wfa.flowLogic.else(
  params: {
    $id: string,
    annotation?: string
  },
  body: () => void
)
```

**Parameters:**

| Parameter  | Type   | Default | Mandatory | Description                         |
| ---------- | ------ | ------- | --------- | ----------------------------------- |
| $id        | string | -       | Yes       | Unique identifier (Now.ID['value']) |
| annotation | string | -       | No        | Description/comment                 |

**Example:**

```typescript
wfa.flowLogic.else(
  { $id: Now.ID["default_case"], annotation: "Handle all other priorities" },
  () => {
    // Default actions
  }
);
```

---

## Loops

### wfa.flowLogic.forEach

Iterates over an array, executing the callback for each item.

**Signature:**

```typescript
wfa.flowLogic.forEach<TArray>(
  items: TArray,
  config: {
    $id: string,
    annotation?: string
  },
  body?: (item: ExtractArrayElement<TArray>) => void
)
```

**Parameters:**

| Parameter  | Type     | Default | Mandatory | Description                                      |
| ---------- | -------- | ------- | --------- | ------------------------------------------------ |
| items      | TArray   | -       | Yes       | Array to iterate (use data pill with array type) |
| $id        | string   | -       | Yes       | Unique identifier (Now.ID['value'])              |
| annotation | string   | -       | No        | Description/comment                              |
| body       | function | -       | No        | Callback receiving each item                     |

**Supported Data Pill Types:**

- `'array.object'` - For lookUpRecords results and object arrays
- `'array.string'` - For string arrays (no item parameter)

**Example:**

```typescript
// Lookup records
const results = wfa.action(
  action.core.lookUpRecords,
  { $id: Now.ID["find_incidents"] },
  {
    table: "incident",
    conditions: "active=true^priority=1",
    max_results: 50
  }
);

// Iterate over results
wfa.flowLogic.forEach(
  wfa.dataPill(results.Records, "array.object"),
  { $id: Now.ID["process_incidents"], annotation: "Update each incident" },
  incident => {
    wfa.action(
      action.core.updateRecord,
      { $id: Now.ID["update_incident"] },
      {
        table_name: "incident",
        record: wfa.dataPill(incident.sys_id, "reference"),
        values: TemplateValue({ state: 2 })
      }
    );
  }
);
```

---

## Loop Control

### wfa.flowLogic.exitLoop

Immediately exits the enclosing forEach loop.

**Signature:**

```typescript
wfa.flowLogic.exitLoop(
  params: {
    $id: string,
    annotation?: string
  }
)
```

**Parameters:**

| Parameter  | Type   | Default | Mandatory | Description                         |
| ---------- | ------ | ------- | --------- | ----------------------------------- |
| $id        | string | -       | Yes       | Unique identifier (Now.ID['value']) |
| annotation | string | -       | No        | Description/comment                 |

**Example:**

```typescript
wfa.flowLogic.forEach(
  wfa.dataPill(results.Records, "array.object"),
  { $id: Now.ID["search_loop"] },
  record => {
    // Check if we found the target record
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_match"],
        condition: `${wfa.dataPill(record.number, "string")}=INC0001234`
      },
      () => {
        // Found the target - exit loop
        wfa.flowLogic.exitLoop({ $id: Now.ID["exit_loop"] });
      }
    );
  }
);
```

---

### wfa.flowLogic.skipIteration

Skips the current iteration and continues with the next item.

**Signature:**

```typescript
wfa.flowLogic.skipIteration(
  params: {
    $id: string,
    annotation?: string
  }
)
```

**Parameters:**

| Parameter  | Type   | Default | Mandatory | Description                         |
| ---------- | ------ | ------- | --------- | ----------------------------------- |
| $id        | string | -       | Yes       | Unique identifier (Now.ID['value']) |
| annotation | string | -       | No        | Description/comment                 |

**Example:**

```typescript
wfa.flowLogic.forEach(
  wfa.dataPill(results.Records, 'array.object'),
  { $id: Now.ID['filter_loop'] },
  (record) => {
    // Skip if assigned
    wfa.flowLogic.if(
      {
        $id: Now.ID['check_assigned'],
        condition: `${wfa.dataPill(record.assigned_to, 'string')}ISNOTEMPTY`
      },
      () => {
        wfa.flowLogic.skipIteration({ $id: Now.ID['skip'] });
      }
    );

    // Process unassigned records
    wfa.action(action.core.updateRecord, { $id: Now.ID['assign'] }, { ... });
  }
);
```

---

## Flow Control

### wfa.flowLogic.endFlow

Immediately terminates flow execution.

**Signature:**

```typescript
wfa.flowLogic.endFlow(
  params: {
    $id: string,
    annotation?: string
  }
)
```

**Parameters:**

| Parameter  | Type   | Default | Mandatory | Description                         |
| ---------- | ------ | ------- | --------- | ----------------------------------- |
| $id        | string | -       | Yes       | Unique identifier (Now.ID['value']) |
| annotation | string | -       | No        | Description/comment                 |

**Example:**

```typescript
// Check for error condition
wfa.flowLogic.if(
  {
    $id: Now.ID["check_error"],
    condition: `${wfa.dataPill(result.status, "integer")}=1`
  },
  () => {
    // Log error
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_error"] },
      {
        log_level: "error",
        log_message: "Critical error - terminating flow"
      }
    );

    // Terminate flow
    wfa.flowLogic.endFlow({ $id: Now.ID["end_flow"] });
  }
);
```

## ⚠️ CRITICAL: JavaScript Functions NOT Supported in if/elseIf/else

**Flow logic conditions (if, elseIf, else) do NOT support JavaScript functions like `javascript:gs.daysAgoStart(30)` or `javascript:gs.beginningOfThisWeek()`.**

Only use:

- Data pill comparisons with static values
- Data pills from trigger outputs
- Data pills from action outputs
- Encoded query operators (=, !=, <, >, ISEMPTY, etc.)

```typescript
// ❌ WRONG - JavaScript function not supported in if
wfa.flowLogic.if({
  condition: `${wfa.dataPill("trigger.record.sys_updated_on")}<javascript:gs.daysAgoStart(30)`
}, () => { ... });

// ❌ WRONG - JavaScript function not supported in elseIf
wfa.flowLogic.if({ condition: "..." }, () => { ... })
wfa.flowLogic.elseIf({
    condition: `${wfa.dataPill("trigger.record.due_date")}<javascript:gs.beginningOfThisWeek()`
  }, () => { ... });

// ✅ CORRECT - Data pill comparison in if/elseIf
wfa.flowLogic.if({
  condition: `${wfa.dataPill("trigger.record.priority")}=1`
}, () => { ... })
wfa.flowLogic.elseIf({
    condition: `${wfa.dataPill("trigger.record.priority")}=2`
  }, () => { ... });

// ✅ CORRECT - Comparing data pills from action outputs
const lookup = wfa.action(action.core.lookUpRecords, ...);
wfa.flowLogic.if({
  condition: `${wfa.dataPill(lookup.Count, "integer")}>0`
}, () => { ... });
```

**Note:** JavaScript functions work in table action conditions (lookUpRecords, updateMultipleRecords), but NOT in flow logic conditions (if/elseIf/else).

---

## Condition Syntax Reference

### Comparison Operators

| Operator | Description      | Example       |
| -------- | ---------------- | ------------- |
| `=`      | Equals           | `priority=1`  |
| `!=`     | Not equals       | `state!=6`    |
| `<`      | Less than        | `priority<3`  |
| `<=`     | Less or equal    | `priority<=2` |
| `>`      | Greater than     | `priority>3`  |
| `>=`     | Greater or equal | `priority>=2` |

### Empty/Not Empty Operators

| Operator     | Description     | Example                      |
| ------------ | --------------- | ---------------------------- |
| `ISEMPTY`    | Field is empty  | `assigned_toISEMPTY`         |
| `ISNOTEMPTY` | Field has value | `assignment_groupISNOTEMPTY` |

### List Operators

| Operator | Description | Example          |
| -------- | ----------- | ---------------- |
| `IN`     | In list     | `stateIN1,2,3`   |
| `NOT IN` | Not in list | `stateNOT IN6,7` |

### String Operators

| Operator     | Description | Example                           |
| ------------ | ----------- | --------------------------------- |
| `STARTSWITH` | Starts with | `numberSTARTSWITHINC`             |
| `ENDSWITH`   | Ends with   | `emailENDSWITH@company.com`       |
| `CONTAINS`   | Contains    | `short_descriptionCONTAINSfailed` |

### Logical Operators

| Operator      | Description          | Example                   |
| ------------- | -------------------- | ------------------------- |
| `^` or `^AND` | Logical AND          | `priority=1^active=true`  |
| `^OR`         | Logical OR           | `priority=1^ORpriority=2` |
| `^NQ`         | New query (OR group) | `priority=1^NQstate=2`    |

---

## Using Data Pills in Conditions

**Template Literal Pattern:**

```typescript
// ✅ CORRECT - Use data pills directly in template literal
condition: `${wfa.dataPill(_params.trigger.current.priority, "string")}=1`;
```

**Comparison Between Data Pills:**

```typescript
// ✅ CORRECT - Use data pills directly
condition: `${wfa.dataPill(_params.trigger.current.assigned_to, "string")}=${wfa.dataPill(_params.trigger.sys_updated_by, "string")}`;
```

**Complex Conditions:**

```typescript
// ✅ CORRECT - Multiple data pills in one condition
condition: `${wfa.dataPill(_params.trigger.current.priority, "string")}=1^${wfa.dataPill(_params.trigger.current.active, "string")}=true`;
```

---

## Complete Example

```typescript
import { Flow, wfa, trigger, action } from "@servicenow/sdk/automation";

Flow(
  {
    $id: Now.ID["process_high_priority_incidents"],
    name: "Process High Priority Incidents"
  },

  wfa.trigger(
    trigger.record.created,
    { $id: Now.ID["incident_created"] },
    {
      table: "incident",
      condition: "priority=1^active=true",
      run_flow_in: "background"
    }
  ),

  _params => {
    // Lookup related problems
    const problems = wfa.action(
      action.core.lookUpRecords,
      { $id: Now.ID["find_problems"] },
      {
        table: "problem",
        conditions: "active=true",
        max_results: 50
      }
    );

    // Check if any problems found
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_problems_found"],
        condition: `${wfa.dataPill(problems.Count, "integer")}>0`
      },
      () => {
        // Process each problem
        wfa.flowLogic.forEach(
          wfa.dataPill(problems.Records, "array.object"),
          { $id: Now.ID["process_problems"] },
          problem => {
            // Skip resolved problems
            wfa.flowLogic.if(
              {
                $id: Now.ID["check_resolved"],
                condition: `${wfa.dataPill(problem.state, "string")}=6`
              },
              () => {
                wfa.flowLogic.skipIteration({ $id: Now.ID["skip_resolved"] });
              }
            );

            // Link problem to incident
            wfa.action(
              action.core.updateRecord,
              { $id: Now.ID["link_problem"] },
              {
                table_name: "incident",
                record: wfa.dataPill(_params.trigger.current, "reference"),
                values: TemplateValue({
                  problem_id: wfa.dataPill(problem.sys_id, "reference")
                })
              }
            );

            // Exit loop after first match
            wfa.flowLogic.exitLoop({ $id: Now.ID["exit_loop"] });
          }
        );
      }
    );

    // If no problems, escalate
    wfa.flowLogic.else({ $id: Now.ID["no_problems_found"] }, () => {
      wfa.action(
        action.core.updateRecord,
        { $id: Now.ID["escalate"] },
        {
          table_name: "incident",
          record: wfa.dataPill(_params.trigger.current, "reference"),
          values: TemplateValue({ state: 3 })
        }
      );
    });

    // Log completion
    wfa.action(
      action.core.log,
      { $id: Now.ID["log_completion"] },
      {
        log_level: "info",
        log_message: "Flow completed successfully"
      }
    );
  }
);
```

---

## Advanced Patterns

### Nested If Statements

If statements can be nested within other if blocks for complex conditional logic:

```typescript
wfa.flowLogic.if(
  {
    $id: Now.ID["check_active"],
    condition: `${wfa.dataPill(_params.trigger.current.active, "boolean")}=true`,
    annotation: "Check if record is active"
  },
  () => {
    // Nested if within outer if
    wfa.flowLogic.if(
      {
        $id: Now.ID["check_priority"],
        condition: `${wfa.dataPill(_params.trigger.current.priority, "string")}=1`,
        annotation: "Check if priority is critical"
      },
      () => {
        wfa.action(
          action.core.log,
          { $id: Now.ID["log"] },
          {
            log_level: "info",
            log_message: "Active and critical priority incident detected"
          }
        );
      }
    );
  }
);
```

### Nested ForEach Loops

ForEach loops can be nested to process multi-dimensional data:

```typescript
// Outer loop: Process each incident
wfa.flowLogic.forEach(
  wfa.dataPill(incidents.Records, "array.object"),
  { $id: Now.ID["process_incidents"], annotation: "Process each incident" },
  incident => {
    // Inner loop: Process related problems for each incident
    wfa.flowLogic.forEach(
      wfa.dataPill(incident.related_problems, "array.object"),
      {
        $id: Now.ID["process_problems"],
        annotation: "Process related problems"
      },
      problem => {
        wfa.action(
          action.core.log,
          { $id: Now.ID["log_problem"] },
          {
            log_level: "info",
            log_message: `Processing problem ${wfa.dataPill(problem.number, "string")} for incident ${wfa.dataPill(incident.number, "string")}`
          }
        );
      }
    );
  }
);
```

---
