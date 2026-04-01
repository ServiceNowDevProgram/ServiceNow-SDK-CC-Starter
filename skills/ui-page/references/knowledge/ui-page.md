# UI_PAGE

# ServiceNow UI Pages - Templates and Examples

This knowledge source provides working code templates for ServiceNow UI Pages. For guidelines, requirements, and patterns, activate the `ui-page` skill first.

## Minimal Starter Template

**When to use:** Simple lists, basic forms, or single-view pages. Start here for most requests.

### File Structure

```
src/
  client/
    tsconfig.json
    index.html
    main.tsx
    app.tsx
    utils/
      fields.ts
  fluent/
    ui-pages/
      page.now.ts
```

## tsconfig.json Contents

```json
{
  "compilerOptions": {
    "moduleResolution": "bundler",
    "module": "es2022",
    "target": "es2022",
    "lib": ["ES2022", "DOM"],
    "jsx": "preserve"
  }
}
```

### 1. Fields Utility (ALWAYS CREATE FIRST)

```typescript
// src/client/utils/fields.ts
export const display = field => field?.display_value || "";
export const value = field => field?.value || "";
```

### 2. UI Page Definition

```typescript
// src/fluent/ui-pages/page.now.ts
import "@servicenow/sdk/global";
import { UiPage } from "@servicenow/sdk/core";
import page from "../../client/index.html";

export const my_page = UiPage({
  $id: Now.ID["my-page"],
  endpoint: "x_app_page.do",
  html: page, // CRITICAL: must import the content to use the output of the build system
  direct: true // CRITICAL: Must be true
});
```

#### 3. HTML Entry (with Array.from Polyfill)

```html
<!-- src/client/index.html -->
<html class="-polaris">
  <head>
    <title>My Page</title>
    <sdk:now-ux-globals></sdk:now-ux-globals>
    <!-- Array.from polyfill to fix prototype.js breaking iterables (Set, Map, etc.) -->
    <!-- MUST be inline script BEFORE module scripts - ESM imports are hoisted so external polyfill files won't work -->
    <script type="text/javascript">
      //<![CDATA[
      (function () {
        var testWorks = (function () {
          try {
            var result = Array.from(new Set([1, 2]));
            return (
              Array.isArray(result) && result.length === 2 && result[0] === 1
            );
          } catch (e) {
            return false;
          }
        })();
        if (testWorks) return;
        var originalArrayFrom = Array.from;
        function specArrayFrom(arrayLike, mapFn, thisArg) {
          if (arrayLike == null)
            throw new TypeError(
              "Array.from requires an array-like or iterable object"
            );
          var C = this;
          if (typeof C !== "function" || C === Window || C === Object) {
            C = Array;
          }
          var mapping = typeof mapFn === "function";
          var iterFn = arrayLike[Symbol.iterator];
          if (typeof iterFn === "function") {
            var result = [];
            var i = 0;
            var iterator = iterFn.call(arrayLike);
            var step;
            while (!(step = iterator.next()).done) {
              result[i] = mapping
                ? mapFn.call(thisArg, step.value, i)
                : step.value;
              i++;
            }
            result.length = i;
            return result;
          }
          var items = Object(arrayLike);
          var len = Math.min(
            Math.max(Number(items.length) || 0, 0),
            Number.MAX_SAFE_INTEGER
          );
          var result = new C(len);
          for (var k = 0; k < len; k++) {
            result[k] = mapping ? mapFn.call(thisArg, items[k], k) : items[k];
          }
          result.length = len;
          return result;
        }
        Array.from = function (arrayLike, mapFn, thisArg) {
          if (
            arrayLike != null &&
            typeof arrayLike[Symbol.iterator] === "function"
          ) {
            try {
              return specArrayFrom.call(this, arrayLike, mapFn, thisArg);
            } catch (e) {
              console.error("Array.from failed with error:", e);
              return originalArrayFrom.call(this, arrayLike, mapFn, thisArg);
            }
          }
          return originalArrayFrom.call(this, arrayLike, mapFn, thisArg);
        };
      })();

      if (window.Element && Element.Methods) {
        Element.Methods.remove = function (element) {
          element = $(element);
          if (element.parentNode) {
            element.parentNode.removeChild(element);
          }
          return element;
        };
        // Re-extend to apply to all elements
        Element.addMethods();
      }

      //]]>
    </script>
    <script
      src="main.tsx?uxpcb=$[UxFrameworkScriptables.getFlushTimestamp()]"
      type="module"
    ></script>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

**IMPORTANT NOTES:**

- The `uxpcb` parameter is required to ensure that stale UI Page contents are not mistakenly cached.
- The Array.from polyfill MUST be an inline `<script>` tag (not `type="module"`) placed BEFORE the module script. This is because ESM imports are hoisted and execute before any inline code in the module, so importing a polyfill file won't work.
- The `//<![CDATA[` and `//]]>` wrappers are required to prevent Jelly from parsing JavaScript operators like `<` and `&&` as XML syntax.

### 4. React Bootstrap

```tsx
// src/client/main.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./app";

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```

### 5. Main Component

```tsx
// src/client/app.tsx
import React, { useEffect, useState } from "react";
import { display, value } from "./utils/fields";

export default function App() {
  const [items, setItems] = useState([]);

  useEffect(() => {
    fetch(
      "/api/now/table/incident?sysparm_display_value=all&sysparm_limit=10",
      {
        headers: {
          Accept: "application/json",
          "X-UserToken": window.g_ck // CRITICAL: Authentication
        }
      }
    )
      .then(res => res.json())
      .then(data => setItems(data.result || []));
  }, []);

  return (
    <div>
      <h1>Incidents</h1>
      <ul>
        {items.map(item => (
          <li key={value(item.sys_id)}>{display(item.short_description)}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Full CRUD Implementation Example

**When to use:** Full CRUD interfaces, forms with validation, or when you need service layers and multiple components.

### Project Structure

```
src/
  client/
    tsconfig.json
    index.html
    main.tsx
    app.tsx
    app.css
    utils/
      fields.ts
    components/
      TodoList.tsx
      TodoList.css
      TodoItem.tsx
      TodoItem.css
      TodoForm.tsx
      TodoForm.css
    services/
      TodoService.ts
  fluent/
    ui-pages/
      todo.now.ts
```

### UI Page Definition

```typescript
// src/fluent/ui-pages/todo.now.ts
import "@servicenow/sdk/global";
import { UiPage } from "@servicenow/sdk/core";
import todoPage from "../../client/index.html";

export const todo_page = UiPage({
  $id: Now.ID["todo-page"],
  endpoint: "x_app_todo.do",
  html: todoPage,
  direct: true
});
```

### HTML Container

```html
<!-- src/client/index.html -->
<html>
  <head>
    <title>Todo App</title>
    <sdk:now-ux-globals></sdk:now-ux-globals>
    <script
      src="main.tsx?uxpcb=$[UxFrameworkScriptables.getFlushTimestamp()]"
      type="module"
    ></script>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

### React Bootstrap

```tsx
// src/client/main.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./app";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

### Main Component with CSS

```tsx
// src/client/app.tsx
import React, { useEffect, useState, useMemo } from "react";
import { TodoService } from "./services/TodoService";
import TodoForm from "./components/TodoForm";
import TodoList from "./components/TodoList";
import "./app.css";

export default function App() {
  const service = useMemo(() => new TodoService(), []);
  const [todos, setTodos] = useState([]);

  useEffect(() => {
    service.list().then(setTodos);
  }, [service]);

  const refreshTodos = () => service.list().then(setTodos);

  return (
    <div className="todo-app">
      <h1>Todo Manager</h1>
      <TodoForm
        service={service}
        onAdd={refreshTodos}
      />
      <TodoList
        todos={todos}
        service={service}
        onChange={refreshTodos}
      />
    </div>
  );
}
```

```css
/* src/client/app.css */
.todo-app {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}
```

### Service Layer

```typescript
// src/client/services/TodoService.ts
export class TodoService {
  constructor() {
    this.tableName = "x_app_todo";
  }

  async list() {
    const response = await fetch(
      `/api/now/table/${this.tableName}?sysparm_display_value=all`,
      {
        headers: {
          Accept: "application/json",
          "X-UserToken": window.g_ck
        }
      }
    );
    const { result } = await response.json();
    return result || [];
  }

  async create(data) {
    const response = await fetch(
      `/api/now/table/${this.tableName}?sysparm_display_value=all`,
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Accept: "application/json",
          "X-UserToken": window.g_ck
        },
        body: JSON.stringify(data)
      }
    );
    return response.json();
  }

  async update(sysId, data) {
    const response = await fetch(
      `/api/now/table/${this.tableName}/${sysId}?sysparm_display_value=all`,
      {
        method: "PATCH",
        headers: {
          "Content-Type": "application/json",
          Accept: "application/json",
          "X-UserToken": window.g_ck
        },
        body: JSON.stringify(data)
      }
    );
    return response.json();
  }

  async delete(sysId) {
    const response = await fetch(`/api/now/table/${this.tableName}/${sysId}`, {
      method: "DELETE",
      headers: {
        Accept: "application/json",
        "X-UserToken": window.g_ck
      }
    });
    return response.ok;
  }
}
```

### Components

```tsx
// src/client/components/TodoForm.tsx
import React from "react";
import "./TodoForm.css";

export default function TodoForm({ service, onAdd }) {
  async function handleSubmit(e) {
    e.preventDefault();
    await service.create({ short_description: e.target.text.value });
    e.target.reset();
    onAdd();
  }

  return (
    <form
      className="todo-form"
      onSubmit={handleSubmit}
    >
      <input
        name="text"
        className="todo-input"
        placeholder="What needs to be done?"
        required
      />
      <button
        type="submit"
        className="todo-submit"
      >
        Add
      </button>
    </form>
  );
}
```

```css
/* src/client/components/TodoForm.css */
.todo-form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.todo-input {
  flex: 1;
  padding: 8px;
}

.todo-submit {
  padding: 8px 16px;
}
```

```tsx
// src/client/components/TodoList.tsx
import React from "react";
import { value } from "../utils/fields";
import TodoItem from "./TodoItem";
import "./TodoList.css";

export default function TodoList({ todos, service, onChange }) {
  return (
    <ul className="todo-list">
      {todos.map(todo => (
        <TodoItem
          key={value(todo.sys_id)}
          todo={todo}
          service={service}
          onChange={onChange}
        />
      ))}
    </ul>
  );
}
```

```css
/* src/client/components/TodoList.css */
.todo-list {
  list-style: none;
  padding: 0;
  margin: 20px 0;
}
```

```tsx
// src/client/components/TodoItem.tsx
import React from "react";
import { display, value } from "../utils/fields";
import "./TodoItem.css";

export default function TodoItem({ todo, service, onChange }) {
  const isDone = display(todo.state) === "closed";

  async function toggle() {
    await service.update(value(todo.sys_id), {
      state: isDone ? "open" : "closed"
    });
    onChange();
  }

  async function remove() {
    await service.delete(value(todo.sys_id));
    onChange();
  }

  return (
    <li className="todo-item">
      <input
        type="checkbox"
        checked={isDone}
        onChange={toggle}
      />
      <span className={isDone ? "todo-done" : ""}>
        {display(todo.short_description)}
      </span>
      {display(todo.assigned_to) && (
        <span className="todo-assigned"> - {display(todo.assigned_to)}</span>
      )}
      <button
        className="todo-delete"
        onClick={remove}
      >
        Delete
      </button>
    </li>
  );
}
```

```css
/* src/client/components/TodoItem.css */
.todo-item {
  display: flex;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #eee;
}

.todo-done {
  text-decoration: line-through;
  opacity: 0.6;
}

.todo-assigned {
  color: #666;
  margin-left: 5px;
}

.todo-delete {
  margin-left: auto;
}
```

### Table Configuration

```typescript
Table({
  name: "x_app_todo",
  label: "Todo",
  web_service_access: true, // Required for API access
  accessible_from: "public",
  actions: ["create", "read", "update", "delete"]
});
```

---

## Common User Requests Mapping

| User Request                           | Agent Implementation                               |
| -------------------------------------- | -------------------------------------------------- |
| "Create a ticket management interface" | React with state-based routing and Table API       |
| "Build a form to submit requests"      | React form component with POST to Table API        |
| "Dashboard showing metrics"            | Multiple React components with aggregated queries  |
| "Add webpack configuration"            | "ServiceNow IDE handles build automatically"       |
| "Configure TypeScript"                 | "IDE handles TypeScript compilation automatically" |
| "Multi-step wizard"                    | State-based SPA with shared context                |
| "Tab interface"                        | Switch statement with currentView state            |
| "Add React Router"                     | "Use built-in state-based routing instead"         |
| "Real-time updates"                    | useEffect with polling interval                    |
| "Export to Excel"                      | Table API query with client-side formatting        |

---

## Quick Reference

### ALWAYS

- Use React 18.2.0 for ALL UI Pages
- Use `sysparm_display_value=all` in Table API calls
- Create fields.ts utility first
- Include X-UserToken header
- Keep files under 100 lines
- Use state-based routing for SPAs (no dependencies)

### NEVER

- Create build configurations
- Add router libraries (use state-based routing)
- Use vanilla JavaScript for UIs
- Use GlideAjax or g_form
- Add build tools to dependencies
- Use CSS Modules
- Create files over 100 lines without splitting
