# Catalog UI Policy Reference

For properties and templates, use `get_knowledge_source` with knowledge source `CATALOG_UI_POLICY`.

## Condition Syntax

Use object references in conditions (e.g., `${catalogItem.variables.priority}=high^EQ`).

## Policy with Client Scripts

Set `runScripts: true` and provide `executeIfTrue` / `executeIfFalse` scripts via `Now.include(...)`. Scripts must be wrapped in `function onCondition() {}`.

## Priority Rules

1. **Mandatory** has highest priority
2. If a variable is mandatory and has no value, readonly/hide actions **do not work**
3. If a variable set/container has a mandatory variable without value, the entire set **cannot be hidden**
4. "Clear value" action does not work on variable sets and containers

## Variable Type Limitations

| Policy Type | Not Applicable To                                        |
| ----------- | -------------------------------------------------------- |
| Mandatory   | Container Split, Container End, UI Macro, Label, UI Page |
| Read-only   | Container Split, Container End, UI Macro, Label, UI Page |
| Visibility  | Container Split, Container End                           |

## When to Use UI Policy vs Client Script

| Use Case                 | UI Policy     | Client Script |
| ------------------------ | ------------- | ------------- |
| Show/hide variables      | **Preferred** | Supported     |
| Make variables mandatory | **Preferred** | Supported     |
| Make variables read-only | **Preferred** | Supported     |
| Set variable values      | Supported     | Supported     |
| Complex validation       | Limited       | **Preferred** |
| Dynamic calculations     | Limited       | **Preferred** |
| API calls / async        | Not supported | Supported     |
| Form submission control  | Not supported | Supported     |

For complete examples and templates, use `get_knowledge_source` with knowledge source `CATALOG_UI_POLICY`.
