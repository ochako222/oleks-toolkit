---
name: frontend-vitest-rtl
description: >
  Generate Vitest + React Testing Library test files for React components.
  Use when the user wants to write tests, create a test file, add unit tests,
  or test a React component. Enforces the canonical VolumePriceMixer.test.tsx
  architecture: async tests, defaultProps, two-level describe, RTL queries,
  vi.fn() spies, controlled-component awareness, and jest-dom matchers.
---

# Frontend Vitest + RTL Component Tester

Generate complete, production-quality Vitest + React Testing Library test files for React components — following the exact architecture established in `src/components/VolumePriceMixer.test.tsx`.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Writing unit tests for a React component
- Creating a `*.test.tsx` file
- Adding tests to an existing component
- Scanning for untested components
- "Write tests for X", "Test this component", "Add unit tests"

**When in doubt about which test tool to use for a React component — always use this skill.**

## Critical Safety Rules

**NEVER overwrite an existing `*.test.tsx` file without explicit user confirmation.**

**NEVER modify the source component file** — this skill only creates/edits test files.

**NEVER assume a component manages its own state** — `VolumePriceMixer` taught us that controlled components call callbacks but don't update their own display. Always read the component first to understand whether it is controlled or stateful.

**NEVER test navigation in unit/component tests** — route transitions belong in E2E tests (Playwright). In unit tests, stub `useNavigate` with a plain `vi.fn()` only to prevent crashes, but never assert on it. Remove any `mockNavigate` spy and all assertions like `expect(mockNavigate).toHaveBeenCalledWith('/somewhere')`.

**Page components CAN have unit tests — but scope them tightly.** Mocking the router, store, API hooks, SVGs, and child components is acceptable boilerplate to make a page renderable. Write unit tests for: static labels, button visibility, click events that open modals or change local state. Do NOT write unit tests for: navigation flows, multi-step API sequences, or full user journeys — those belong in Playwright E2E.

Safe targets:
```
src/components/
├── ComponentName.tsx         ← READ ONLY — never touch
└── ComponentName.test.tsx    ← CREATE / EDIT here
```

## Architecture / Overview

| Tool | Role | Config |
|------|------|--------|
| Vitest | Test runner | `vite.config.ts` — `globals: true`, `environment: 'jsdom'` |
| `@testing-library/react` | Render + query | `render`, `screen`, `rerender` |
| `@testing-library/user-event` | User interactions | `userEvent.click()` |
| `@testing-library/jest-dom` | Extra matchers | `toBeVisible`, `toBeInTheDocument`, `toHaveClass` |

## Canonical Test File Pattern

**Every generated test file MUST follow this structure exactly — no exceptions:**

```tsx
import { render, screen } from '@testing-library/react';
import { userEvent } from '@testing-library/user-event';
import { describe, expect, it, vi } from 'vitest';
import { ComponentName } from './ComponentName';
import '@testing-library/jest-dom';

const defaultProps = {
  // ALL required props with realistic default values
};

describe('ComponentName', () => {
  describe('feature group name', () => {
    it('test description', async () => {
      render(<ComponentName {...defaultProps} />);

      expect(screen.getByText('...')).toBeVisible();
    });
  });
});
```

### Import block — always in this exact order

```tsx
import { render, screen } from '@testing-library/react';   // 1st
import { userEvent } from '@testing-library/user-event';   // 2nd
import { describe, expect, it, vi } from 'vitest';         // 3rd
import { ComponentName } from './ComponentName';            // 4th
import '@testing-library/jest-dom';                        // 5th — side-effect, always last
```

### Structure rules

- All `it()` functions are **`async`** — no exceptions
- `defaultProps` is a **module-level const** covering all required props with realistic values
- **Two-level `describe`**: outer = component name, inner = feature group (e.g. `'rendering'`, `'callbacks'`, `'edge cases'`)
- **One `render()` per `it()` block** — no shared state between tests
- Each test file lives **next to its component**: `ComponentName.tsx` → `ComponentName.test.tsx`

## RTL Query Decision Table

| What to find | Query to use |
|---|---|
| Visible text content | `screen.getByText('...')` |
| Input/textarea value | `screen.getByDisplayValue('...')` |
| Button or ARIA role | `screen.getByRole('button', { name: '...' })` |
| Element that must be **absent** | `screen.queryByRole(...)` or `screen.queryByText(...)` + `.not.toBeInTheDocument()` |
| CSS class on root element | `const { container } = render(...)` → `container.firstChild` + `.toHaveClass('...')` |
| Element present but needs `rerender` | `const { rerender } = render(...)` → `rerender(<C {...newProps} />)` |

**CRITICAL: Never use `getByX` to assert absence** — it throws when the element is not found, preventing the assertion from running. Always use `queryByX` for absence checks.

## Controlled Component Rule

**Never assume a component manages its own state.**

Before writing tests, read the component and determine:

- **Controlled** (receives state as props, fires callbacks) → test callbacks with `vi.fn()` spies; use `rerender` to simulate prop changes
- **Stateful** (manages own state internally) → test that clicking/typing updates the displayed values

### Callback testing pattern

```tsx
it('calls handleIncrease when + is clicked', async () => {
  const handleIncrease = vi.fn();
  render(<ComponentName {...defaultProps} handleIncrease={handleIncrease} />);

  await userEvent.click(screen.getByRole('button', { name: '+' }));
  expect(handleIncrease).toHaveBeenCalledOnce();
});
```

### Prop update testing pattern (controlled components)

```tsx
it('updates display when quantity prop changes', async () => {
  const { rerender } = render(<ComponentName {...defaultProps} />);

  expect(screen.getByText('5.00 € TTC')).toBeVisible();

  rerender(<ComponentName {...defaultProps} quantity={700} />);

  expect(screen.getByText('7.00 € TTC')).toBeVisible();
});
```

## Test Categories to Cover

For every component, analyse and generate tests in these categories (cover all that apply):

| Category | What to test | Example |
|---|---|---|
| **Rendering** | Text labels, static content, input values | `getByText('for')`, `getByDisplayValue('500 ml')` |
| **Conditional rendering** | Boolean props that show/hide elements | `hideControls={true}` → buttons absent |
| **Callbacks** | Every optional callback prop | `vi.fn()` + `toHaveBeenCalledOnce()` |
| **CSS variants** | Props that switch class names | `view="mixer"` → `toHaveClass('volume-price-mixer-container')` |
| **Edge cases** | Zero values, boundary values, optional props omitted | `quantity=0`, missing callback (no crash) |

> **Out of scope for unit tests:** Route transitions (`navigate('/scan')`), full page flows, URL changes. Test those in Playwright E2E specs instead.

## Operations

### Step 1 — Identify the target component

Ask the user which component to test. Also offer:

> "Or I can scan the project for components that have no test file yet — want me to do that instead?"

**If scan requested:**
```bash
# Find all component files with no sibling test file
ls src/components/*.tsx | grep -v test
```
Compare against existing `*.test.tsx` files — list untested components and let the user pick.

### Step 2 — Read and analyse the component

Read the component file and extract:
- Component name and export style (`export const` vs `export default`)
- All props (required and optional) and their types
- Conditional rendering branches (boolean flags, variant props)
- Callback props (`onClick`, `onChange`, `handleX`)
- CSS class variants (props that switch class names)
- Computed/derived output (calculations, formatted strings)
- Whether it is **controlled** or **stateful**

### Step 3 — Generate the test file

Write `src/components/ComponentName.test.tsx` following the canonical pattern. Cover all 5 categories from the table above.

### Step 4 — Run and fix

```bash
npx vitest run
```

Fix any failures before presenting the result to the user.

## Configuration / Key Locations

```
frontend/
├── vite.config.ts                    # globals: true, environment: 'jsdom'
├── src/components/
│   ├── VolumePriceMixer.tsx          # Reference component
│   ├── VolumePriceMixer.test.tsx     # Canonical test pattern — READ THIS
│   └── <YourComponent>.test.tsx     ← skill creates here
└── package.json                      # @testing-library/react, user-event, jest-dom, vitest
```

**Vitest config** (`vite.config.ts`):
```ts
test: {
  globals: true,        // describe/it/expect/vi available globally
  environment: 'jsdom', // browser-like DOM
}
```

Even though `globals: true` makes `describe`, `it`, `expect`, `vi` available globally, **always import them explicitly from `'vitest'`** — the project linter enforces explicit imports.

## Decision Framework

When generating tests for a component:

1. **Component has no callback props?** → Focus on rendering and conditional rendering tests
2. **Component has callback props?** → Add `vi.fn()` spy tests for each one
3. **Component switches CSS classes via a prop?** → Add `container.firstChild` + `toHaveClass` tests
4. **Component has numeric/computed output?** → Add edge case tests (zero, boundary, rounding)
5. **Component has optional props that could be undefined?** → Add a "does not crash when X is not provided" test
6. **Unsure if controlled or stateful?** → Read the component source — if `useState` is present it's stateful; if it only receives props it's controlled

## Example Requests

- "Write tests for the Border component" → Read `Border.tsx`, generate `Border.test.tsx`
- "Add unit tests to VolumePriceMixer" → Already has tests — ask before overwriting
- "Which components have no tests yet?" → Scan `src/components/*.tsx` for missing `*.test.tsx` siblings
- "Test the Buttons component" → Read `Buttons.tsx`, generate `Buttons.test.tsx`
- "Create a test file for ProductDetailsPage" → Read page file, generate test following this pattern
