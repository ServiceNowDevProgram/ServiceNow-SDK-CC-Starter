# Single Page Application (SPA) Patterns

## Contents

- [Default Navigation Approach](#default-navigation-approach)
- [When to Use SPA Architecture](#when-to-use-spa-architecture)
- [URL-Based Routing Pattern](#url-based-routing-pattern-default)
- [Shared State Management with Context](#shared-state-management-with-context)
- [Using the Context](#using-the-context)
- [SPA Best Practices](#spa-best-practices)

## Default Navigation Approach

**IMPORTANT: Full SPA with URL-based routing is the DEFAULT navigation pattern for ALL UI Pages unless the user explicitly specifies otherwise.**

Every UI Page application should:

- Use URLSearchParams for navigation (`?view=list`, `?view=details&id=123`, `?tab=overview`)
- Implement view switching based on URL parameters
- Support browser back/forward buttons via `popstate` event
- Follow patterns in `navigation.md` for context-aware routing (Polaris iframe vs standalone)

## When to Use SPA Architecture

SPA architecture is the DEFAULT for:

- Any application with multiple views (list, detail, edit, create)
- Multi-step forms or wizards
- Tab-based interfaces
- Complex state management across views
- Real-time data updates
- Dashboard with multiple sections
- ANY UI Page unless user explicitly requests a single-view page

## URL-Based Routing Pattern (DEFAULT)

```tsx
// src/client/app.tsx
import React, { useState, useEffect } from "react";
import { Button } from "@servicenow/react-components/Button";
import Dashboard from "./components/Dashboard.tsx";
import TicketList from "./components/TicketList.tsx";
import TicketDetail from "./components/TicketDetail.tsx";
import Settings from "./components/Settings.tsx";

export default function App() {
  const [currentView, setCurrentView] = useState("dashboard");
  const [viewParams, setViewParams] = useState(null);

  // Handle URLSearchParams-based routing
  useEffect(() => {
    const handleNavigation = () => {
      const params = new URLSearchParams(window.location.search);
      setCurrentView(params.get("view") || "dashboard");
      setViewParams(params.get("id"));
    };

    window.addEventListener("popstate", handleNavigation);
    handleNavigation(); // Initial load

    return () => window.removeEventListener("popstate", handleNavigation);
  }, []);

  // IMPORTANT: Use the navigateToView() function from navigation.md instead of this simplified version
  // This example shows the pattern, but production code MUST use navigation.md for Polaris iframe support
  const navigate = (view, id) => {
    const params = new URLSearchParams({ view });
    if (id) params.set("id", id);
    window.history.pushState({ view, id }, "", `?${params}`);
    setCurrentView(view);
    setViewParams(id);
  };

  // Navigation menu — check package_docs for correct Button click event prop
  const renderNav = () => (
    <nav>
      <Button label="Dashboard" /* click event from docs */ />
      <Button label="Tickets" /* click event from docs */ />
      <Button label="Settings" /* click event from docs */ />
    </nav>
  );

  // Route to appropriate component
  const renderView = () => {
    switch (currentView) {
      case "tickets":
        return <TicketList onSelectTicket={id => navigate("ticket", id)} />;
      case "ticket":
        return (
          <TicketDetail
            ticketId={viewParams}
            onBack={() => navigate("tickets")}
          />
        );
      case "settings":
        return <Settings />;
      default:
        return <Dashboard />;
    }
  };

  return (
    <div>
      {renderNav()}
      {renderView()}
    </div>
  );
}
```

## Shared State Management with Context

```tsx
// src/client/contexts/AppContext.tsx
import React, { createContext, useState, useContext } from "react";

const AppContext = createContext();

export function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [filters, setFilters] = useState({});
  const [currentView, setCurrentView] = useState("dashboard");

  // IMPORTANT: Use the navigateToView() function from navigation.md instead of this simplified version
  // This example shows the pattern, but production code MUST use navigation.md for Polaris iframe support
  const navigate = view => {
    setCurrentView(view);
    const params = new URLSearchParams({ view });
    window.history.pushState({ view }, "", `?${params}`);
  };

  return (
    <AppContext.Provider
      value={{
        user,
        setUser,
        filters,
        setFilters,
        currentView,
        navigate
      }}
    >
      {children}
    </AppContext.Provider>
  );
}

export const useApp = () => useContext(AppContext);
```

## Using the Context

```tsx
// src/client/main.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { AppProvider } from "./contexts/AppContext.tsx";
import App from "./app.tsx";

ReactDOM.createRoot(document.getElementById("root")).render(
  <AppProvider>
    <App />
  </AppProvider>
);
```

```tsx
// src/client/components/TicketList.tsx
import React from "react";
import { Button } from "@servicenow/react-components/Button";
import { useApp } from "../contexts/AppContext.tsx";

export default function TicketList() {
  const { filters, navigate } = useApp();

  // Use filters and navigate from context — check package_docs for correct Button click event prop
  return (
    <div>
      <Button label="View Ticket" /* click event from docs */ />
    </div>
  );
}
```

## SPA Best Practices

- **DEFAULT APPROACH:** Always implement full SPA with URL-based routing unless user explicitly specifies otherwise
- Keep route components under 80 lines
- Use URLSearchParams-based routing (no router library dependencies)
- Centralize API calls in a service layer
- Use React Context for shared state across views
- Maintain a single HTML entry point
- NEVER use hash routing (`#/route`) - ALWAYS use URLSearchParams (`?view=details`) for navigation (see `navigation.md`)
- Each logical view/tab MUST have its own URL parameter
- Support browser back/forward navigation with `popstate` event listener
