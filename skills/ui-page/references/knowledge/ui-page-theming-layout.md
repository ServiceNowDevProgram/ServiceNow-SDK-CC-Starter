# UI_PAGE_THEMING_LAYOUT

# **ServiceNow Horizon UI Generation Specification - LAYOUT**

**Document:** UI_PAGE_THEMING_LAYOUT  
**Related Documents:**

- `UI_PAGE_THEMING_FOUNDATIONS` - Token selection rules, alias patterns, core principles
- `UI_PAGE_THEMING_CONTROLS` - Button, input, form control implementations
- `UI_PAGE_THEMING_COMPONENTS` - Navigation, feedback, indicator components

---

## **Overview**

This document defines canonical implementations for **layout patterns**: containers, cards, modals, dialogs, forms, lists, menus, and tables.

**Prerequisites:** Read `UI_PAGE_THEMING_FOUNDATIONS` for token selection rules.

---

# [SPEC-4.2] Containers Ontology

## 4.2.1 General Rules (Binding)

- Containers define structure, not behavior
- Containers MUST use `container`, `window`, or `reference-token` categories
- Containers MUST support adaptive width and responsive padding
- Shadows MUST serve elevation differentiation, not decoration
- Nested containers beyond 3 levels SHOULD be avoided

---

## 4.2.2 Cards (`container`)

**Purpose:** Present grouped content in elevated surface.

**Rules:**

1. Cards MUST contain `header`, `body`, and optional `footer` slots
2. Borders and radius derive from `container` tokens
3. Shadow uses minimal elevation tokens
4. Headers MAY include action buttons
5. Cards use consistent padding per Horizon spacing scale

```css
.card {
  /* Uses secondary background for subtle elevation */
  background-color: rgb(var(--now-color_background--secondary, 245, 246, 247));
  border: 1px solid rgb(var(--now-container-card--border-color, 207, 213, 215));
  border-radius: var(--now-container-card--border-radius, 8px);
  box-shadow: var(--now-static-drop-shadow--sm);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.card__header {
  padding: var(--now-static-space--lg, 1rem);
  border-bottom: 1px solid
    rgb(var(--now-container-card--border-color, 207, 213, 215));
  font-weight: 600;
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
  background-color: rgb(var(--now-container--color, 255, 255, 255));
}

.card__body {
  padding: var(--now-static-space--lg, 1rem);
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
  flex: 1;
}

.card__footer {
  padding: var(--now-static-space--lg, 1rem);
  border-top: 1px solid
    rgb(var(--now-container-card--border-color, 207, 213, 215));
  display: flex;
  justify-content: flex-end;
  gap: var(--now-static-space--sm, 0.5rem);
}
```

**When to Use Cards:**

- ✅ Grouped related content requiring visual elevation
- ✅ Multi-section forms with distinct headers
- ✅ Data tables with context/filters
- ✅ Content blocks needing separation from page

**When NOT to Use Cards:**

- ❌ Page headers and titles
- ❌ Simple sections with direct content
- ❌ Navigation elements
- ❌ Full-page forms without groupings

---

## 4.2.3 Modals and Dialogs (`window`)

**Purpose:** Temporarily interrupt workflow to display/collect focused information.

**Rules:**

1. MUST use semantic tokens from `window` and `window_header`
2. MUST include header, body, and optional footer regions
3. Overlay MUST use alpha-blended tokenized background
4. Focus MUST trap within modal
5. Dimensions follow Horizon modal size scale (sm, md, lg, fullscreen)
6. Only one modal MAY be visible at a time

```css
/* Modal overlay - covers page */
.modal-overlay {
  position: fixed;
  inset: 0;
  background-color: rgba(
    var(--now-window_overlay--background-color, 0, 0, 0),
    var(--now-window_overlay--background-color-alpha, 0.5)
  );
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

/* Modal container */
.modal {
  background-color: rgb(var(--now-window--background-color, 255, 255, 255));
  border: 1px solid rgb(var(--now-window--border-color, 207, 213, 215));
  border-radius: var(--now-window--border-radius, 8px);
  width: 90%;
  max-width: 600px;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  box-shadow: var(--now-static-drop-shadow--lg);
}

/* Modal header */
.modal__header {
  padding: var(--now-static-space--lg, 1rem);
  border-bottom: 1px solid rgb(var(--now-window--border-color, 207, 213, 215));
  font-weight: var(--now-window_header--font-weight, 600);
  font-size: var(--now-window_header--font-size--lg, 1.125rem);
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
  display: flex;
  justify-content: space-between;
  align-items: center;
  /* CRITICAL: Do NOT set background-color here.
     The header inherits the modal container's background to preserve
     the border-radius at the top corners. Setting an opaque background
     would create sharp corners inside the rounded modal container. */
}

/* Modal close button - CRITICAL for dark mode */
.modal__close {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0;
  width: 2rem;
  height: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  /* MUST inherit text color for dark mode inversion */
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
}

.modal__close:hover {
  /* Semi-transparent overlay adapts to both themes */
  background-color: rgba(
    var(
      --now-actionable--primary-negative--background-color--hover,
      16,
      23,
      26
    ),
    0.08
  );
}

.modal__close:active {
  background-color: rgba(
    var(
      --now-actionable--primary-negative--background-color--active,
      16,
      23,
      26
    ),
    0.15
  );
}

.modal__close:focus-visible {
  outline: none;
  box-shadow: inset 0 0 0 2px
    rgb(var(--now-color_focus-ring_shadow, 53, 147, 37));
}

/* Modal body - scrollable if content exceeds height */
.modal__body {
  padding: var(--now-static-space--lg, 1rem);
  overflow-y: auto;
  flex: 1;
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
}

/* Modal footer - typically contains actions */
.modal__footer {
  padding: var(--now-static-space--lg, 1rem);
  border-top: 1px solid rgb(var(--now-window--border-color, 207, 213, 215));
  display: flex;
  justify-content: flex-end;
  gap: var(--now-static-space--sm, 0.5rem);
  /* CRITICAL: Do NOT set background-color here.
     The footer inherits the modal container's background to preserve
     the border-radius at the bottom corners. Setting an opaque background
     would create sharp corners inside the rounded modal container. */
}
```

**Close Button Requirements:**

1. MUST use `color: rgb(var(--now-color_text--primary))` for dark mode inversion
2. Hover MUST use semi-transparent overlay: `rgba(var(--now-button-iconic--bare_tertiary--background-color--hover, 0.08)`
3. MUST have `aria-label="Close"` for accessibility
4. Use `×` character (U+00D7) or SVG icon

---

## 4.2.4 Page Containers (`reference-token`)

**Purpose:** Define root layout surfaces for workspaces.

**Rules:**

1. Page containers MUST define primary scrollable content region
2. Global padding derives from spacing tokens
3. Child containers inherit spacing rhythm (outer:inner ratio ~1.25)
4. Page containers provide focus management and accessibility landmarks

```css
.page {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  padding: var(--now-static-space--lg, 1rem);
  /* Parent theme typically sets this automatically */
  background-color: rgb(var(--now-color_background--primary, 255, 255, 255));
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
}

.page__header {
  margin-bottom: var(--now-static-space--xl, 1.5rem);
}

.page__content {
  flex: 1;
}

.page__footer {
  margin-top: var(--now-static-space--xl, 1.5rem);
}
```

---

## 4.2.5 Expandable/Collapsible UI (Accordion, Tree View)

**Purpose:** Organize hierarchical or segmented content with user-controlled expansion.

**Rules:**

1. Icons MUST use SVG, never unicode characters
2. Icon size MUST be `1rem` (inherits font size)
3. Rotation animation MUST use CSS transforms
4. Icons MUST use `fill: currentColor` to inherit text color

**Down-pointing chevron** (accordions, rotates 180° when collapsed):

```html
<svg
  aria-hidden="true"
  viewBox="0 0 16 16"
>
  <path
    d="M12.5 5a.5.5 0 0 1 .372.834l-4.5 5a.5.5 0 0 1-.744 0l-4.5-5A.5.5 0 0 1 3.5 5z"
  ></path>
</svg>
```

**Right-pointing chevron** (tree views, rotates 90° when expanded):

```html
<svg
  aria-hidden="true"
  viewBox="0 0 16 16"
>
  <path
    d="M5 3.5a.5.5 0 0 1 .834-.372l5 4.5a.5.5 0 0 1 0 .744l-5 4.5A.5.5 0 0 1 5 12.5z"
  ></path>
</svg>
```

### Accordion Implementation

```css
.accordion__item {
  border-bottom: 1px solid
    rgb(var(--now-container--border-color, 207, 213, 215));
}

.accordion__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: var(--now-static-space--md, 0.75rem)
    var(--now-static-space--lg, 1rem);
  cursor: pointer;
  background-color: rgb(var(--now-container--color, 255, 255, 255));
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
}

.accordion__header:hover {
  background-color: rgb(var(--now-color_background--secondary, 245, 246, 247));
}

.accordion__icon {
  display: inline-flex;
  transition: transform 200ms ease;
}

.accordion__icon svg {
  block-size: 1rem;
  inline-size: 1rem;
  fill: currentColor;
}

/* Rotate icon when expanded */
.accordion__item.expanded .accordion__icon {
  transform: rotate(180deg);
}

.accordion__content {
  padding: var(--now-static-space--lg, 1rem);
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
}
```

### Tree View Implementation

```css
.tree__item {
  display: flex;
  align-items: center;
  gap: var(--now-static-space--sm, 0.5rem);
  padding: var(--now-static-space--sm, 0.5rem) var(--now-static-space--lg, 1rem);
  cursor: pointer;
  color: rgb(var(--now-content-tree--color, 16, 23, 26));
}

.tree__item:hover {
  background-color: rgb(
    var(--now-content-tree--background-color--hover, 224, 234, 222)
  );
}

.tree__toggle {
  display: inline-flex;
  transition: transform 100ms ease;
}

.tree__toggle svg {
  block-size: 1rem;
  inline-size: 1rem;
  fill: currentColor;
}

/* Rotate 90° when expanded */
.tree__item.expanded .tree__toggle {
  transform: rotate(90deg);
}

.tree__children {
  padding-left: var(--now-static-space--xl, 1.5rem);
}
```

**Key Rules:**

1. MUST NOT use unicode (▼, ►, ▾, ▸) for toggle icons
2. MUST use inline SVG with `aria-hidden="true"`
3. MUST use `1rem` sizing for icons
4. Icon wrapper MUST have `display: inline-flex`

---

# [SPEC-4.3] Forms Ontology

## 4.3.1 Structure

Forms MUST contain:

- Header with title and optional action buttons
- Body containing fields in responsive columns
- Optional footer for contextual actions

## 4.3.2 Layout Rules

1. Form fields MUST use consistent label alignment and width
2. Labels MUST use tokens from `form-control_label--primary`
3. Spacing between fields derives from spacing scale
4. Sections MAY include side panels for metadata
5. Labels and inputs MUST align vertically/horizontally per token scale

```css
.form {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--now-static-space--lg, 1rem);
}

/* Single column on mobile */
@media (max-width: 768px) {
  .form {
    grid-template-columns: 1fr;
  }
}

.form__field {
  display: flex;
  flex-direction: column;
}

.form__label {
  font-weight: var(--now-form-control_label--primary--font-weight, 600);
  color: rgb(var(--now-form-control_label--primary--color, 16, 23, 26));
  margin-bottom: var(--now-static-space--sm, 0.5rem);
  font-size: 0.875rem;
}

.form__input {
  /* See UI_PAGE_THEMING_CONTROLS for input implementations */
  height: calc(1rem + var(--now-form-field--scale-size-block, 1) * 2rem / 2);
  padding: 0 var(--now-static-space--md, 0.75rem);
  border: 1px solid
    rgb(var(--now-form-control-input--primary--border-color, 207, 213, 215));
  border-radius: var(--now-form-control-input--primary--border-radius, 2px);
  background-color: rgb(var(--now-input--background-color, 255, 255, 255));
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
}

.form__helper-text {
  font-size: 0.75rem;
  color: rgb(var(--now-color_text--secondary, 75, 85, 89));
  margin-top: var(--now-static-space--xs, 0.25rem);
}

.form__error {
  font-size: 0.75rem;
  color: rgb(
    var(--now-indicator--primary_critical--background-color, 229, 34, 57)
  );
  margin-top: var(--now-static-space--xs, 0.25rem);
}
```

---

# [SPEC-4.4] Lists Ontology

## 4.4.1 Structure

Lists MUST contain:

- Toolbar (filters, search, actions)
- Header row
- Data region (rows)
- Optional footer (pagination or counts)

## 4.4.2 Rules

1. List headers/rows use `menu` or `container` tokens
2. Hover and selected states use `--hover` and `--selected` variants
3. Column headers support sorting icons
4. Text overflow uses ellipsis
5. Grouped lists preserve hierarchy via indentation or accent

```css
.list {
  border: 1px solid rgb(var(--now-container--border-color, 207, 213, 215));
  border-radius: var(--now-container--border-radius, 8px);
  overflow: hidden;
}

.list__header,
.list__row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  padding: var(--now-static-space--md, 0.75rem)
    var(--now-static-space--lg, 1rem);
  align-items: center;
}

.list__header {
  background-color: rgb(
    var(--now-menu-list--primary--background-color, 245, 247, 249)
  );
  font-weight: 600;
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
  border-bottom: 1px solid
    rgb(var(--now-container--border-color, 207, 213, 215));
}

.list__row {
  border-bottom: 1px solid
    rgb(var(--now-container--border-color, 207, 213, 215));
  background-color: rgb(var(--now-container--color, 255, 255, 255));
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
}

.list__row:hover {
  background-color: rgb(
    var(--now-menu-list--primary--background-color--hover, 235, 237, 239)
  );
}

.list__row--selected {
  border-left: 4px solid rgb(var(--now-color--primary-1, 0, 128, 163));
  background-color: rgb(
    var(--now-menu-list--primary--background-color--selected, 230, 240, 245)
  );
}

.list__row:last-child {
  border-bottom: none;
}
```

---

# [SPEC-4.5] Menus Ontology

## 4.5.1 Purpose

Provide grouped actionable items in dropdowns, context menus, and command palettes.

## 4.5.2 Rules

1. Menus MUST derive from `menu`, `menu_container` and `menu-list` token categories
2. Menu items use tokenized hover and selected states
3. Divider lines use border tokens from `menu_divider--primary`
4. Focus states visible and non-disruptive
5. Icons inside menu items use `currentColor`
6. Disabled items lower opacity, remove pointer interactions
7. Nested submenus inherit same border and background tokens

```css
.menu {
  background-color: rgb(
    var(--now-menu_container--primary--background-color, 255, 255, 255)
  );
  border: 1px solid
    rgb(var(--now-menu_container--primary--border-color, 207, 213, 215));
  border-radius: var(--now-menu--border-radius, 6px);
  min-width: 200px;
  box-shadow: var(--now-static-drop-shadow--md);
  padding: var(--now-static-space--xs, 0.25rem) 0;
}

.menu__item {
  display: flex;
  align-items: center;
  gap: var(--now-static-space--sm, 0.5rem);
  padding: var(--now-static-space--sm, 0.5rem) var(--now-static-space--lg, 1rem);
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
  font-size: 0.875rem;
  cursor: pointer;
  transition: background-color 100ms linear;
}

.menu__item:hover {
  background-color: rgb(
    var(--now-menu-list--primary--background-color--hover, 240, 242, 243)
  );
}

.menu__item:active {
  background-color: rgb(
    var(--now-menu-list--primary--background-color--active, 230, 235, 237)
  );
}

.menu__item:focus-visible {
  outline: 2px solid rgb(var(--now-color_focus-ring, 53, 147, 37));
  outline-offset: -2px;
}

.menu__item--disabled {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

.menu__divider {
  height: 1px;
  background-color: rgb(
    var(--now-menu-list_divider--primary--color, 207, 213, 215)
  );
  margin: var(--now-static-space--xs, 0.25rem) 0;
}

.menu__item-icon {
  block-size: 1rem;
  inline-size: 1rem;
  fill: currentColor;
}
```

---

# [SPEC-4.10] Tables Ontology

## 4.10.1 Purpose

Display structured data in grid layout with sortable, filterable, selectable rows.

## 4.10.2 Rules

1. Tables MUST use semantic HTML (`<table>`, `<thead>`, `<tbody>`, `<tr>`, `<td>`, `<th>`)
2. Borders and backgrounds derive from `container` or `list` tokens
3. Row hover and selected states use tokenized background variants
4. Header cells use slightly darker background for hierarchy
5. Tables support responsive overflow within scroll containers

```css
.table {
  width: 100%;
  border-collapse: collapse;
  border: 1px solid rgb(var(--now-container--border-color, 207, 213, 215));
  border-radius: var(--now-container--border-radius, 8px);
}

.table th,
.table td {
  padding: var(--now-static-space--md, 0.75rem)
    var(--now-static-space--lg, 1rem);
  border-bottom: 1px solid
    rgb(var(--now-container--border-color, 207, 213, 215));
  text-align: left;
}

.table thead {
  background-color: rgb(var(--now-color_background--tertiary, 226, 229, 231));
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
  font-weight: 600;
}

.table tbody tr {
  background-color: rgb(var(--now-color_background--primary, 255, 255, 255));
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
}

.table tbody tr:hover {
  background-color: rgba(
    var(--now-color--primary-1, 245, 246, 247),
    var(--now-opacity--least, 0.1)
  );
}

.table tbody tr:last-child td {
  border-bottom: none;
}

/* Responsive table wrapper */
.table-wrapper {
  overflow-x: auto;
  border-radius: var(--now-container--border-radius, 8px);
}
```

---

## Implementation Checklist

**Token Compliance:**

- [ ] Containers use `container`, `window`, or appropriate category tokens
- [ ] No hard-coded spacing, borders, or shadows
- [ ] Fallback chains present
- [ ] Colors wrapped with `rgb()`/`rgba()`

**Layout Stability:**

- [ ] No reflow on state changes
- [ ] Consistent spacing rhythm (outer:inner ~1.25)
- [ ] Proper nesting depth (max 3 levels)
- [ ] Responsive breakpoints use relative units

**Accessibility:**

- [ ] Semantic HTML structure
- [ ] Proper landmarks (`<header>`, `<main>`, `<nav>`)
- [ ] Focus management in modals
- [ ] ARIA attributes where needed

**Visual Hierarchy:**

- [ ] Elevation used consistently (shadows on modals > cards > flat)
- [ ] Header backgrounds distinct from body
- [ ] Selected states visually distinct from hover

---

## Common Mistakes (Claude 4 Watch Points)

❌ **Using cards for everything** - Page headers, nav don't need cards  
✅ Cards for grouped elevated content only

❌ **Modal close button not inverting** in dark mode  
✅ Use `color: rgb(var(--now-color_text--primary))`

❌ **Using unicode for accordion icons** (▼, ►)  
✅ Use inline SVG with `1rem` size

❌ **Mixing table and list tokens** arbitrarily  
✅ Tables use `container`, Lists use `menu-list`

❌ **Fixed px spacing** instead of tokens  
✅ Use `--now-static-space--{size}` tokens

❌ **Forms without responsive columns**  
✅ Use CSS Grid with mobile breakpoints

---

## Next Steps

For interactive controls, see `UI_PAGE_THEMING_CONTROLS`.  
For navigation and feedback components, see `UI_PAGE_THEMING_COMPONENTS`.
