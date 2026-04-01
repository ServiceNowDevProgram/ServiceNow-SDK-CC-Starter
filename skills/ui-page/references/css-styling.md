# CSS Styling Guidelines

## Contents

- [Supported CSS Patterns](#supported-css-patterns)
- [NOT Supported](#not-supported)
- [Best Practices](#best-practices)
  - [File Organization](#file-organization)
  - [Naming Conventions](#naming-conventions)
- [ServiceNow Theming Integration](#servicenow-theming-integration)
  - [Related Knowledge Sources](#related-knowledge-sources)
  - [Example Using Theme Variables](#example-using-theme-variables)

## Supported CSS Patterns

**Import CSS files directly in TSX/TS files using ESM syntax:**

```tsx
import "./filename.css";
import "./app.css";
import "./components/TodoItem.css";
```

## NOT Supported

- **CSS Modules**: `import styles from './file.module.css'` - NOT supported
- **@import statements**: Within CSS files - NOT supported
- **Link tags**: `<link rel="stylesheet" href="...">` in HTML - NOT supported
- **CSS-in-CSS imports**: Relative stylesheet references - NOT supported

## Best Practices

### File Organization

Place CSS files in `src/client` alongside their components. Each component can have its own CSS file. The build system automatically bundles all imported CSS.

```
src/client/
  app.tsx
  app.css
  components/
    TodoList.tsx
    TodoList.css
    TodoItem.tsx
    TodoItem.css
    TodoForm.tsx
    TodoForm.css
```

### Naming Conventions

Since CSS Modules aren't supported, use naming conventions like BEM to avoid conflicts:

```css
/* TodoItem.css */
.todo-item {
  display: flex;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #eee;
}

.todo-item__text {
  flex: 1;
}

.todo-item__text--done {
  text-decoration: line-through;
  opacity: 0.6;
}

.todo-item__delete {
  margin-left: auto;
}
```

```jsx
// TodoItem.tsx
import "./TodoItem.css";

export default function TodoItem({ todo, onDelete }) {
  return (
    <li className="todo-item">
      <span
        className={`todo-item__text ${todo.done ? "todo-item__text--done" : ""}`}
      >
        {todo.text}
      </span>
      <button
        className="todo-item__delete"
        onClick={onDelete}
      >
        Delete
      </button>
    </li>
  );
}
```

## ServiceNow Theming Integration

Use the `css_variable_explorer` tool to search for CSS variables and consult the UI_PAGE_THEMING knowledge sources for how to apply ServiceNow design system themes to your UI Page.

These CSS variables allow customer-authored themes to be applied to your UI Page that can affect the entire ServiceNow instance. This design system is referred to as "Horizon Design System".

### Related Knowledge Sources

Use `get_knowledge_source` tool with:

- **UI_PAGE_THEMING_FOUNDATIONS** - Color, typography, spacing variables
- **UI_PAGE_THEMING_LAYOUT** - Layout patterns and containers
- **UI_PAGE_THEMING_COMPONENTS** - UI components styling
- **UI_PAGE_THEMING_CONTROLS** - Form controls and inputs

### Example Using Theme Variables

```css
.my-card {
  background-color: var(--now-color-background-primary);
  border: 1px solid var(--now-color-border-primary);
  border-radius: var(--now-border-radius-md);
  padding: var(--now-spacing-lg);
  color: var(--now-color-text-primary);
}

.my-button {
  background-color: var(--now-color-interactive-primary);
  color: var(--now-color-text-inverse);
  padding: var(--now-spacing-sm) var(--now-spacing-md);
  border-radius: var(--now-border-radius-sm);
}

.my-button:hover {
  background-color: var(--now-color-interactive-primary-hover);
}
```
