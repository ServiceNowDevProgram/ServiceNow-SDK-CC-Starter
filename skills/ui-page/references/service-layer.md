# Service Layer Pattern

## Contents

- [Purpose](#purpose)
- [Basic Service Class](#basic-service-class)
- [Service with Error Handling](#service-with-error-handling)
- [Using Services in Components](#using-services-in-components)
- [Key Points](#key-points)

## Purpose

Centralize all API calls in a service layer to:

- Keep components focused on UI logic
- Enable easy testing and mocking
- Standardize error handling
- Maintain consistent authentication

## Basic Service Class

```ts
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

## Service with Error Handling

```ts
// src/client/services/ApiService.ts
export class ApiService {
  constructor(tableName) {
    this.tableName = tableName;
    this.baseUrl = `/api/now/table/${tableName}`;
  }

  async request(url, options = {}) {
    try {
      const response = await fetch(url, {
        ...options,
        headers: {
          Accept: "application/json",
          "X-UserToken": window.g_ck,
          ...options.headers
        }
      });

      if (!response.ok) {
        // ServiceNow returns error details in JSON body
        const error = await response.json().catch(() => ({}));
        throw new Error(
          error.error?.message || `Request failed: ${response.status}`
        );
      }

      // DELETE returns no content
      if (response.status === 204) {
        return true;
      }

      return await response.json();
    } catch (error) {
      console.error("API Error:", error);
      throw error;
    }
  }

  async list(query = "") {
    const params = new URLSearchParams({
      sysparm_display_value: "all"
    });
    if (query) params.set("sysparm_query", query);

    const { result } = await this.request(`${this.baseUrl}?${params}`);
    return result || [];
  }

  async get(sysId) {
    const { result } = await this.request(
      `${this.baseUrl}/${sysId}?sysparm_display_value=all`
    );
    return result;
  }

  async create(data) {
    const { result } = await this.request(
      `${this.baseUrl}?sysparm_display_value=all`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data)
      }
    );
    return result;
  }

  async update(sysId, data) {
    const { result } = await this.request(
      `${this.baseUrl}/${sysId}?sysparm_display_value=all`,
      {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data)
      }
    );
    return result;
  }

  async delete(sysId) {
    return this.request(`${this.baseUrl}/${sysId}`, { method: "DELETE" });
  }
}
```

## Using Services in Components

```jsx
// src/client/app.tsx
import React, { useEffect, useState, useMemo } from "react";
import { Button } from "@servicenow/react-components/Button";
import { TodoService } from "./services/TodoService";
import { display, value } from "./utils/fields";

export default function App() {
  const service = useMemo(() => new TodoService(), []);
  const [todos, setTodos] = useState([]);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadTodos();
  }, []);

  async function loadTodos() {
    try {
      setLoading(true);
      setError(null);
      const data = await service.list();
      setTodos(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  async function handleCreate(text) {
    try {
      await service.create({ short_description: text });
      await loadTodos();
    } catch (err) {
      setError(err.message);
    }
  }

  async function handleDelete(sysId) {
    try {
      await service.delete(sysId);
      await loadTodos();
    } catch (err) {
      setError(err.message);
    }
  }

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h1>Todos</h1>
      <ul>
        {/* Note: .map() is acceptable here for custom non-record data managed via service layer */}
        {todos.map(todo => (
          <li key={value(todo.sys_id)}>
            {display(todo.short_description)}
            <Button
              label="Delete" /* check package_docs for correct click event prop */
            />
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Key Points

1. **Always include `X-UserToken: window.g_ck`** - Required for authentication
2. **Always use `sysparm_display_value=all`** - Returns both display and raw values
3. **Centralize error handling** - Parse JSON errors from ServiceNow responses
4. **Use `useMemo` for service instances** - Prevents recreation on re-renders
5. **Keep services under 60 lines** - Split into multiple services if needed
