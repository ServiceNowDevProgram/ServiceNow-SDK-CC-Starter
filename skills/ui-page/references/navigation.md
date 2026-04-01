# URL Generation and Navigation

Use context-aware navigation with URLSearchParams. Each view MUST have its own URL.

## Navigation Pattern

```typescript
function navigateToView(
  viewName: string,
  recordId?: string | null,
  { reload = false } = {}
) {
  // CRITICAL: Create separate URLs for each view using URLSearchParams
  const params = new URLSearchParams({ view: viewName });
  if (recordId) params.set("id", recordId);
  const path = `/x_app_page.do?${params}`;

  // CRITICAL: Generate title for both Polaris and standalone
  const title = viewName.charAt(0).toUpperCase() + viewName.slice(1);

  if (window.self !== window.top) {
    // Polaris: Update parent frame URL and title
    window.CustomEvent.fireTop("magellanNavigator.permalink.set", {
      relativePath: path,
      title: title // CRITICAL: Include title for Polaris
    });
    // CRITICAL: Polaris requires reload for new record creation
    if (reload) window.location.reload();
  } else {
    // Standalone: Update current page URL and title
    window.history.pushState({ viewName, recordId }, "", path);
    document.title = title; // CRITICAL: Set document title for standalone
  }
}

// Usage - each creates a separate URL:
navigateToView("list"); // /x_app_page.do?view=list
navigateToView("detail", "abc123"); // /x_app_page.do?view=detail&id=abc123
navigateToView("create", null, { reload: true }); // /x_app_page.do?view=create (reloads in Polaris)
```

After fetching a record, update the page title with the record's display name:

```typescript
function updatePageTitle(label: string) {
  if (window.self !== window.top) {
    window.CustomEvent.fireTop("magellanNavigator.permalink.set", {
      relativePath: window.location.pathname + window.location.search,
      title: label
    });
  } else {
    document.title = label;
  }
}

// Call after fetching record data in the detail/edit component:
// e.g., updatePageTitle(record.name || record.short_description);
```

## Key Points

- Each view needs its own URL with URLSearchParams
- Check `window.self !== window.top` for iframe context
- Use `window.CustomEvent.fireTop` in Polaris iframe
- Use `history.pushState()` for standalone pages
- NEVER use hash-based routing (`#/path`)
- ALWAYS use query strings (`?view=details`)
