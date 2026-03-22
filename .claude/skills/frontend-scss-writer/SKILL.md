---
name: frontend-scss-writer
description: >
  Write and audit SCSS for this project. Enforces the minimalist, non-redundant style rules
  established in _product-cards.scss. Use when adding new CSS classes, reviewing inline styles,
  or refactoring existing SCSS.
---

# Frontend SCSS Writer

Guidelines for writing SCSS in this project. The goal is **maximum reuse, minimum code**.

## Project SCSS Files

- `frontend/src/app/styles/_product-cards.scss` — all product-related component styles (cards, upload areas, form elements)
- `frontend/src/app/styles/_variables.scss` — breakpoints, colors, touch targets
- `frontend/src/app/styles/_layout.scss`, `_base.scss`, `_machines.scss`, `_login.scss`, `_logo.scss`
- `frontend/src/app/styles/styles.scss` — root import

## Rules Before Writing Any New Class

### 1. Check for duplicates first
Read the target SCSS file. If an existing class already does what you need:
- **Same properties** → reuse it, do not create a new class.
- **Same properties + one extra** → add the extra property to the existing class only if it won't break other usages. Otherwise create a new class that extends logically.

### 2. Identical content = one class
Never create two classes with the same CSS properties. Use a single class and apply it everywhere it's needed.

```scss
// WRONG
.product-field-group  { margin-bottom: 1rem; }
.product-form-row     { margin-bottom: 1rem; }
.product-upload-btn   { margin-bottom: 1rem; }

// RIGHT
.product-form-row { margin-bottom: 1rem; } // used on divs, rows, and buttons alike
```

### 3. Color-only variants → use IDs (project pattern)
When two elements share all properties except color, write one class for the shared properties and separate IDs for the color. This matches the existing `#basic` / `#booster` pattern.

- **ID names must be descriptive** — include the element context and the word `color`, e.g. `#required-badge-color`, not just `#required`.
- **Single color rule per ID** — each ID gets its own color. Only combine IDs under one rule if they truly share the same color value.

```scss
// WRONG — two separate classes with duplicated shared properties
.product-ingredient-required { font-size: 0.75rem; font-weight: normal; color: variables.$danger-color; }
.product-ingredient-optional { font-size: 0.75rem; font-weight: normal; color: variables.$hint-text-color; }

// WRONG — vague ID names
#required { color: variables.$danger-color; }
#optional { color: variables.$hint-text-color; }

// RIGHT — shared class + descriptive IDs with their own colors
.product-ingredient-badge { font-size: 0.75rem; font-weight: normal; }

#required-badge-color { color: variables.$danger-color; }
#optional-badge-color { color: variables.$hint-text-color; }
```

In components: `<span className="product-ingredient-badge" id="required-badge-color">`.

### 4. Extend existing classes before creating new ones
Before writing any new class, ask:
- Does `.product-card-type` (font-size: 0.75rem + responsive) already cover this?
- Does `.product-card-details` (height: 100%) already handle the card height?
- Does `.product-upload-hint` (nested in `.product-upload-area`) cover secondary text in upload areas?

If yes, reuse or add a small modifier rather than duplicating.

### 5. Always use variables for colors — never raw hex or rgba values
Every color must live in `_variables.scss`. If the variable doesn't exist yet, add it there first, then use it.

```scss
// WRONG
color: #8c8c8c;
color: rgba(0, 0, 0, 0.45);  // same color, different format — still wrong
color: #ff4d4f;
color: #1890ff;

// RIGHT
color: variables.$hint-text-color;    // #8c8c8c — muted/hint text
color: variables.$danger-color;       // #ff4d4f — errors, required fields
color: variables.$basic-product-color; // #1890ff — primary blue
```

**Available color variables** (check `_variables.scss` before adding new ones):
```scss
$primary-text-color    // #000000
$secondary-text-color  // #969697
$hint-text-color       // #8c8c8c — muted labels, optional text
$danger-color          // #ff4d4f — errors, required badges
$basic-product-color   // #1890ff — primary blue
$booster-product-color // #800080 — purple
$primary-background    // #ffffff
$header-background     // #000000
```

**Responsive breakpoints via mixin:**
```scss
@include variables.respond-to('tablet')       // min-width
@include variables.respond-to-max('tablet')   // max-width
// Breakpoints: 'mobile', 'tablet', 'laptop', 'desktop'
```

### 6. Nested selectors for component-scoped styles
When styles only apply inside a specific component container, nest them:

```scss
// RIGHT — scoped, no global leak
.product-upload-area {
  border: 1px dashed #d9d9d9;  // structural color, no variable needed

  .product-upload-icon { font-size: 3rem; color: variables.$basic-product-color; }
  .product-upload-hint { color: variables.$hint-text-color; }
}

// WRONG — creates unnecessary global classes
.product-upload-area { border: 1px dashed #d9d9d9; }
.product-upload-icon  { font-size: 3rem; color: variables.$basic-product-color; }
.product-upload-hint  { color: variables.$hint-text-color; }
```

### 7. No class for a single inline style that's truly unique
If a style appears once and won't be reused, an inline `style={{}}` is acceptable. Only extract to SCSS when:
- The same style appears 2+ times, OR
- The style involves responsive breakpoints (which inline styles can't do), OR
- The style belongs to a logical group with other extracted styles

## Refactoring Inline Styles Checklist

When moving inline styles to SCSS:

1. **Audit all components** for `style={{...}}` — list every unique value
2. **Group by content** — identical declarations → one class
3. **Group by variance** — same properties, one field differs → class + ID
4. **Check existing classes** — can any existing class be reused as-is?
5. **Write SCSS** — add to `_product-cards.scss` at the bottom under a section comment
6. **Replace in components** — swap `style={{...}}` for `className` / `id`
7. **Verify** — `grep -rn 'style={{' src/app/components/Products/` should return nothing
