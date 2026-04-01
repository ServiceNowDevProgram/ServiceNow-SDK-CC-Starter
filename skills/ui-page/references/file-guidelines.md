# File Size Guidelines

## Optimal Ranges by File Type

### Components (50-80 lines ideal, 100 max)

- Simple display: 30-50 lines
- Forms: 50-80 lines
- Complex with state: 60-100 lines max
- Break at 100 lines into sub-components

### Service Modules (30-60 lines)

- API service: 40-60 lines (with error handling)
- Single responsibility services: 30-50 lines

### Hooks (20-50 lines)

- Simple hooks: 20-30 lines
- Complex hooks with cleanup: 40-50 lines

### Utility Functions (20-40 lines)

- Group related utilities together
- 3-5 related functions per file

### Main App Component (50-100 lines)

- Composition and routing logic
- OK to be larger as it's mostly TSX structure

## When to Split Files

Split when:

- File exceeds 100 lines
- Multiple unrelated responsibilities
- Component has 3+ useEffect hooks
- Service has 5+ API methods

## Component Structure Best Practices

- Separate components into individual files under `src/client/components/`
- Keep files under 100 lines
- Use `.tsx` extension for TSX files
- Modularize - don't define everything in one file
- Each component should have a single responsibility
- Minimize comments - only add them when absolutely necessary for complex logic

## Example File Organization

```
src/client/
  components/
    TodoList.tsx        # 40 lines - list display
    TodoItem.tsx        # 45 lines - single item with actions
    TodoForm.tsx        # 35 lines - input form
    Header.tsx          # 25 lines - navigation header
    LoadingSpinner.tsx  # 15 lines - reusable spinner
  services/
    TodoService.ts      # 55 lines - CRUD operations
    UserService.ts      # 40 lines - user operations
  hooks/
    useFetch.ts         # 30 lines - data fetching hook
    useDebounce.ts      # 20 lines - debounce utility
  utils/
    fields.ts           # 10 lines - field extraction
    formatters.ts       # 25 lines - date/number formatting
  contexts/
    AppContext.tsx      # 45 lines - global state
```
