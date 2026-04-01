# Service Catalog Trigger

Triggers when a Service Catalog request item workflow needs to be processed. Used for automating fulfillment, approval, and notification workflows for catalog requests.

## Table of Contents

- [Best Practices](#best-practices)
- [Common Use Cases](#common-use-cases)
- [Important Notes](#important-notes)
- [Performance Considerations](#performance-considerations)
- [Next Steps](#next-steps)

## Best Practices

1. **Use Background Execution for Most Cases:** Set `run_flow_in: 'background'` for catalog processing workflows
   - Prevents performance impact on user request submission
   - Allows asynchronous processing of fulfillment tasks
   - Default and recommended for most catalog workflows

2. **Use Foreground Execution Sparingly:** Only use `run_flow_in: 'foreground'` when immediate synchronous processing is required
   - Blocks user until flow completes
   - Use for critical validations or instant responses
   - Can impact user experience if flow takes time

3. **Access Request Item Details:** Use trigger outputs to access catalog request information
   - `_params.trigger.request_item` - Reference to the sc_req_item record
   - `_params.trigger.table_name` - Table name (typically "sc_req_item")
   - `_params.trigger.run_start_date_time` - When the trigger fired

4. **Dot-Walk to Related Records:** Access related catalog information via relationships
   - `_params.trigger.request_item.request` - Parent request (sc_request)
   - `_params.trigger.request_item.cat_item` - Catalog item definition
   - `_params.trigger.request_item.requested_for` - User the item is for
   - `_params.trigger.request_item.opened_by` - User who submitted the request

5. **Check Request Item State:** Verify request item is in expected state before processing
   - Request items can be cancelled or modified after trigger fires
   - Check `_params.trigger.request_item.state` and `_params.trigger.request_item.active`
   - Handle different states appropriately (pending, approved, cancelled)

6. **Retrieve Catalog Variables:** Use `getCatalogVariables` action to access user selections
   - Catalog variables contain user input (size, color, configuration, etc.)
   - Define CatalogItem with variables for type safety
   - Access variable values via data pills for conditional logic
   - Drive workflow routing based on variable selections

7. **Use for Approval Workflows:** Common pattern for catalog items requiring approval
   - Trigger when request item is created
   - Look up approver based on item/user/department
   - Use askForApproval action to create approval records
   - Handle approval/rejection outcomes

8. **Use for Fulfillment Automation:** Automate catalog item fulfillment tasks
   - Create fulfillment tasks using createCatalogTask action
   - Update inventory or provision resources
   - Send notifications to fulfillment teams
   - Update request item status upon completion

9. **Avoid Recursive Triggers:** Be careful when updating request items in the flow
   - Updates to request items could potentially re-trigger the flow
   - Use specific conditions or flags to prevent infinite loops
   - Consider using separate workflow stages or states

10. **Log Catalog Events:** Track catalog processing for audit and troubleshooting
    - Log request item number, catalog item, requested user
    - Include timestamps and processing steps
    - Helps diagnose fulfillment delays or failures

## Common Use Cases

- **Approval Workflows:** Route high-value catalog items for manager approval before fulfillment
- **Automated Fulfillment:** Create tasks, provision resources, or update inventory automatically
- **Multi-Stage Workflows:** Orchestrate complex fulfillment processes with multiple steps
- **Notification Workflows:** Notify requesters, approvers, and fulfillment teams at key stages
- **Integration Workflows:** Integrate with external systems for ordering or provisioning
- **Variable-Based Routing:** Route requests to different teams based on catalog variable selections

## Important Notes

- **Trigger Timing:** Fires when a service catalog request item workflow is initiated
  - Typically fires when request item is created or submitted
  - Can be configured to fire at different stages of request lifecycle
  - Background execution is default and recommended

- **Request Item Access:** Trigger provides direct access to request item record
  - `_params.trigger.request_item` is a reference to sc_req_item record
  - Use dot-walking to access all request item fields
  - Example: `_params.trigger.request_item.short_description`

- **Execution Context Options:**
  - `any` - System chooses context (default behavior)
  - `background` - Asynchronous execution (recommended for most workflows)
  - `foreground` - Synchronous execution (blocks user, use sparingly)

- **Table Name Output:** Always returns "sc_req_item" as the table name
  - Useful for generic actions that need table context
  - Can be passed to actions that require table_name parameter

- **Multiple Triggers per Item:** A single request item can trigger multiple flows
  - Use specific conditions to target appropriate flows
  - Consider catalog item type, category, or price thresholds
  - Prevent duplicate processing with proper flow conditions

- **State Considerations:** Request items have multiple states
  - Common states: pending, approved, rejected, cancelled, closed
  - Check state before performing actions
  - Handle state transitions appropriately

- **Relationship to Service Catalog Actions:** Commonly used with catalog-specific actions
  - `getCatalogVariables` - Retrieve catalog variable values from request items
  - `createCatalogTask` - Create fulfillment tasks for request items
  - `submitCatalogItemRequest` - Submit new catalog requests (for related items)

## Performance Considerations

- **Use Background Execution:** Default to background execution unless synchronous processing required
- **Filter by Catalog Item:** Use conditions to target specific catalog items if possible
- **Minimize Lookups:** Use trigger data and dot-walking instead of additional lookups
- **Batch Operations:** Group multiple operations together when possible
- **Monitor Flow Performance:** Track execution times for catalog workflows
- **Avoid Heavy Processing in Foreground:** Never use foreground execution for long-running operations

## Next Steps

For Fluent API signatures, parameters, output fields, and coding examples, use `get_knowledge_source` tool to get the **WFA_FLOW_TRIGGER_APPLICATION** knowledge source.
