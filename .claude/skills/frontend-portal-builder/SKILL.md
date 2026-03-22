---
name: frontend-portal-builder
description: >
  Enforce React portal usage for all modals, tooltips, and toast/notification UI.
  Automatically triggered when creating or refactoring any overlay component.
  Shows a portal decision block before writing code and guides setup of the
  centralized portal infrastructure (BasePortal, ToastProvider, portal-root).
---

# Frontend Portal Builder

**AUTOMATICALLY INVOKE THIS SKILL** whenever the task involves creating or refactoring any modal, tooltip, dropdown overlay, or toast/notification component. Show the portal decision block to the user before writing any overlay code.

---

## When to Invoke

Trigger this skill for any task that includes:

- Creating a new modal, dialog, or confirmation overlay
- Creating a tooltip, popover, or dropdown
- Creating a toast, notification, or alert banner component
- Refactoring existing modal/tooltip/notification components
- Any UI that must escape CSS stacking contexts or `overflow: hidden` containers

---

## Step 1 — Assess Portal Infrastructure

Before writing any component, check whether the portal infrastructure already exists in the project:

1. **`#portal-root` in `index.html`** — `grep -n "portal-root" frontend/index.html`
2. **`BasePortal` component** — `frontend/src/app/components/BaseComponents/BasePortal.tsx`
3. **`ToastProvider` context** — `frontend/src/app/context/ToastContext.tsx`
4. **`ToastProvider` in `main.tsx`** — check if `<ToastProvider>` wraps `<App />`

Record what exists and what is missing. If anything is missing, add "Setup required" to the decision block output.

---

## Step 2 — Output the Portal Decision Block

**Always** show this block before writing code. Choose the format that matches the task.

---

### Format A — Custom Portal Component Required

```
## Portal Decision: Custom Portal Component

**Decision:** Build with `createPortal` via `BasePortal`.

**Why:**
- [Specific reason — e.g. "This tooltip sits inside a table cell with overflow: hidden — Ant Design Tooltip
  would be clipped. A portal-based tooltip renders into #portal-root, escaping the constraint."]

**What will be done:**
- [e.g. "Create PortalTooltip in src/app/components/BaseComponents/PortalTooltip.tsx"]
- [e.g. "Uses BasePortal as the mounting primitive"]
- [e.g. "Positioned with useLayoutEffect + getBoundingClientRect"]

**Infrastructure needed:** [List any missing items from Step 1 — e.g. "#portal-root missing from index.html"]

**Template used:** frontend-portal-pattern.md — Section 5 (Portal Tooltip)
```

---

### Format B — Ant Design Built-in Is Sufficient

```
## Portal Decision: Ant Design Built-in (Already Uses Portals)

**Decision:** Use Ant Design's `Modal` / `Tooltip` / `notification` API.

**Why:**
- [Specific reason — e.g. "This is a standard confirmation dialog. Ant Design Modal already renders via
  an internal portal to document.body — no custom portal setup needed."]
- [e.g. "No stacking context conflict exists at the modal's call site."]

**What will be done:**
- [e.g. "Create AddMachineModal using Ant Design Modal + Form, matching the pattern in AddBottleModal.tsx"]

**Infrastructure needed:** None — Ant Design handles portal mounting internally.

**Template used:** frontend-portal-pattern.md — Section 7 (Decision Table)
```

---

### Format C — Toast / Notification (Global Portal System)

```
## Portal Decision: Global Toast System via Portal

**Decision:** Use the centralized `ToastProvider` + `useToast` hook.

**Why:**
- [Specific reason — e.g. "This success notification must be triggerable from any component in the tree,
  including deeply nested API callbacks. A portal-based context provides a single mount point."]

**What will be done:**
- [e.g. "Add ToastProvider to main.tsx wrapping <App />"]
- [e.g. "Use useToast() hook at the call site: showToast('Saved', 'success')"]

**Infrastructure needed:** [List any missing items — e.g. "ToastProvider not yet in main.tsx"]

**Template used:** frontend-portal-pattern.md — Section 4 (Toast System)
```

---

## Step 3 — Set Up Missing Infrastructure

If Step 1 found missing infrastructure, set it up **before** writing the component:

### Add `#portal-root` to `index.html`

Find `frontend/index.html`. Add immediately after the `<div id="root">` line:

```html
<div id="portal-root"></div>
```

### Create `BasePortal`

Follow the code in `.claude/templates/frontend-portal-pattern.md` — Section 2.

Place at: `frontend/src/app/components/BaseComponents/BasePortal.tsx`

### Create `ToastProvider`

Follow the code in `.claude/templates/frontend-portal-pattern.md` — Section 4.

Place at: `frontend/src/app/context/ToastContext.tsx`

Register in `frontend/src/main.tsx` following Section 6.

---

## Step 4 — Build the Component

Write the component following the appropriate pattern from the template:

| Component type | Template section |
|---------------|-----------------|
| `BasePortal` primitive | Section 2 |
| Custom modal/dialog | Section 3 |
| Toast / notification system | Section 4 |
| Tooltip / popover | Section 5 |

**Code rules (enforced — no exceptions):**

- **Always mount via `BasePortal`** — never call `createPortal(children, document.body)` directly
- **No inline `style={{}}`** — all portal styles go in dedicated SCSS files under `src/app/styles/`
- **Colors from `_variables.scss`** — never hardcode hex values
- **Accessibility required:**
  - Custom modals: `role="dialog"` + `aria-modal="true"` + Escape key handler + body scroll lock
  - Tooltips: `role="tooltip"` on the tooltip div
  - Toast containers: `role="status"` + `aria-live="polite"`
- **Stop event propagation** on modal/overlay content to prevent click-through to the backdrop handler
- **Use `useLayoutEffect`** (not `useEffect`) for tooltip position calculations — avoids flash of mispositioned element

---

## Step 5 — SCSS Files

When creating portal SCSS, invoke the `frontend-scss-writer` skill to write the styles. Portal SCSS files follow the same rules as all other SCSS:

- New file per portal component: `_portal-modal.scss`, `_portal-toast.scss`, `_portal-tooltip.scss`
- Import in `styles.scss`
- All colors via CSS variables from `_variables.scss`
- No magic numbers — use spacing variables

---

## Rules

1. **Never skip the portal decision block.** Every modal/tooltip/toast task gets a block — even if Ant Design built-ins are used.
2. **Show the decision block before writing code.** Infrastructure setup and component writing come after.
3. **Always check infrastructure first.** Don't assume `#portal-root` or `BasePortal` exist.
4. **Ant Design is acceptable** for standard overlay patterns it already handles well — do not force a custom portal when the built-in is sufficient.
5. **Custom portals are required** when: element is inside `overflow: hidden`, complex animation, or the built-in Ant Design API cannot satisfy the design.
6. **`useToast` for all app-wide notifications** — never use `message.success()` / `notification.open()` Ant Design globals; use the portal-based `ToastProvider` instead.
7. **One `#portal-root`** — there must be exactly one portal mount point in the DOM.
8. **Invoke `frontend-scss-writer`** for all portal SCSS — do not write styles inline.

---

## Templates

| Template | Location |
|----------|----------|
| `frontend-portal-pattern` | `.claude/templates/frontend-portal-pattern.md` |
