# UI_PAGE_THEMING_CONTROLS

# **ServiceNow Horizon UI Generation Specification - CONTROLS**

**Document:** UI_PAGE_THEMING_CONTROLS  
**Related Documents:**

- `UI_PAGE_THEMING_FOUNDATIONS` - Token selection rules, alias patterns, core principles
- `UI_PAGE_THEMING_LAYOUT` - Container, card, modal, form layout patterns
- `UI_PAGE_THEMING_COMPONENTS` - Navigation, feedback, indicator components

---

## **Overview**

This document defines canonical implementations for **interactive controls**: buttons, inputs, checkboxes, radio buttons, selects, toggles, and pills/chips.

**Prerequisites:** Read `UI_PAGE_THEMING_FOUNDATIONS` for token selection rules and alias layer conventions.

---

# [SPEC-4.1] Controls Ontology

## 4.1.1 General Rules (Binding)

1. All controls MUST use semantic HTML (`<button>`, `<input>`, `<select>`)
2. All controls MUST derive styles from `actionable` or `form-control` token categories
3. Controls MUST support states: `base`, `hover`, `active`, `focus-visible`, `disabled`, `selected`
4. Controls MUST NOT cause layout reflow during state changes
5. Focus indicators MUST use inset box-shadow or outline (no border width changes)

**Focus without reflow:**

```css
/* ✅ Correct - no reflow */
.my-input {
  border: 1px solid rgb(var(--now-form-control-input--primary--border-color));
}
.my-input:focus-visible {
  border-color: rgb(
    var(--now-form-control-input--primary--border-color--focus)
  );
  box-shadow: inset 0 0 0 1px
    rgb(var(--now-form-control_focus-ring--primary--color));
}

/* ❌ Incorrect - causes reflow */
.bad-input:focus {
  border-width: 2px;
}
```

---

## 4.1.2 Buttons (`actionable`)

**Purpose:** Trigger immediate actions on user interaction.

**Variants:** `primary`, `secondary`, `tertiary`, `brand-primary`, `brand-secondary`, `iconic`, `bare`

**States:** `base`, `hover`, `active`, `focus-visible`, `disabled`, `selected`

### Basic Button Implementation

```css
.btn {
  display: inline-flex;
  align-items: center;
  gap: var(--now-static-space--sm, 0.5rem);
  /* Responsive height using scale token */
  min-block-size: calc(
    1rem + var(--now-button--scale-size-block, 1) * 2rem / 2
  );
  padding-block: calc(0.375rem * var(--now-button--scale-size-block, 1));
  padding-inline: calc(1rem * var(--now-button--scale-size-inline, 1));
  border: 1px solid transparent;
  border-radius: var(--now-actionable--border-radius, 6px);
  font-family: var(--now-font-family, system-ui, sans-serif);
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 100ms linear;
}

/* Primary button - filled with brand color */
.btn--primary {
  background-color: rgb(
    var(--now-actionable--primary--background-color, 0, 128, 163)
  );
  color: rgb(var(--now-actionable_label--primary--color, 255, 255, 255));
  border-color: rgb(var(--now-actionable--primary--border-color, 0, 128, 163));
}

.btn--primary:hover {
  background-color: rgb(
    var(--now-actionable--primary--background-color--hover, 0, 138, 173)
  );
}

.btn--primary:active {
  background-color: rgb(
    var(--now-actionable--primary--background-color--active, 0, 108, 143)
  );
}

.btn--primary:focus-visible {
  box-shadow: inset 0 0 0 2px
    rgb(var(--now-color_focus-ring_shadow, 53, 147, 37));
}

.btn--primary:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
```

### Secondary/Tertiary Button Alpha Channel (CRITICAL)

**Common Mistake:** Secondary/tertiary buttons use RGBA with alpha for transparency, NOT solid RGB.

```css
/* ✅ CORRECT - uses RGBA with separate alpha token */
.btn--secondary {
  background-color: rgba(
    var(--now-actionable--secondary--background-color, 0, 113, 143),
    var(--now-actionable--background-color-alpha, 0)
  );
  color: rgb(
    var(
      --now-button--secondary--color,
      var(
        --now-actionable_label--secondary--color,
        var(--now-color--secondary-1)
      )
    )
  );
  border: 1px solid
    rgb(
      var(
        --now-button--secondary--border-color,
        var(
          --now-actionable--secondary--border-color,
          var(--now-color--secondary-1)
        )
      )
    );
}

.btn--secondary:hover {
  background-color: rgba(
    var(--now-actionable--secondary--background-color--hover, 0, 76, 97),
    var(--now-actionable--background-color-alpha--hover, 0.08)
  );
}

/* ❌ INCORRECT - loses alpha channel */
.btn--secondary-wrong {
  background-color: rgb(var(--now-actionable--secondary--background-color));
  /* This creates a dark solid button instead of transparent */
}
```

**Alpha Channel Pattern:**

| State  | Alpha Token                                        | Value | Effect            |
| ------ | -------------------------------------------------- | ----- | ----------------- |
| base   | `--now-actionable--background-color-alpha`         | 0     | Fully transparent |
| hover  | `--now-actionable--background-color-alpha--hover`  | 0.08  | 8% tint           |
| active | `--now-actionable--background-color-alpha--active` | 0.12  | 12% tint          |

### Split Buttons & Dropdown Buttons

**Split Button Structure:**

- Two adjacent buttons: main action + dropdown trigger
- Both share borders (remove border-radius on adjacent sides)
- Trigger uses scale token for width

**Trigger Width vs Icon Size (CRITICAL):**

- **Button trigger WIDTH** → Uses `--now-button--scale-size` token (scalable)
- **Icon SIZE** → Uses `1rem` (fixed, inherits font size)

```css
/* Split button trigger */
.btn--split-trigger {
  padding: 0;
  /* WIDTH uses scale token */
  inline-size: calc(1rem + var(--now-button--scale-size, 1) * 2rem / 2);
  width: calc(1rem + var(--now-button--scale-size, 1) * 2rem / 2);
}

/* Icon sizing - FIXED at 1rem */
.btn--split-trigger svg,
.btn--dropdown svg {
  block-size: 1rem;
  inline-size: 1rem;
  fill: currentColor;
}
```

**Standard Chevron SVG (down-pointing):**

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

**Example HTML:**

```html
<!-- Split button -->
<div class="btn-group">
  <button class="btn btn--primary">Save</button>
  <button
    class="btn btn--primary btn--split-trigger"
    aria-label="More options"
  >
    <svg
      aria-hidden="true"
      viewBox="0 0 16 16"
    >
      <path
        d="M12.5 5a.5.5 0 0 1 .372.834l-4.5 5a.5.5 0 0 1-.744 0l-4.5-5A.5.5 0 0 1 3.5 5z"
      ></path>
    </svg>
  </button>
</div>

<!-- Dropdown button -->
<button class="btn btn--primary btn--dropdown">
  <span>Actions</span>
  <svg
    aria-hidden="true"
    viewBox="0 0 16 16"
  >
    <path
      d="M12.5 5a.5.5 0 0 1 .372.834l-4.5 5a.5.5 0 0 1-.744 0l-4.5-5A.5.5 0 0 1 3.5 5z"
    ></path>
  </svg>
</button>
```

**Key Rules:**

1. MUST NOT use unicode characters (▼, ▾) for dropdown icons
2. MUST use inline SVG with `aria-hidden="true"`
3. Icon size MUST be `1rem` (inherits font size)
4. Dropdown buttons MUST use `display: inline-flex` with gap

---

## 4.1.3 Input Fields (`form-control-input`)

**Purpose:** Capture text, numeric, or typed values.

**Rules:**

1. Inputs MUST use `form-control-input` tokens
2. Borders MUST remain constant width across states
3. Focus state MUST use inset shadow ring
4. Placeholder text MUST use secondary text tokens
5. Validation errors MUST use `invalid` or `warning` variants

```css
input[type="text"],
input[type="email"],
input[type="number"],
textarea {
  /* Responsive height using scale token */
  height: calc(1rem + var(--now-form-field--scale-size-block, 1) * 2rem / 2);
  padding: 0 var(--now-static-space--md, 0.75rem);
  border: 1px solid
    rgb(var(--now-form-control-input--primary--border-color, 115, 127, 132));
  border-radius: var(--now-form-control-input--primary--border-radius, 2px);
  /* Dedicated input background for visibility in dark mode */
  background-color: rgb(var(--now-input--background-color, 255, 255, 255));
  color: rgb(var(--now-color_text--primary, 16, 23, 26));
  font-family: var(--now-font-family, system-ui, sans-serif);
  font-size: 1rem;
}

input::placeholder {
  color: rgb(var(--now-color_text--secondary, 75, 85, 89));
  opacity: 1; /* Override browser defaults */
}

input:focus-visible {
  outline: none;
  border-color: rgb(
    var(--now-form-control-input--primary--border-color--focus, 53, 147, 37)
  );
  box-shadow: inset 0 0 0 1px
    rgb(var(--now-form-control_focus-ring--primary--color, 53, 147, 37));
}

/* Invalid state */
input:invalid,
input[aria-invalid="true"] {
  border-color: rgb(
    var(--now-form-control-input--primary--border-color--invalid, 229, 34, 57)
  );
}

input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

---

## 4.1.4 Checkboxes & Radio Buttons (`form-control-input-selection`)

**Key Difference:**

- **Checkboxes** use `form-control-input-selection--primary`
- **Radio buttons** use `form-control-input-selection--secondary`

**States:** `base`, `hover`, `active`, `checked`, `invalid`, `disabled`, combinations

### Checkbox Implementation

```css
input[type="checkbox"] {
  width: 1rem;
  height: 1rem;
  border: 1px solid
    rgb(
      var(
        --now-form-control-input-selection--primary--border-color,
        115,
        127,
        132
      )
    );
  border-radius: var(
    --now-form-control-input-selection--primary--border-radius,
    2px
  );
  background-color: rgb(
    var(
      --now-form-control-input-selection--primary--background-color,
      255,
      255,
      255
    )
  );
  cursor: pointer;
}

input[type="checkbox"]:checked {
  background-color: rgb(
    var(
      --now-form-control-input-selection--primary--background-color--selected,
      0,
      128,
      163
    )
  );
  border-color: rgb(
    var(
      --now-form-control-input-selection--primary--border-color--selected,
      0,
      128,
      163
    )
  );
}

input[type="checkbox"]:focus-visible {
  box-shadow: 0 0 0
    var(--now-form-control_focus-halo--primary--border-width, 2px)
    rgb(var(--now-form-control_focus-ring--secondary--color, 53, 147, 37));
}
```

### Radio Button Implementation

```css
input[type="radio"] {
  width: 1rem;
  height: 1rem;
  border: 1px solid
    rgb(
      var(
        --now-form-control-input-selection--secondary--border-color,
        115,
        127,
        132
      )
    );
  border-radius: 50%; /* Circular */
  background-color: rgb(
    var(
      --now-form-control-input-selection--secondary--background-color,
      255,
      255,
      255
    )
  );
  cursor: pointer;
}

input[type="radio"]:checked {
  background-color: rgb(
    var(
      --now-form-control-input-selection--secondary--background-color--selected,
      0,
      128,
      163
    )
  );
  border-color: rgb(
    var(
      --now-form-control-input-selection--secondary--border-color--selected,
      0,
      128,
      163
    )
  );
}

input[type="radio"]:focus-visible {
  box-shadow: 0 0 0
    var(--now-form-control_focus-halo--secondary--border-width, 2px)
    rgb(var(--now-form-control_focus-ring--secondary--color, 53, 147, 37));
}
```

**Subcategories:**

- `form-control-input-selection_indicator--{variant}` - checkmark/radio dot
- `form-control_label--primary` - label text
- `form-control_focus-halo--{variant}` - focus ring width
- `form-control_focus-ring--secondary` - focus ring color

---

## 4.1.5 Select / Dropdown Fields

**Purpose:** Display and select single value from list.

**Rules:**

1. MUST use `form-control-input--primary` tokens
2. Support hover, focus, opened states
3. Focus indicator uses inset shadow
4. Dropdown menus align with `menu` tokens

**Subcategories:**

- `form-control_value--primary` - selected value text
- `form-control-input_trigger--primary` - dropdown icon
- `form-control-input_inner-shadow--primary` - inner shadows

```css
select {
  appearance: none;
  height: calc(1rem + var(--now-form-field--scale-size-block, 1) * 2rem / 2);
  padding: 0 var(--now-static-space--md, 0.75rem);
  background-color: rgb(
    var(--now-form-control-input--primary--background-color, 255, 255, 255)
  );
  border: 1px solid
    rgb(var(--now-form-control-input--primary--border-color, 207, 213, 215));
  border-radius: var(--now-form-control-input--primary--border-radius, 2px);
  color: rgb(var(--now-form-control_value--primary--color, 16, 23, 26));
  font-family: var(--now-font-family, system-ui, sans-serif);
  font-size: 1rem;
  cursor: pointer;
}

select:hover {
  background-color: rgb(
    var(
      --now-form-control-input--primary--background-color--hover,
      245,
      246,
      247
    )
  );
}

select:focus-visible {
  outline: none;
  border-color: rgb(
    var(--now-form-control-input--primary--border-color--focus, 53, 147, 37)
  );
  box-shadow: inset 0 0 0 1px
    rgb(var(--now-form-control_focus-ring--primary--color, 53, 147, 37));
}

select:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

---

## 4.1.6 Toggles and Sliders (`form-control_track`, `form-control_handle`)

**Purpose:** Binary or range inputs with animated handles.

**Rules:**

1. Track and handle MUST derive from distinct subcategories
2. Selected and hover states use `--selected` and `--selected_hover` tokens
3. Disabled state grays out while preserving affordance
4. Focus uses inset halo ring

```css
/* Toggle track */
.toggle-track {
  position: relative;
  width: 2.5rem;
  height: 1.25rem;
  border-radius: var(--now-form-control_track--border-radius, 12px);
  background-color: rgb(
    var(--now-form-control_track--background-color, 170, 178, 182)
  );
  cursor: pointer;
  transition: background-color 100ms linear;
}

.toggle-track:hover {
  background-color: rgb(
    var(--now-form-control_track--background-color--hover, 151, 162, 166)
  );
}

.toggle-track.selected {
  background-color: rgb(
    var(--now-form-control_track--background-color--selected, 0, 128, 163)
  );
}

/* Toggle handle */
.toggle-handle {
  position: absolute;
  top: 2px;
  left: 2px;
  width: calc(1.25rem - 4px);
  height: calc(1.25rem - 4px);
  border-radius: 50%;
  background-color: rgb(
    var(--now-form-control_handle--background-color, 255, 255, 255)
  );
  border: 1px solid
    rgb(var(--now-form-control_handle--border-color, 115, 127, 132));
  transition: transform 100ms linear;
}

.toggle-track.selected .toggle-handle {
  transform: translateX(1.25rem);
  background-color: rgb(
    var(--now-form-control_handle--background-color--selected, 255, 255, 255)
  );
  border-color: rgb(
    var(--now-form-control_handle--border-color--selected, 0, 128, 163)
  );
}

.toggle-track:focus-visible {
  box-shadow: 0 0 0 2px rgb(var(--now-color_focus-ring_shadow, 53, 147, 37));
}
```

---

## 4.1.7 Pills and Chips (`form-control-pill`)

**Purpose:** Selected items in multi-select or tagging contexts.

**Token Naming:** Pills support TWO equivalent conventions:

- **Short (recommended):** `--now-pill--*`
- **Long (alternate):** `--now-form-control-pill--*`

Both have identical values.

**Rules:**

1. Background and border derive from pill tokens
2. Text color uses pill color tokens (invert for dark mode)
3. Hover and selected states alter background and color
4. Remove icons MUST inherit text color via `currentColor`

```css
.pill {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.25rem 0.75rem;
  border-radius: var(--now-form-control-pill--border-radius 16px);
  /* Background inverts: 255,255,255 → 5,8,9 */
  background-color: rgb(var(--now-pill--background-color, 255, 255, 255));
  /* Text inverts: 16,23,26 → 255,255,255 */
  color: rgb(var(--now-pill--color, 16, 23, 26));
  /* Border inverts: 170,178,182 → 151,162,166 */
  border: var(--now-pill--border-width, 1px) solid
    rgb(var(--now-pill--border-color, 170, 178, 182));
  font-size: 0.875rem;
  transition:
    background-color 100ms linear,
    color 100ms linear;
}

.pill:hover {
  background-color: rgb(
    var(--now-pill--background-color--hover, 245, 246, 247)
  );
  /* Hover text: teal 0,86,110 → 77,166,191 */
  color: rgb(var(--now-pill--color--hover, 0, 86, 110));
}

.pill--selected {
  /* Selected: primary blue background */
  background-color: rgb(
    var(--now-pill--background-color--selected, 0, 128, 163)
  );
  /* Selected text: white */
  color: rgb(var(--now-pill--color--selected, 255, 255, 255));
  border-color: rgb(var(--now-pill--border-color--selected, 0, 128, 163));
}

/* Remove button within pill */
.pill__remove {
  background: none;
  border: none;
  color: currentColor; /* Inherits pill text color */
  cursor: pointer;
  padding: 0;
  font-size: 1rem;
  line-height: 1;
  opacity: 0.7;
}

.pill__remove:hover {
  opacity: 1;
}
```

**Token Values:**

| Token                                    | Light Mode  | Dark Mode   | Purpose               |
| ---------------------------------------- | ----------- | ----------- | --------------------- |
| `--now-pill--background-color`           | 255,255,255 | 5,8,9       | Base background       |
| `--now-pill--color`                      | 16,23,26    | 255,255,255 | Text color            |
| `--now-pill--border-color`               | 170,178,182 | 151,162,166 | Border                |
| `--now-pill--background-color--hover`    | 245,246,247 | 22,31,35    | Hover background      |
| `--now-pill--color--hover`               | 0,86,110    | 77,166,191  | Hover text (teal)     |
| `--now-pill--background-color--selected` | 0,128,163   | 0,128,163   | Selected (blue)       |
| `--now-pill--color--selected`            | 255,255,255 | 255,255,255 | Selected text (white) |

---

## Implementation Checklist

Before finalizing control implementations:

**Token Compliance:**

- [ ] All controls use appropriate category tokens (`actionable` or `form-control`)
- [ ] No hard-coded colors, spacing, or radii
- [ ] Fallback chains present and degrade to neutral tokens
- [ ] Colors wrapped with `rgb()`/`rgba()`

**State Implementation:**

- [ ] All states defined: base, hover, active, focus-visible, disabled
- [ ] Focus states use inset shadow or outline (no reflow)
- [ ] State changes are visually distinct
- [ ] Disabled states remove pointer interactions

**Accessibility:**

- [ ] Semantic HTML elements used
- [ ] Labels associated with inputs (`<label for>` or aria-label)
- [ ] Focus styles visible (WCAG 2.4.7)
- [ ] Contrast ratios meet AA (≥4.5:1)
- [ ] Keyboard navigation supported

**Consistency:**

- [ ] Spacing scale indices consistent across controls
- [ ] Border widths remain constant across states
- [ ] Motion limited to approved properties (100ms linear)
- [ ] Alias layer reused where applicable

---

## Common Mistakes (Claude 4 Watch Points)

❌ **Using `rgb()` for secondary buttons** - Loses alpha channel transparency  
✅ Use `rgba()` with separate alpha token

❌ **Mixing checkbox and radio tokens** - They use different variants  
✅ Checkboxes: `--primary`, Radio: `--secondary`

❌ **Inventing token names** like `--now-input--color`  
✅ Use `css_variable_explorer` to find actual tokens

❌ **Changing border width on focus** - Causes layout reflow  
✅ Use inset box-shadow for focus rings

❌ **Using unicode for dropdown icons** (▼)  
✅ Use inline SVG with `aria-hidden="true"`

❌ **Hardcoding `1rem` icon size as width/height**  
✅ Use `block-size`/`inline-size` with `1rem`

---

## Next Steps

For layout patterns and containers, see `UI_PAGE_THEMING_LAYOUT`.  
For navigation and feedback components, see `UI_PAGE_THEMING_COMPONENTS`.
