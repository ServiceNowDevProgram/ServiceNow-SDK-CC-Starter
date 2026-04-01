# UI_PAGE_THEMING_COMPONENTS

# **ServiceNow Horizon UI Generation Specification - COMPONENTS**

**Document:** UI_PAGE_THEMING_COMPONENTS  
**Related Documents:**

- `UI_PAGE_THEMING_FOUNDATIONS` - Token selection rules, alias patterns, core principles
- `UI_PAGE_THEMING_CONTROLS` - Button, input, form control implementations
- `UI_PAGE_THEMING_LAYOUT` - Container, card, modal, form layout patterns

---

## **Overview**

This document defines canonical implementations for **UI components**: navigation (tabs, breadcrumbs, pagination, stepper), feedback (alerts, toasts), indicators (badges, progress), plus extrapolation guidelines and validation protocols.

**Prerequisites:** Read `UI_PAGE_THEMING_FOUNDATIONS` for token selection rules.

## **Severity Variant Reference**

All messaging, alert, and indicator components use these semantic variants:

| Display Name | Token Variant | Icon Suggestion | Use For                             |
| ------------ | ------------- | --------------- | ----------------------------------- |
| Success      | `positive`    | ✓ checkmark     | Successful operations, valid states |
| Error        | `critical`    | ⚠ alert         | Errors, failures, urgent issues     |
| Warning      | `warning`     | ⚠ warning       | Warnings, cautions                  |
| Info         | `info`        | ⓘ info          | Informational messages              |

**CRITICAL:** Always use `positive`, `critical`, `warning`, `info` — NEVER "success", "error", or "informational".

---

# [SPEC-4.6] Messaging and Banners Ontology

## 4.6.1 Purpose

Provide contextual feedback, warnings, or system status messages.

## 4.6.2 Normative Rules

1. Alert/messaging components MUST use `alert` tokens for backgrounds and borders
2. Severity variants as per global reference (see §Overview)
3. Text color MUST use fallback chain: `--now-alert--color` → `--now-messaging_label--primary--color` → `--now-color_text--primary`
4. Messages MUST include severity icons (SVG only)
5. Close buttons use `--now-alert_button-iconic--{variant}--color` tokens
6. Toast notifications MUST use `--now-container-card--background-color` (NOT `--now-container--color`) for proper dark mode inversion
7. All components MUST maintain WCAG AA contrast (≥4.5:1)

## 4.6.3 Alert/Banner Examples

**Warning Alert:**

```css
.alert {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1rem;
  border-radius: 6px;
}

.alert--warning {
  /* Use alert tokens that invert for dark mode */
  background-color: rgb(
    var(--now-alert--warning--background-color, 239, 224, 176)
  );
  border: 1px solid rgb(var(--now-alert--warning--border-color, 221, 182, 101));
  /* CRITICAL: Use proper fallback chain for text color */
  color: rgb(
    var(
      --now-alert--color,
      var(
        --now-messaging_label--primary--color,
        var(--now-color_text--primary, var(--now-color--neutral-18, 16, 23, 26))
      )
    )
  );
}

.alert--warning .alert__close {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.125rem;
  /* State-specific close button color */
  color: rgb(var(--now-alert_button-iconic--warning--color, 132, 79, 0));
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
}

.alert--warning .alert__close:hover {
  color: rgb(var(--now-alert_button-iconic--warning--color--hover, 84, 50, 0));
  background-color: rgba(
    var(--now-alert_button-iconic--background-color--hover, 115, 127, 132),
    var(--now-alert_button-iconic--background-color-alpha--hover, 0.25)
  );
}
```

**Token Pattern:** All variants follow `--now-alert--{variant}--{property}` structure.

**Toast Notifications:**

```css
.toast {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1rem;
  border-radius: 6px;
  /* CRITICAL: Use card tokens for proper dark mode inversion */
  background-color: rgb(
    var(--now-container-card--background-color, 255, 255, 255)
  );
  border: 1px solid rgb(var(--now-container-card--border-color, 207, 213, 215));
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 1000;
}
```

---

# [SPEC-4.7] Navigation Components Ontology

## 4.7.1 Tabs and Breadcrumbs

**Normative Rules**

1. Tabs MUST use `--now-tabs--*` design tokens for colors and states
2. Tab borders MUST be `0.125rem` (2px) thickness using `--now-color_focus-ring` color
3. Active tabs show bottom border only (not full border)
4. Tabs MUST NOT use rounded corners or background fills (transparent with alpha 0)
5. Tabs MUST NOT have gaps between them (no margin, only internal padding)
6. Tabs MUST NEVER have vertical scrolling (overflow-y must be hidden or clip)
7. Tabs MAY permit horizontal scrolling only (overflow-x may be auto or scroll when content exceeds container width)
8. Elements within tabs (icons, buttons, badges) MUST use proper spacing tokens (e.g., `--now-static-space--sm` or `--now-static-space--xs` for gaps)
9. Elements within tabs MUST NOT interfere with tab padding or layout consistency
10. Breadcrumbs MUST use tokenized separators and link colors

**Tab States:**

| State       | Selector       | Border                      | Font Weight | Distinction                |
| ----------- | -------------- | --------------------------- | ----------- | -------------------------- |
| **Base**    | `.tab`         | 0.125rem transparent bottom | normal      | Default                    |
| **Hover**   | `.tab:hover`   | 0.125rem focus-ring bottom  | normal      | Temporary indicator        |
| **Active**  | `.tab--active` | 0.125rem focus-ring bottom  | **600**     | Current page (persistent)  |
| **Pressed** | `.tab:active`  | 0.125rem outline all sides  | normal      | Click feedback (temporary) |

**Note:** All states use transparent backgrounds (alpha 0). Active = bottom border + bold. Pressed = full outline during click.

**Example:**

```css
.tabs {
  display: flex;
  border-bottom: 1px solid
    rgb(var(--now-container--border-color, 207, 213, 215));
  overflow-x: auto; /* Allow horizontal scrolling when tabs exceed container width */
  overflow-y: hidden; /* Never allow vertical scrolling */
  /* No gap - tabs are adjacent */
}

.tab {
  padding: 0.75rem 1rem;
  color: rgb(var(--now-tabs--color, 16, 23, 26));
  cursor: pointer;
  border: none;
  border-bottom: 0.125rem solid transparent;
  background-color: rgba(
    var(--now-tabs--background-color, 255, 255, 255),
    var(--now-tabs--background-color-alpha, 0)
  );
  font-family: var(--now-font-family, system-ui, sans-serif);
  font-size: 1rem;
  border-radius: 0;
  margin-bottom: -1px; /* Overlap with tabs container border */
}

/* Hover state - bottom border only */
.tab:hover:not(.tab--active) {
  color: rgb(var(--now-tabs--color--hover, 34, 88, 27));
  background-color: rgba(
    var(--now-tabs--background-color--hover, 255, 255, 255),
    var(--now-tabs--background-color-alpha--hover, 0)
  );
  border-bottom-color: rgb(
    var(--now-color_focus-ring, var(--now-color--secondary-1, 0, 128, 163))
  );
}

/* Active tab - bottom border only + bold text (current page) */
.tab--active {
  color: rgb(var(--now-tabs--color, 16, 23, 26));
  background-color: rgba(
    var(--now-tabs--background-color--selected, 255, 255, 255),
    var(--now-tabs--background-color-alpha--selected, 0)
  );
  border-bottom-color: rgb(
    var(--now-color_focus-ring, var(--now-color--secondary-1, 0, 128, 163))
  );
  font-weight: 600;
}

/* Pressed state - full outline during mouse click */
.tab:active:not(.tab--active) {
  outline: 0.125rem solid
    rgb(var(--now-color_focus-ring, var(--now-color--secondary-1, 0, 128, 163)));
  outline-offset: -0.125rem;
  border-bottom-color: transparent;
}
```

**Tabs with Internal Elements (Icons, Buttons, Badges):**

When tabs contain additional elements like icons or close buttons, proper spacing and alignment are critical:

```html
<!-- Tab with icon -->
<button class="tab tab--with-icon">
  <svg
    class="tab__icon"
    aria-hidden="true"
    viewBox="0 0 16 16"
  >
    <path d="M8 2a6 6 0 1 0 0 12A6 6 0 0 0 8 2z"></path>
  </svg>
  <span>Dashboard</span>
</button>

<!-- Tab with close button -->
<button class="tab tab--with-action">
  <span>Document.txt</span>
  <button
    class="tab__close"
    aria-label="Close tab"
    onclick="event.stopPropagation();"
  >
    <svg
      aria-hidden="true"
      viewBox="0 0 16 16"
    >
      <path
        d="M12.5 3.5l-9 9m0-9l9 9"
        stroke-linecap="round"
      ></path>
    </svg>
  </button>
</button>
```

```css
/* Tab with internal elements uses flexbox with proper gap */
.tab--with-icon,
.tab--with-action {
  display: inline-flex;
  align-items: center;
  gap: var(
    --now-static-space--xs,
    0.25rem
  ); /* Proper spacing between elements */
}

/* Icon sizing */
.tab__icon {
  block-size: 1rem;
  inline-size: 1rem;
  fill: currentColor;
  flex-shrink: 0;
  display: block;
}

/* Close button within tab */
.tab__close {
  display: inline-flex;
  justify-content: center;
  padding: 0.125rem;
  background: transparent;
  border-radius: 6px;
  border: none;
  color: inherit;
  cursor: pointer;
  flex-shrink: 0;
  vertical-align: middle;
  margin: 0.75rem 0; /* Vertical margin for proper alignment with tab text */
}

.tab__close:hover {
  background-color: rgba(
    var(--now-actionable--primary--background-color--hover, 16, 23, 26),
    0.1
  );
}
s .tab__close:active {
  background-color: rgb(
    var(--now-actionable--primary--background-color--active, 207, 213, 215)
  );
}

.tab__close:focus-visible {
  outline: none;
  box-shadow: inset 0 0 0 2px
    rgb(var(--now-color_focus-ring_shadow, 53, 147, 37));
}

.tab__close svg {
  block-size: 0.875rem;
  inline-size: 0.875rem;
  stroke: currentColor;
  stroke-width: 1.5;
  fill: none;
}
```

---

## 4.7.2 Side Navigation

**Normative Rules**

1. Background and border MUST derive from `content-tree` tokens for proper dark mode support
2. Active item highlight MUST use tokenized border or background accent
3. Collapsible sections MUST use Horizon motion tokens for transitions
4. Icons MUST use `currentColor`
5. Text colors MUST use `content-tree--color` tokens to ensure proper inversion

**CRITICAL:** Use `content-tree` tokens (NOT `navigation-sidebar`) for proper contrast.

**Example:**

```css
.sidenav {
  /* MUST use content-tree background (255,255,255 light → 5,8,9 dark) */
  background-color: rgb(
    var(--now-content-tree--background-color, 255, 255, 255)
  );
  width: 240px;
  border: 1px solid rgb(var(--now-container-card--border-color, 207, 213, 215));
  border-radius: 6px;
}

.sidenav__item {
  display: flex;
  align-items: center;
  padding: 0.75rem 1rem;
  /* MUST use content-tree color for proper inversion (16,23,26 light → 255,255,255 dark) */
  color: rgb(var(--now-content-tree--color, 16, 23, 26));
  cursor: pointer;
  transition:
    background-color 100ms linear,
    color 100ms linear;
  text-decoration: none;
  border-left: 4px solid transparent;
}

.sidenav__item:hover {
  /* Uses content-tree hover tokens (224,234,222 light → 11,37,7 dark) */
  background-color: rgb(
    var(--now-content-tree--background-color--hover, 224, 234, 222)
  );
  color: rgb(var(--now-content-tree--color--hover, 16, 23, 26));
}

.sidenav__item--active {
  /* Uses content-tree selected tokens for proper contrast */
  background-color: rgb(
    var(--now-content-tree--background-color--selected, 224, 234, 222)
  );
  border-left-color: rgb(var(--now-color--primary-1, 0, 128, 163));
  color: rgb(var(--now-content-tree--color--selected, 16, 23, 26));
  font-weight: 600;
}
```

**Key Token Values:**

| Token                                            | Light Mode  | Dark Mode   | Purpose              |
| ------------------------------------------------ | ----------- | ----------- | -------------------- |
| `--now-content-tree--background-color`           | 255,255,255 | 5,8,9       | Container background |
| `--now-content-tree--color`                      | 16,23,26    | 255,255,255 | Text color (base)    |
| `--now-content-tree--background-color--hover`    | 224,234,222 | 11,37,7     | Hover background     |
| `--now-content-tree--color--hover`               | 16,23,26    | 255,255,255 | Hover text color     |
| `--now-content-tree--background-color--selected` | 224,234,222 | 11,37,7     | Active background    |
| `--now-content-tree--color--selected`            | 16,23,26    | 255,255,255 | Active text color    |

---

## 4.7.3 Pagination

**Purpose:** Navigate through paginated data sets with page numbers and directional controls.

**Normative Rules:**

1. Pagination buttons MUST use bare/borderless style (no borders in default state)
2. Navigation arrows MUST use SVG icons, never unicode characters
3. Active page MUST have distinct background color and bold font weight
4. All interactive states (hover, active, focus, disabled) MUST be implemented
5. Info text (e.g., "Showing 11-20 of 1000") SHOULD precede navigation controls
6. Icon sizing MUST use `1rem` fixed sizing

**Button States:**

| State              | Background                                                                | Opacity | Cursor      | Other                                                   |
| ------------------ | ------------------------------------------------------------------------- | ------- | ----------- | ------------------------------------------------------- |
| **Base**           | transparent                                                               | 1       | pointer     | No border                                               |
| **Hover**          | `rgba(--now-view-control_item--secondary--background-color--hover, 0.08)` | 1       | pointer     | Semi-transparent dark overlay for visibility            |
| **Active/Pressed** | `--now-container--border-color`                                           | 1       | pointer     | Solid darker gray                                       |
| **Focus**          | transparent                                                               | 1       | pointer     | Green inset ring (2px)                                  |
| **Selected**       | `--now-container--border-color`                                           | 1       | pointer     | **Bold text + solid medium gray** (distinct from hover) |
| **Disabled**       | transparent                                                               | 0.4     | not-allowed | No pointer events                                       |

**SVG Icons:**

Pagination MUST use the following SVG icons:

**First page** (far left):

```html
<svg
  aria-hidden="true"
  viewBox="0 0 16 16"
>
  <path
    d="M2 2.5a.5.5 0 0 0-1 0v12a.5.5 0 0 0 1 0zm6.854.646a.5.5 0 0 1 0 .708L4.707 8H14.5a.5.5 0 0 1 0 1H4.707l4.147 4.146a.5.5 0 0 1-.708.708l-5-5a.5.5 0 0 1 0-.708l5-5a.5.5 0 0 1 .708 0"
  ></path>
</svg>
```

**Previous page**:

```html
<svg
  aria-hidden="true"
  viewBox="0 0 16 16"
>
  <path
    d="M6.854 13.146a.5.5 0 0 1-.708.708l-5-5a.5.5 0 0 1 0-.708l5-5a.5.5 0 1 1 .708.708L2.707 8H14.5a.5.5 0 0 1 0 1H2.707z"
  ></path>
</svg>
```

**Next page**:

```html
<svg
  aria-hidden="true"
  viewBox="0 0 16 16"
>
  <path
    d="M9.146 13.146a.5.5 0 0 0 .708.708l5-5a.5.5 0 0 0 0-.708l-5-5a.5.5 0 1 0-.708.708L13.293 8H1.5a.5.5 0 0 0 0 1h11.793z"
  ></path>
</svg>
```

**Last page** (far right):

```html
<svg
  aria-hidden="true"
  viewBox="0 0 16 16"
>
  <path
    d="M14 2.5a.5.5 0 0 1 1 0v12a.5.5 0 0 1-1 0zm-6.854.646a.5.5 0 0 0 0 .708L11.293 8H1.5a.5.5 0 0 0 0 1h9.793l-4.147 4.146a.5.5 0 0 0 .708.708l5-5a.5.5 0 0 0 0-.708l-5-5a.5.5 0 0 0-.708 0"
  ></path>
</svg>
```

**CSS Implementation:**

```css
.pagination-container {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.pagination-info {
  font-size: 0.875rem;
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
}

.pagination {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.pagination__item {
  display: flex;
  align-items: center;
  justify-content: center;
  /* Button sizing uses scale tokens for responsive heights */
  min-width: calc(1rem + var(--now-button--scale-size-block, 1) * 2rem / 2);
  min-block-size: calc(
    1rem + var(--now-button--scale-size-block, 1) * 2rem / 2
  );
  padding: 0 0.5rem;
  border: none; /* Bare style - no borders */
  border-radius: 6px;
  background: transparent;
  cursor: pointer;
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
  font-family: var(--now-font-family, system-ui, sans-serif);
  font-size: 0.875rem;
  transition: all 100ms linear;
}

.pagination__item svg {
  block-size: 1rem;
  inline-size: 1rem;
  fill: currentColor;
}

/* Hover state - semi-transparent for visibility */
.pagination__item:hover:not(:disabled):not(.pagination__item--active) {
  background-color: rgba(
    var(
      --now-view-control_item--secondary--background-color--hover,
      16,
      23,
      26
    ),
    0.08
  );
}

/* Active/pressed state */
.pagination__item:active:not(:disabled):not(.pagination__item--active) {
  background-color: rgb(
    var(
      --now-view-control_item--secondary--background-color--active,
      207,
      213,
      215
    )
  );
}

/* Focus state for keyboard navigation */
.pagination__item:focus-visible {
  outline: none;
  box-shadow: inset 0 0 0 2px
    rgb(var(--now-color_focus-ring_shadow, 53, 147, 37));
}

/* Selected/current page state - solid background for distinction */
.pagination__item--active {
  background-color: rgb(
    var(
      --now-view-control_item--secondary--background-color--selected,
      207,
      213,
      215
    )
  );
  color: rgb(
    var(--now-view-control_item_label--secondary--color--selected, 16, 23, 26)
  );
  font-weight: 600;
}

.pagination__item--active:hover {
  background-color: rgb(
    var(
      --now-view-control_item--secondary--background-color--selected_hover 207,
      213,
      215
    )
  );
  color: rgb(
    var(
      --now-view-control_item_label--secondary--color--selected_hover,
      16,
      23,
      26
    )
  );
}

/* Disabled state */
.pagination__item:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  pointer-events: none;
}
```

**Quick Reference:** Bare style (no borders), SVG icons only (`1rem` size), selected = gray background + bold text, disable at boundaries.

---

## 4.7.4 Stepper (Progress Indicator)

**Purpose:** Visualize multi-step processes showing completed, current, and upcoming steps.

**Normative Rules:**

1. Step indicators MUST be circular with consistent sizing (typically 2rem diameter)
2. Completed steps MUST use checkmark SVG icon, never unicode characters
3. Active step MUST have visual emphasis (filled background + glow/ring)
4. Incomplete steps MUST show step number
5. Step connectors MUST visually link adjacent steps
6. All SVG icons MUST use `fill: currentColor` to inherit indicator color

**Checkmark Icon:**

Completed steps MUST use the following SVG checkmark:

```html
<svg
  aria-hidden="true"
  viewBox="0 0 16 16"
>
  <path
    d="M14.685 3.772a1 1 0 0 1 .043 1.413l-8 8.5a1 1 0 0 1-1.435.022l-4-4a1 1 0 0 1 1.414-1.414l3.271 3.271 7.294-7.75a1 1 0 0 1 1.413-.042"
  ></path>
</svg>
```

**Step States:**

Steppers use semantic tokens with green theming to indicate progression:

| State          | Background                  | Border                | Text/Icon                   | Token Category          |
| -------------- | --------------------------- | --------------------- | --------------------------- | ----------------------- |
| **Completed**  | Light green (224,234→11,37) | Green (34,88→105,168) | Green (34,88→105,168) + SVG | `stepper_step--done`    |
| **Active**     | White (255→5)               | Green (34,88→105,168) | Green (34,88→105,168)       | `stepper_step--partial` |
| **Incomplete** | White (255→5)               | Gray (55,68→188,195)  | Gray (55,68→188,195)        | `stepper_step--none`    |

Each token category (`--done`, `--partial`, `--none`) includes complete sets of tokens for:

- `background-color` - Container fill
- `border-color` - Border color
- `color` - Text/icon color
- Label colors use corresponding `stepper_label--done`, `stepper_label--partial`, and `stepper_label--none` tokens

**IMPORTANT:** All stepper states MUST use the semantic `stepper_step` and `stepper_label` tokens to ensure proper WCAG-compliant dark mode inversion.

**CSS Implementation:**

```css
.stepper {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
}

.step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  flex: 1;
  position: relative;
}

.step__indicator {
  width: 2rem;
  height: 2rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  /* Incomplete state - white background with gray border */
  background-color: rgb(
    var(--now-stepper_step--none--background-color, 255, 255, 255)
  );
  border: 2px solid rgb(var(--now-stepper_step--none--border-color, 55, 68, 74));
  color: rgb(var(--now-stepper_step--none--color, 55, 68, 74));
  font-weight: 600;
  position: relative;
  z-index: 2;
}

.step__indicator svg {
  block-size: 1rem;
  inline-size: 1rem;
  fill: currentColor;
}

/* Completed step - light green background with green border */
.step--completed .step__indicator {
  background-color: rgb(
    var(--now-stepper_step--done--background-color, 224, 234, 222)
  );
  border-color: rgb(var(--now-stepper_step--done--border-color, 34, 88, 27));
  color: rgb(var(--now-stepper_step--done--color, 34, 88, 27));
}

/* Active step - white background with green border */
.step--active .step__indicator {
  background-color: rgb(
    var(--now-stepper_step--partial--background-color, 255, 255, 255)
  );
  border-color: rgb(var(--now-stepper_step--partial--border-color, 34, 88, 27));
  color: rgb(var(--now-stepper_step--partial--color, 34, 88, 27));
  /* Optional: add glow effect */
  box-shadow: 0 0 0 4px
    rgba(var(--now-stepper_step--partial--border-color, 34, 88, 27), 0.2);
}

.step__label {
  font-size: 0.875rem;
  /* MUST use stepper label token (55,68,74 light → 255,255,255 dark) */
  color: rgb(var(--now-stepper_label--none--color, 55, 68, 74));
  text-align: center;
}

.step--active .step__label {
  /* MUST use stepper label active token (16,23,26 light → 255,255,255 dark) */
  color: rgb(var(--now-stepper_label--partial--color--active, 16, 23, 26));
  font-weight: 600;
}

.step--completed .step__label {
  /* MUST use stepper label done token (34,88,27 light → 105,168,94 dark) */
  color: rgb(var(--now-stepper_label--done--color, 34, 88, 27));
}

/* Connector line between steps */
.step__connector {
  position: absolute;
  top: 1rem;
  left: 50%;
  right: -50%;
  height: 2px;
  background-color: rgb(var(--now-container--border-color, 207, 213, 215));
  z-index: 1;
}

.step--completed .step__connector {
  background-color: rgb(
    var(--now-actionable--primary--background-color, 0, 128, 163)
  );
}

.step:last-child .step__connector {
  display: none;
}
```

**Quick Reference:** SVG checkmark (not unicode), `1rem` icons, active step gets glow, circular indicators (2rem), color connectors for completed.

---

# [SPEC-4.8] Indicators and Badges Ontology

## 4.8.1 Purpose

Display short state indicators or counts that accompany other components.

## 4.8.2 Normative Rules

1. MUST use `indicator` tokens corresponding to severity or status
2. Background, border, and text color MUST be derived from the same indicator family
3. Shape MUST be circular or pill-like based on content length
4. Badge MUST align vertically with adjacent text or icons
5. Icon badges MUST use `currentColor` for scalable fill

**CRITICAL:** Badge text MUST use `--now-indicator_label--{variant}--color` (note `_label` subcategory), NOT base indicator color.

## 4.8.3 Example

```css
.badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 1.5em;
  height: 1.5em;
  padding: 0 0.5rem;
  border-radius: var(
    --now-indicator--border-radius,
    12px
  ); /* 12px for pill shape */
  font-size: 0.75rem;
  line-height: 1;
  font-weight: 600;
}

/* Success/positive badge */
.badge--positive {
  /* CRITICAL: Use positive variant, not "success" */
  background-color: rgb(
    var(--now-indicator--primary_positive--background-color, 62, 134, 0)
  );
  /* CRITICAL: Use _label subcategory for text color */
  color: rgb(
    var(--now-indicator_label--primary_positive--color, 255, 255, 255)
  );
  border: 1px solid
    rgb(var(--now-indicator--primary_positive--border-color, 48, 103, 0));
}

/* Error/critical badge */
.badge--critical {
  /* CRITICAL: Use critical variant, not "error" */
  background-color: rgb(
    var(--now-indicator--primary_critical--background-color, 229, 34, 57)
  );
  color: rgb(
    var(--now-indicator_label--primary_critical--color, 255, 255, 255)
  );
  border: 1px solid
    rgb(var(--now-indicator--primary_critical--border-color, 179, 27, 44));
}

/* Warning badge */
.badge--warning {
  background-color: rgb(
    var(--now-indicator--primary_warning--background-color, 219, 137, 0)
  );
  color: rgb(var(--now-indicator_label--primary_warning--color, 255, 255, 255));
  border: 1px solid
    rgb(var(--now-indicator--primary_warning--border-color, 171, 107, 0));
}

/* Info badge */
.badge--info {
  /* CRITICAL: Use info variant, not "informational" */
  background-color: rgb(
    var(--now-indicator--primary_info--background-color, 0, 121, 204)
  );
  color: rgb(var(--now-indicator_label--primary_info--color, 255, 255, 255));
  border: 1px solid
    rgb(var(--now-indicator--primary_info--border-color, 0, 92, 155));
}
```

**Token Pattern:** Background and border use `--now-indicator--primary_{variant}--{property}`, text uses `--now-indicator_label--primary_{variant}--color`.

---

# [SPEC-4.9] Progress Indicators and Spinners Ontology

## 4.9.1 Purpose

Communicate ongoing operations or loading states.

## 4.9.2 Normative Rules

1. MUST use tokens from `loading_indicator` or `progress_bar`
2. Animation MUST be minimal and token-controlled
3. Progress bars MUST use background and fill tokens
4. Spinners MUST use consistent stroke widths and radii
5. Color MUST maintain contrast with surrounding surface

## 4.9.3 Example

```css
.progress {
  width: 100%;
  height: 0.5rem;
  background-color: rgb(
    var(--now-progress-bar--background-color, 240, 242, 243)
  );
  border-color: rgb(var(--now-progress-bar--border-color,));
  border-radius: var(--now-static-border-radius--lg, 0.5rem);
  border-width: var(--now-progress-bar--border-width, 1px);
  overflow: hidden;
}

.progress__fill {
  width: 40%; /* Dynamic based on progress */
  height: 100%;
  background-color: rgb(
    var(--now-progress-bar_path--initial--background-color, 0, 128, 163)
  );
  transition: width 200ms ease;
}

.spinner {
  border: 2px solid
    rgba(var(--now-loading_indicator--primary--color, 0, 128, 163), 0.2);
  border-top-color: rgb(
    var(--now-loading_indicator--primary--color, 0, 128, 163)
  );
  border-radius: 50%;
  width: 1.5rem;
  height: 1.5rem;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  100% {
    transform: rotate(360deg);
  }
}
```

---

# [SPEC-5] Extrapolation Guidelines (Horizon-Consistent Extensions)

## 5.1 Purpose

When components are not explicitly defined in the Horizon system or this spec, the LLM **MUST** extrapolate using the philosophies, token categories, spacing rhythm, and state logic established herein.

## 5.2 Extrapolation Principles (Normative)

1. **Philosophy First:** Extrapolated components **MUST** embody _Fluid, Symphonic, Accessible_ principles
2. **Category Fidelity:** Extrapolated elements **MUST** select tokens from the closest semantic category (e.g., informational banners → `messaging`; contextual overlays → `menu`/`window`)
3. **State Parity:** Extrapolated interactive elements **MUST** implement the standard state set: `base`, `hover`, `active`, `focus-visible`, `disabled`, and any relevant `selected` or `invalid` states
4. **Rhythm Preservation:** Spacing and typography **MUST** follow the same scale indices used by similar components
5. **Elevation Discipline:** Surfaces **MUST** adopt the same elevation scale as comparable containers (no arbitrary shadows)
6. **Focus Integrity:** Extrapolated controls **MUST** use inset focus rings/outlines without reflow
7. **Alias Reuse:** The alias layer (`--snx-*`) **MUST** be reused; new aliases MAY be added but MUST NOT conflict with existing names

## 5.3 Pattern Recognition and Analogical Reasoning

When generating a component similar to an existing one, models with extended thinking SHOULD explicitly reason:

**ANALOGY TEMPLATE:**

```
New Component: [name]
Most Similar To: [existing component]
Shared Characteristics:
  - [e.g., interactive control]
  - [e.g., has hover/focus states]
  - [e.g., contains text and icon]

Token Category Inheritance:
  ✓ Same category because: [reasoning]
  ✗ Different category because: [reasoning]

State Implementation:
  ✓ Copy state pattern because: [reasoning]
  ✗ Modify state pattern because: [reasoning]

Spacing/Sizing:
  ✓ Use same scale indices because: [reasoning]
  ✗ Use different scale because: [reasoning]
```

This explicit analogical reasoning prevents drift while enabling intelligent extrapolation.

**Consistency Cross-Check:**
After generating each new component, ask:

- "If I saw this component and Component X side-by-side, would they feel cohesive?"
- "Have I used the same token for the same visual purpose as Component Y?"
- "Does this state change feel like other state changes in this system?"

## 5.4 Mapping Table for "Nearest Neighbor" Categories

| New/Undefined Element | Recommended Base Analogue | Token Category Guidance                         |
| --------------------- | ------------------------- | ----------------------------------------------- |
| Split button          | Button + Menu             | `actionable` for trigger; `menu` for list       |
| Command palette       | Menu/Dialog hybrid        | `menu` for list; `window` for container         |
| Inline tag editor     | Pill/Chip                 | `form-control-pill`                             |
| Empty state panel     | Card + Messaging          | `container` for surface; `messaging` for accent |
| Inline toaster        | Banner/Toast              | `messaging` (severity-based variants)           |
| Stepper               | Navigation + Indicators   | `navigation` for steps; `indicator` for status  |

**LLM Directive:**

> When extrapolating, the model MUST explicitly declare which analogue and token categories were chosen, in a short comment at the start of the CSS block.

## 5.5 Example — "Command Palette" (Extrapolated)

```css
/* EXTRAPOLATION REASONING:
   New Component: Command Palette
   Most Similar To: Menu + Modal Dialog
   Shared Characteristics:
     - Interactive list with selectable items
     - Overlays main content
     - Contains search input + results list
   Token Category: menu (list items) + window (container)
   State Pattern: Copied from menu-item (hover, focus-visible, aria-selected)
   Spacing: Reusing --snx-space-outer for consistency with other overlays
*/
.command-palette {
  background-color: rgb(var(--now-window--background-color, 255, 255, 255));
  border: 1px solid rgb(var(--now-window--border-color, 207, 213, 215));
  border-radius: var(--now-window--border-radius, 8px);
  box-shadow: var(--now-static-drop-shadow--md);
  width: 640px;
  max-height: 70vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.command-palette__input {
  padding: var(--now-static-space--lg, 1rem);
  border-bottom: 1px solid rgb(var(--now-window--border-color, 207, 213, 215));
  font-size: 1rem;
}

.command-palette__list {
  overflow-y: auto;
  background-color: rgb(
    var(--now-menu-list--primary--background-color, 255, 255, 255)
  );
}

.command-palette__item {
  padding: var(--now-static-space--sm, 0.5rem) var(--now-static-space--lg, 1rem);
  cursor: pointer;
}

.command-palette__item:hover,
.command-palette__item[aria-selected="true"] {
  background-color: rgb(var(--now-color_background--secondary, 240, 242, 243));
}

.command-palette__item:focus-visible {
  outline: 1px solid rgb(var(--now-color_focus-ring, 53, 147, 37));
  outline-offset: -1px;
}
```

---

# [SPEC-6] Multi-Stage Validation Protocol

## 6.1 Overview

Before finalizing output, the model **MUST** perform automated self-checks to enforce consistency, accessibility, and token correctness. Models with extended thinking capabilities SHOULD use the structured templates below.

## 6.2 Validation Checklist (Execute Before Finalization)

**Token Compliance:**

- [ ] All visual properties use instance tokens or alias variables
- [ ] Tokens match semantic category (e.g., `actionable` for buttons)
- [ ] Fallback chains present, degrade to neutral reference tokens
- [ ] Colors wrapped: `rgb(var(...))` or `rgba(var(...), opacity)`
- [ ] No hard-coded brand values

**Accessibility (WCAG 2.1 AA):**

- [ ] Semantic HTML used (button, input, label, nav, etc.)
- [ ] Focus visible with inset ring/outline (`:focus-visible`, no reflow)
- [ ] Contrast ≥4.5:1 (foreground/background)
- [ ] Labels associated (`<label for>`, `aria-label`, `aria-labelledby`)
- [ ] Keyboard navigation supported
- [ ] ARIA attributes where needed (`aria-current`, `aria-disabled`, etc.)

**Consistency:**

- [ ] Spacing scale indices consistent across component family
- [ ] Border widths constant across states (use inset shadow for focus)
- [ ] Motion limited to approved properties (100ms linear)
- [ ] Alias layer reused; no conflicts
- [ ] State set complete: base, hover, active, focus-visible, disabled

**System Coherence:**

- [ ] Components feel cohesive as a family
- [ ] Elevation levels used consistently
- [ ] First-fit token selection with documented deviations only

## 6.3 Example Self-Check Comment Block

```css
/* SELF-CHECK
Token compliance: PASS (all variables → actionable/form-control/container)
Accessibility: PASS (focus-visible inset ring, labels present, AA contrast via tokens)
Consistency: PASS (spacing inner=--snx-space-inner, outer=--snx-space-outer; radius=--snx-radius)
Determinism: PASS (selected first-ranked tokens; 1 documented deviation for focus color)
*/
```

---

# [SPEC-7] Implementation Patterns, Resets, and Layout Stability

## 7.1 Browser Defaults Reset (Normative)

1. Controls, headings, paragraphs, and lists **MUST** reset browser margins/padding where they conflict with tokens
2. `box-sizing: border-box` **MUST** be applied to component blocks
3. Form controls **MUST** explicitly set font family/size/line-height from tokens

**Example Reset Snippet**

```css
.snx-root,
.snx-root * {
  box-sizing: border-box;
}
.snx-root h1,
.snx-root h2,
.snx-root h3,
.snx-root p,
.snx-root ul,
.snx-root ol,
.snx-root input,
.snx-root textarea,
.snx-root select,
.snx-root button {
  margin: 0;
  padding: 0;
}
.snx-root input,
.snx-root select,
.snx-root textarea,
.snx-root button {
  font-family: var(
    --now-font-family,
    system-ui,
    -apple-system,
    "Segoe UI",
    Roboto,
    sans-serif
  );
  line-height: var(--now-static-line-height, 1.25);
  font-size: var(--now-static-font-size--md, 1rem);
}
```

## 7.2 Layout Stability (No Reflow) (Normative)

- Focus and hover **MUST NOT** alter layout metrics
- Border widths **MUST** remain constant; use inset shadow or outline for focus
- Font-weight changes **SHOULD** be avoided unless constrained (e.g., fixed width containers)

**Correct vs. Incorrect**

```css
/* ❌ Incorrect: causes reflow */
.btn:focus {
  border-width: 2px;
}

/* ✅ Correct: inset focus ring, no reflow */
.btn:focus-visible {
  box-shadow: inset 0 0 0 1px rgb(var(--now-color_focus-ring, 53, 147, 37));
}
```

## 7.3 Border vs Box-Shadow (Normative)

- Structural boundaries → **border**
- Focus emphasis → **inset box-shadow** or **outline** with offset
- Double borders (border + non-inset shadow of similar color) **MUST NOT** occur

---

# [SPEC-12] LLM Behavioral Guardrails (Inline, Binding)

1. **Ranking:** Treat ordered token lists as descending semantic relevance; apply first-fit selection with documented deviations
2. **No Invented Tokens:** Do not create variable names not surfaced by the token explorer or alias layer
3. **No Hardcoded Values:** Literal brand colors, fixed radii, and static spacings are prohibited except as temporary TODO fallbacks
4. **Explain Deviations:** When a higher-ranked token is skipped, include a concise `/* rationale */` comment
5. **Use Thinking Protocols:** Models with extended thinking capabilities SHOULD use the reasoning checkpoints, planning phases, and validation templates throughout this specification
6. **Monitor Self-Correction Triggers:** Pay attention to the self-correction triggers defined in `UI_PAGE_THEMING_FOUNDATIONS` and pause when they activate
7. **Emit Self-Check Block:** Append a brief "SELF-CHECK" comment block (see §6.3) in deliverables to certify compliance
8. **Deterministic Structure:** Output order MUST follow: (1) Aliases, (2) Component CSS, (3) HTML examples, (4) Self-check comments

## Summary

This document provides complete implementations for:

**Navigation Components:**

- Tabs (with internal elements support)
- Side navigation (using content-tree tokens)
- Pagination (bare style with SVG icons)
- Stepper (progress indicators with semantic tokens)

**Feedback Components:**

- Alerts and banners (severity-based with proper variants)
- Toast notifications (using card tokens)
- Badges and indicators (with `_label` subcategory)
- Progress bars and spinners

**Guidelines:**

- Extrapolation principles for undefined components
- Multi-stage validation protocol
- Implementation patterns and browser resets
- LLM behavioral guardrails

**Cross-References:**

- For foundational token selection rules, see `UI_PAGE_THEMING_FOUNDATIONS`
- For interactive controls (buttons, inputs, forms), see `UI_PAGE_THEMING_CONTROLS`
- For layout patterns (containers, cards, modals), see `UI_PAGE_THEMING_LAYOUT`

All examples use validated design tokens and follow WCAG 2.1 AA accessibility standards with proper dark mode support.
