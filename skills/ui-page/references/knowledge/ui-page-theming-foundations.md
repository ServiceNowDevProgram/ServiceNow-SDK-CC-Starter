# UI_PAGE_THEMING_FOUNDATIONS

# **ServiceNow Horizon UI Generation Specification - FOUNDATIONS**

**Document:** UI_PAGE_THEMING_FOUNDATIONS  
**Related Documents:**

- `UI_PAGE_THEMING_CONTROLS` - Button, input, form control implementations
- `UI_PAGE_THEMING_LAYOUT` - Container, card, modal, form layout patterns
- `UI_PAGE_THEMING_COMPONENTS` - Navigation, feedback, indicator components

---

## **Preamble**

This document defines the **foundational rules** for generating ServiceNow Horizon-compliant UIs. It covers:

- Design system philosophy and token discipline
- Token selection algorithms and ranking interpretation
- CSS variable structure and usage patterns
- Alias layer conventions

**For component implementations**, see the related documents listed above.

### **Critical for Claude 4 Models**

When working with this specification:

1. **DO NOT overcomplicate** - Follow the first valid token match, document deviations
2. **DO NOT invent tokens** - Only use tokens returned by the `css_variable_explorer` tool or defined aliases
3. **VERIFY used tokens** - All tokens used in CSS must be validated against tokens returned by the `css_variable_explorer` tool or defined aliases.
4. **DO track your decisions** - Use the validation checklists as you work
5. **DO reference exact token names** - Copy token names precisely, including all dashes and underscores

**Working Memory Protocol:**

Track these during generation:

```json
{
  "current_component": "name",
  "token_category": "actionable|form-control|container|etc",
  "aliases_defined": ["--snx-space-inner", "--snx-radius"],
  "deviations_from_rank_1": []
}
```

---

# [SPEC-0] Goals and Scope

### 0.1 Purpose

Generate ServiceNow-aligned interfaces using **HTML, CSS, and design tokens** that:

- Use semantic HTML and token-based CSS
- Align with Horizon Design System
- Maintain accessibility, consistency, and determinism

### 0.2 Scope

**This specification governs:**

- UI generation with HTML, CSS, and CSS variables (design tokens)
- Token selection, fallback, aliasing, and extrapolation rules
- Accessibility and responsive layout requirements

**This specification does NOT govern:**

- JavaScript frameworks or runtime behavior
- Server-side rendering logic
- Non-visual ServiceNow workflows

---

# [SPEC-1] Design System Alignment (Horizon Principles)

### 1.1 Core Principles

#### 1.1.1 Fluid

UIs MUST adapt seamlessly to varying contexts. Use relative units (`rem`, `%`, `auto`) and spacing tokens, not fixed pixels.

#### 1.1.2 Symphonic

Components MUST visually harmonize. Consistent radius, shadows, and typography ensure unified experience.

#### 1.1.3 Accessible

All UIs MUST comply with **WCAG 2.1 AA**:

- Text contrast ≥ 4.5:1
- Visible focus states (tokenized focus-ring colors)
- Keyboard navigation support
- Semantic HTML

#### 1.1.4 Colors

Every color choice should have a clear purpose. Establish a consistent color palette and defining color roles.
Palettes Terms:

- `primary` keyword is intended for use with primary actions or elements important to your UI
- `secondary` keyword is used as a complementary color to the primary colors
  Backgrounds:
- `primary` `background` - default theme background
- `secondary` `background` - can be used as a background, but should be used sparingly. A secondary background provides a strong division between sections and content.s
- Avoid the use of color backgrounds when spacing can easily create a separation of content.
  Text:s
- The `text` variables provide a palette to stylize written content like sentences, paragraphs, or captions.
  Selection:
- `primary` used for selection states that require a strong visual cue. It adds additional flexibility to create a unique selection state that is not related to primary or secondary colors.s
- `secondary` used for selection states that require an alternative visual cue.s
  Semantic:
- Colors from the shared palette are intentionally chosen to convey feedback, status, or urgency, ensuring that they consistently communicate important information as needed to the user.
- terms include `alert`, `warning`, `positive`

### 1.2 Accessibility Standards (Binding Rules)

1. All interactive elements MUST have distinct states: `base`, `hover`, `active`, `focus-visible`, `disabled`
2. Focus rings MUST use inset shadows or outlines (no layout reflow)
3. Screen-reader labels MUST exist for all actionable elements
4. Landmarks (`<header>`, `<nav>`, `<main>`, `<footer>`) SHOULD be used

### 1.3 Motion and Feedback

- Transitions MUST be minimal: 75-100ms (color changes), 100-150ms (active feedback)
- Only animate: `background-color`, `border-color`, `transform`
- Use `cubic-bezier` easing aligned with motion type

### 1.4 Browser Default Styles (CRITICAL)

**When using base HTML elements, you MUST consider browser default styling.**

Browsers apply default styles to many HTML elements that can interfere with design token-based styling:

**Common Browser Defaults:**

```css
/* Examples of browser defaults that MUST be reset */
form {
  display: block;
  margin-top: 0em;
  margin-block-end: 1em; /* ⚠️ Unwanted margin */
  unicode-bidi: isolate;
}

ul,
ol {
  margin-block-start: 1em; /* ⚠️ Unwanted margin */
  margin-block-end: 1em;
  padding-inline-start: 40px; /* ⚠️ Unwanted padding */
}

button {
  padding: 2px 6px; /* ⚠️ Inconsistent with design tokens */
  border: 2px outset buttonface; /* ⚠️ Non-standard border */
}

input,
textarea,
select {
  margin: 0;
  font-family: inherit; /* Usually inherited but not guaranteed */
  font-size: inherit;
}
```

**Rules:**

1. **ALWAYS reset browser defaults** that conflict with design tokens
2. **Use explicit token-based values** for all spacing, typography, and borders
3. **Test with browser DevTools** to identify inherited default styles
4. **Common resets needed:**
   - `margin: 0` on forms, lists, headings, paragraphs
   - `padding: 0` on lists
   - `border: none` on buttons and inputs (then add token-based borders)
   - `font-family`, `font-size`, `line-height` inheritance

**Example - Proper Form Reset:**

```css
/* ✅ CORRECT - Reset browser defaults, use tokens */
.form {
  display: grid;
  margin: 0; /* Reset browser default */
  padding: 0; /* Reset browser default */
  border: none; /* Reset browser default */
  grid-template-columns: 1fr 1fr;
  gap: var(--now-static-space--lg, 1rem); /* Use design tokens */
}

/* ❌ INCORRECT - Browser defaults interfere */
.form-wrong {
  /* Missing resets - browser adds 1em bottom margin */
  display: grid;
  gap: var(--now-static-space--lg, 1rem);
}
```

**Example - Proper List Reset:**

```css
/* ✅ CORRECT - Reset browser defaults */
.menu-list {
  margin: 0; /* Reset browser default */
  padding: 0; /* Reset browser default */
  list-style: none; /* Remove bullets */
  /* Now apply token-based styling */
  background-color: rgb(
    var(--now-menu-list--primary--background-color, 255, 255, 255)
  );
  border-radius: var(--now-menu--border-radius, 6px);
}
```

**WARNING:** Failing to reset browser defaults is a common source of:

- Inconsistent spacing between light and dark modes
- Layout shifts and misalignments
- Unexpected margins breaking container designs
- Non-token values overriding theme variables

---

# [SPEC-2] Consistency and Variable Correctness Framework

### 2.1 Token Integrity Rules (Binding)

1. Each visual property MUST use a valid design token
2. Tokens MUST belong to correct semantic category (e.g., `actionable` for buttons)
3. Fallbacks MUST follow chained `var()` structure:
   ```css
   var(--now-actionable--primary--background-color, var(--now-color--primary-1, 0,128,163))
   ```
4. Colors MUST be wrapped with `rgb()` or `rgba()`:
   ```css
   background-color: rgb(var(--now-color_background--primary, 255, 255, 255));
   ```

### 2.2 Alias Layer (CRITICAL)

**WRONG - Generic alias hides differences:**

```css
:root {
  --snx-radius: 6px; /* Buttons need 6px, inputs need 2px! */
}
.button {
  border-radius: var(--snx-radius);
} /* 6px - correct */
.input {
  border-radius: var(--snx-radius);
} /* 6px - WRONG! Should be 2px */
```

**CORRECT - Component-specific aliases:**

```css
:root {
  --snx-radius-button: var(--now-actionable--border-radius, 6px);
  --snx-radius-input: var(
    --now-form-control-input--primary--border-radius,
    2px
  );
  --snx-radius-container: var(--now-container--border-radius, 8px);
}
.button {
  border-radius: var(--snx-radius-button);
} /* 6px */
.input {
  border-radius: var(--snx-radius-input);
} /* 2px */
```

**Rules:**

1. NEVER create generic aliases that ignore component requirements
2. ALWAYS use component-specific aliases
3. BETTER YET: Use tokens directly when possible
4. Aliases MUST remain stable between generations

### 2.3 Foundation Token Patterns

| Property          | Token Pattern                        | Available Values                                                                                          |
| ----------------- | ------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| **Spacing**       | `--now-static-space--{size}`         | xxs (0.125rem), xs (0.25rem), sm (0.5rem), md (0.75rem), lg (1rem), xl (1.5rem), xxl (2rem), 3xl (2.5rem) |
| **Drop Shadows**  | `--now-static-drop-shadow--{size}`   | sm, md, lg, xl, xxl                                                                                       |
| **Border Radius** | `--now-static-border-radius--{size}` | sm (0.125rem), md (0.25rem), lg (0.5rem)                                                                  |
| **Font Sizes**    | `--now-static-font-size--{size}`     | sm (0.75rem), md (1rem), lg (1.25rem), xl (1.5rem)                                                        |
| **Line Height**   | `--now-static-line-height`           | 1.25 (unitless)                                                                                           |
| **Font Family**   | `--now-font-family`                  | Lato, Arial, sans-serif                                                                                   |

**Control Heights** use scale tokens:

```css
/* Button height */
min-block-size: calc(1rem + var(--now-button--scale-size-block, 1) * 2rem / 2);
/* Form field height */
height: calc(1rem + var(--now-form-field--scale-size-block, 1) * 2rem / 2);
```

### 2.4 No Hard-Coded Values Policy (Binding)

**✅ Correct:**

```css
border-color: rgb(var(--now-actionable--primary--border-color, 0, 128, 163));
```

**❌ Incorrect:**

```css
border-color: #0080a3;
```

If no matching token exists, use neutral fallback with `TODO` comment:

```css
/* TODO: missing token for custom-widget focus ring */
border-color: rgb(var(--now-color_focus-ring, 53, 147, 37));
```

### 2.5 Self-Correction Triggers

**STOP and reconsider if:**

1. **Alias Inconsistency** - About to define alias differently than before
2. **Category Mixing** - Using tokens from 3+ categories for single component
3. **Hard-Coded Value** - Tempted to use literal value
4. **Accessibility Uncertainty** - Unsure about contrast or focus visibility
5. **State Inconsistency** - Different components have different state sets

---

# [SPEC-3] Token Selection and Fuzzy Ranking Heuristics

### 3.1 Semantic Category Selection (Binding)

ALWAYS choose tokens from the appropriate category:

| Element Type       | Token Category    | Example                                                 |
| ------------------ | ----------------- | ------------------------------------------------------- |
| Buttons, CTAs      | `actionable`      | `--now-actionable--primary--background-color`           |
| Inputs, Checkboxes | `form-control`    | `--now-form-control-input--primary--border-color`       |
| Containers, Cards  | `container`       | `--now-container-card--background-color-alpha`          |
| Windows, Modals    | `window`          | `--now-window--border-color`                            |
| Menus, Lists       | `menu`            | `--now-menu-list--primary--background-color`            |
| Navigation         | `navigation`      | `--now-navigation-page_tabs--primary--background-color` |
| Alerts, Banners    | `messaging`       | `--now-messaging--primary_warning--border-color`        |
| Status Indicators  | `indicator`       | `--now-indicator--primary_critical--background-color`   |
| Typography         | `display-type`    | `--now-display-type_label--font-weight`                 |
| Base Values        | `reference-token` | `--now-color--primary`                                  |

**CRITICAL: Subcategories**

Some tokens require subcategories for specific properties:

**Indicators/Badges - Use `_label` for text:**

```css
/* ✅ CORRECT */
.badge {
  background-color: rgb(
    var(--now-indicator--primary_positive--background-color)
  );
  color: rgb(
    var(--now-indicator_label--primary_positive--color)
  ); /* _label subcategory */
}

/* ❌ INCORRECT */
.badge-wrong {
  color: rgb(
    var(--now-indicator--primary_positive--color)
  ); /* This token doesn't exist! */
}
```

### 3.2 Ranking Behavior for Fuzzy Search Results

When `css_variable_explorer` returns multiple tokens:

**STEP 1: Validate top candidate**

- ✓/✗ Property type match (color for color, spacing for padding)
- ✓/✗ Unit type match (rem/px for lengths, rgb for colors)
- ✓/✗ Semantic alignment (category makes sense)
- ✓/✗ Value range reasonable

**STEP 2: Use first valid match**

**STEP 3: Document if deviating**

```css
/* Token Selection: Rank 2 used (rank 1 had unit mismatch) */
```

**Rules:**

1. Order indicates relevance - first = most semantically correct
2. Select first token that matches property type AND semantic intent
3. Skip and document if top-ranked conflicts by unit/type
4. DO NOT combine candidates arbitrarily

### 3.3 Conflict Resolution Hierarchy

When rules compete, apply this precedence:

**TIER 1 (Cannot Violate):**

1. Accessibility (WCAG AA contrast, visible focus)
2. No layout reflow on state changes
3. Semantic category correctness

**TIER 2 (Strongly Prefer):** 4. First-ranked token selection 5. Alias layer consistency 6. Spacing ratio maintenance

**TIER 3 (Optimize When Possible):** 7. Motion constraints 8. Elevation consistency 9. Documentation completeness

**Document conflicts:**

```css
/* CONFLICT: Tier 1 accessibility overrides Tier 2 first-ranked token
   Used second-ranked for WCAG AA compliance */
```

### 3.4 Token Values (For Disambiguation Only)

When `css_variable_explorer` returns values:

- Use values to DECIDE which token fits
- NEVER hardcode the value into CSS
- ALWAYS output the variable reference

Example:

```json
[
  { "name": "--now-actionable--border-width", "value": "1px" },
  { "name": "--now-form-control--border-width", "value": "1px" }
]
```

Usage:

```css
/* Use the token, not the value */
border-width: var(--now-actionable--border-width, 1px);
```

### 3.5 Consistency Matrix

Maintain these spacing/sizing relationships:

| Relationship           | Ratio    | Example Tokens                                                                |
| ---------------------- | -------- | ----------------------------------------------------------------------------- |
| Padding : Margin       | 1 : 1.25 | `--now-static-space--md` (0.75rem) vs `--now-static-space--lg` (1rem)         |
| Radius : Padding       | 1 : 2    | `--now-static-border-radius--lg` (0.5rem) vs `--now-static-space--lg` (1rem)  |
| Focus Ring : Border    | 2 : 1    | Focus ring 2px, border 1px                                                    |
| Label Font : Body Font | 0.9 : 1  | `--now-static-font-size--sm` (0.75rem) vs `--now-static-font-size--md` (1rem) |

---

# [SPEC-9] CSS Variable Structure and Usage

### 9.1 Token Structure

Variables follow:

```
--now-[category]--[element]--[property]--[state]
```

Examples:

- `--now-form-control-input--primary--border-color--focus`
- `--now-actionable--primary--background-color--hover`

### 9.2 Token Values and Dark Mode

Each token has **light** and **dark** mode values:

| Token                           | Light Mode  | Dark Mode   | Purpose     |
| ------------------------------- | ----------- | ----------- | ----------- |
| `--now-color_text--primary`     | 16,23,26    | 255,255,255 | Text color  |
| `--now-container--color`        | 255,255,255 | 5,8,9       | Backgrounds |
| `--now-container--border-color` | 207,213,215 | 115,127,132 | Borders     |

**Implementation:**

```css
/* Token automatically switches between light/dark */
background-color: rgb(var(--now-container--color, 255, 255, 255));
color: rgb(var(--now-color_text--primary, 16, 23, 26));
```

The theme system handles switching automatically.

### 9.3 Fallback Chain Pattern

Most-specific → category base → reference:

```css
color: rgb(
  var(
    --now-form-control_label--primary--color,
    var(--now-color_text--primary, var(--now-color--neutral-18, 16, 23, 26))
  )
);
```

### 9.4 Colors Must Use `rgb()/rgba()`

```css
/* ✅ CORRECT */
background-color: rgb(
  var(--now-actionable--primary--background-color, 0, 128, 163)
);

/* For alpha transparency */
background-color: rgba(
  var(--now-actionable--secondary--background-color, 0, 113, 143),
  var(--now-actionable--background-color-alpha, 0)
);
```

---

# [SPEC-10] Alias Layer (Canonical)

### 10.1 Purpose

Provide stable, reusable names mapping to instance tokens for concise, deterministic CSS.

### 10.2 Canonical Aliases

```css
:root {
  /* Surfaces & borders */
  --snx-color-surface: rgb(var(--now-container--color, 255, 255, 255));
  --snx-color-surface-alt: rgb(
    var(--now-heading--header-primary--color, 245, 247, 249)
  );
  --snx-color-border: rgb(var(--now-container--border-color, 207, 213, 215));

  /* Text */
  --snx-color-text: rgb(var(--now-color_text--primary, 16, 23, 26));
  --snx-color-text-muted: rgb(var(--now-color_text--secondary, 75, 85, 89));

  /* Actions & focus */
  --snx-color-primary: rgb(
    var(--now-actionable--primary--background-color, 0, 128, 163)
  );
  --snx-color-on-primary: rgb(
    var(--now-actionable_label--primary--color, 255, 255, 255)
  );
  --snx-color-focus: rgb(var(--now-color_focus-ring, 53, 147, 37));

  /* Spacing - use static tokens */
  --snx-space-inner: var(--now-static-space--md, 0.75rem);
  --snx-space-outer: var(--now-static-space--lg, 1rem);

  /* Border radius - component-specific */
  --snx-radius-button: var(--now-actionable--border-radius, 6px);
  --snx-radius-input: var(
    --now-form-control-input--primary--border-radius,
    2px
  );
  --snx-radius-container: var(--now-container--border-radius, 8px);

  /* Typography */
  --snx-font-body: var(--now-font-family, system-ui, sans-serif);
  --snx-line-height: var(--now-static-line-height, 1.25);
}
```

**Rules:**

1. Reference `--snx-*` aliases in component CSS where possible
2. DO NOT redefine aliases with different meanings
3. Extend aliases as needed but maintain consistency

---

# [SPEC-14] Glossary

- **Alias Layer:** Local, stable variables (`--snx-*`) that map to instance tokens
- **Category:** Semantic grouping of tokens (e.g., `actionable`, `form-control`)
- **Focus Ring:** High-contrast visual indicator for focus-visible state, no reflow
- **Horizon:** ServiceNow design system principles governing look & feel
- **Token:** CSS custom property (CSS variable) defining a design value
- **Fallback Chain:** Nested `var()` functions providing graceful degradation
- **Semantic Match:** Token category aligns with component purpose

---

## **Implementation Quick Reference**

### Token Search Workflow

1. Identify component type → determine token category
2. Use `css_variable_explorer` to search for tokens
3. Validate top-ranked result (type, unit, semantic fit)
4. Use first valid match
5. Build fallback chain
6. Document deviations from rank 1

### Common Pitfalls for Claude 4

❌ **Creating generic aliases** that hide component differences  
✅ Use component-specific aliases

❌ **Mixing 3+ token categories** in one component  
✅ Stick to primary category, use reference tokens for fallbacks

❌ **Hardcoding values** from token search results  
✅ Use the token variable reference

❌ **Inventing token names** that seem logical  
✅ Must only use tokens from `css_variable_explorer` or defined aliases

❌ **Skipping validation checklists**  
✅ Use TIER 1 > TIER 2 > TIER 3 conflict resolution

### Next Steps

For component implementations, see:

- **`UI_PAGE_THEMING_CONTROLS`** - Buttons, inputs, form controls
- **`UI_PAGE_THEMING_LAYOUT`** - Cards, modals, containers, forms
- **`UI_PAGE_THEMING_COMPONENTS`** - Navigation, feedback, indicators
