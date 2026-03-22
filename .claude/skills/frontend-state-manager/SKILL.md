---
name:frontend-state-manager
description: >
  Creates React state management solutions — React Context, Zustand stores, or Redux Toolkit slices.
  Recommends the best approach based on scope, complexity, and usage patterns.
  Triggers: state, context, store, zustand, redux, slice, provider, global state, shared state,
  createContext, useContext, state management.
---

# Frontend State Manager

Generates type-safe state management code for the React frontend, choosing between React Context, Zustand, or Redux Toolkit based on the use case.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating a new React Context with Provider and custom hook
- Adding a Zustand store for shared UI or feature state
- Creating a new Redux Toolkit slice with actions and selectors
- Asking "should I use Context, Zustand, or Redux for X?"
- Adding global or shared state to the frontend
- Creating a Provider component for dependency injection

**If the user mentions state, context, store, slice, or provider — use this skill.**

## Critical Safety Rules

**NEVER modify the existing Redux store configuration (`frontend/src/app/store/index.ts`) without confirming with the user first.**

Changing the store affects the entire application. When adding a new Redux slice, the skill MUST show the user the exact changes to `combineReducers` before applying them.

**NEVER remove or rename existing slices, contexts, or stores.**

**ALWAYS check if a similar state solution already exists** before creating a new one — search for existing contexts in `frontend/src/app/contexts/`, stores in `frontend/src/app/stores/`, and slices in `frontend/src/app/store/`.

## Architecture / Overview

The SLA-Admin frontend supports three state management approaches, each suited to different scopes:

| Approach | Scope | Provider Needed? | Location | Best For |
|----------|-------|-------------------|----------|----------|
| **React Context** | Subtree | Yes — wrap consumers | `frontend/src/app/contexts/` | Small, rarely-changing state shared by a subtree (theme, locale, modal state, feature flags) |
| **Zustand** | Cross-component | No — import and use | `frontend/src/app/stores/` | Mid-scale shared state across distant components (UI state, filters, form wizards, feature-specific state) |
| **React Redux (RTK)** | App-wide | Yes — already at root | `frontend/src/app/store/` | Full-scale app state with middleware, devtools, API interceptor integration (auth, entities) |

### Existing State in This Project

| Slice/Context | Type | Location |
|---------------|------|----------|
| `authSlice` | Redux | `frontend/src/app/store/authSlice.ts` |
| `machineSlice` | Redux | `frontend/src/app/store/machineSlice.ts` |

Redux Provider is already configured at the root in `frontend/src/main.tsx`.

## Operations

### Operation 1: Recommend an Approach

Before generating code, analyze the user's requirements and recommend the best approach.

**Ask the user:**

1. What data/state do you need to manage?
2. How many components need access to this state?
3. Does the state change frequently (e.g., every keystroke) or rarely (e.g., on login)?
4. Does it need to persist across page reloads?
5. Does it need to integrate with API interceptors or middleware?

**Decision Framework:**

1. **State is small, changes rarely, used by a subtree of components?** → React Context
   - Examples: theme toggle, locale, modal open/close, feature flags, form wizard step
2. **State is shared across distant components, moderate complexity, no middleware needed?** → Zustand
   - Examples: filter selections, sidebar state, notification queue, multi-step form data, table view preferences
3. **State is app-wide, needs middleware/devtools, or integrates with API layer?** → React Redux (RTK)
   - Examples: auth tokens, entity caches, multi-feature shared data, anything needing time-travel debugging
4. **Unsure?** → Start with React Context. Migrate to Zustand or Redux later if complexity grows.

**Present the recommendation with reasoning, then let the user confirm or override.**

### Operation 2: Generate React Context

When the user needs a React Context, generate code following the template.

**Steps:**

1. Read the `frontend-react-context` template from `.claude/templates/frontend-react-context.md`
2. Create the context file at `frontend/src/app/contexts/<ContextName>Context.tsx`
3. Export: the Context object, the Provider component, and the custom hook
4. Inform the user where to add the Provider in the component tree

**File naming:** `<ContextName>Context.tsx` (PascalCase name + "Context" suffix)

**Pattern summary (from AnySkin reference):**

- Define TypeScript interface for context shape
- Create default value with no-op functions
- Create context with `createContext<Type>(defaultValue)`
- Provider uses `useState` + `useCallback` + `useMemo` for stable references
- Custom hook checks for usage outside Provider (warn or throw)

### Operation 3: Generate Zustand Store

When the user needs a Zustand store, generate code following the template.

**Steps:**

1. Check if `zustand` is in `frontend/package.json` dependencies
2. If not installed, tell the user to run: `cd frontend && npm install zustand`
3. Read the `frontend-zustand-store` template from `.claude/templates/frontend-zustand-store.md`
4. Create the store file at `frontend/src/app/stores/<storeName>Store.ts`
5. Export: the store hook and any selector functions

**File naming:** `<storeName>Store.ts` (camelCase name + "Store" suffix)

### Operation 4: Generate Redux Toolkit Slice

When the user needs a Redux slice, generate code following the template.

**Steps:**

1. Read the `frontend-redux-slice` template from `.claude/templates/frontend-redux-slice.md`
2. Create the slice file at `frontend/src/app/store/<sliceName>Slice.ts`
3. Register the new reducer in `frontend/src/app/store/index.ts` — add to `combineReducers` and import
4. Check if typed hooks (`useAppDispatch`, `useAppSelector`) exist in `frontend/src/app/hooks/`
   - If missing, create them following the project's typing pattern (`AppDispatch`, `RootState`)
5. Export: actions, selectors, and the reducer

**File naming:** `<sliceName>Slice.ts` (camelCase name + "Slice" suffix)

**Existing store structure to match:**

- Store: `frontend/src/app/store/index.ts` — `configureStore` + `combineReducers`, exports `RootState`, `AppStore`, `AppDispatch`
- Slices use `createSlice` with typed `PayloadAction`
- Selectors are co-located in the slice file (e.g., `selectMachineDetails`)
- Components use `useSelector` and `useDispatch` (or typed equivalents)

## Configuration / Key Locations

```
frontend/src/
├── app/
│   ├── contexts/           # React Context files (NEW — created by this skill)
│   │   └── <Name>Context.tsx
│   ├── store/              # Redux Toolkit
│   │   ├── index.ts        # Store config, combineReducers, RootState
│   │   ├── authSlice.ts    # Auth state (DO NOT MODIFY)
│   │   └── machineSlice.ts # Machine state (DO NOT MODIFY)
│   ├── stores/             # Zustand stores (NEW — created by this skill)
│   │   └── <name>Store.ts
│   └── hooks/              # Custom hooks
│       └── useMediaQuery.ts
├── main.tsx                # Redux Provider wraps app here
└── api/
    └── baseRequest.ts      # Uses store.getState() and store.dispatch() directly
```

**Code style conventions:**

- Tabs for indentation
- Single quotes for strings
- Semicolons required
- Path alias: `src/` → `./src`
- React 19.2 with React Compiler (automatic memoization)
- Biome linter/formatter

## Templates

This skill references 3 templates for code generation. **ALL code patterns come from templates, not inline.**

| Template | Path | Used For |
|----------|------|----------|
| `frontend-react-context` | `.claude/templates/frontend-react-context.md` | Context + Provider + custom hook |
| `frontend-zustand-store` | `.claude/templates/frontend-zustand-store.md` | Zustand store with typed state and actions |
| `frontend-redux-slice` | `.claude/templates/frontend-redux-slice.md` | Redux Toolkit slice following project patterns |

## Troubleshooting

**Context value not updating:**

- Ensure the Provider is above the consuming components in the tree
- Check that `useMemo` wraps the context value object
- Verify `useCallback` wraps setter functions for stable references

**Zustand store not found:**

- Verify `zustand` is installed: check `frontend/package.json`
- Ensure the store file exports a named hook (not default export)

**Redux slice not in store:**

- Confirm the new reducer is added to `combineReducers` in `frontend/src/app/store/index.ts`
- Confirm the import path is correct
- Restart the dev server if HMR doesn't pick up store changes

**TypeScript errors with useDispatch/useSelector:**

- Use typed hooks `useAppDispatch` and `useAppSelector` if available
- Or cast: `useDispatch<AppDispatch>()` and `useSelector((state: RootState) => ...)`

## Decision Framework

When the user requests state management:

1. **Only a few components in the same subtree need it?** → React Context
2. **Multiple distant components, but no middleware/devtools needed?** → Zustand
3. **App-wide state, middleware, devtools, or API interceptor integration?** → React Redux (RTK)
4. **Unsure or prototyping?** → Start with React Context, migrate later if needed

## Example Requests

- "Create a theme context for dark/light mode" → Recommend React Context, generate context + provider + hook
- "I need a store for managing filter selections across pages" → Recommend Zustand, generate store
- "Add a Redux slice for notifications" → Generate RTK slice, register in store
- "Should I use Context or Redux for sidebar state?" → Analyze and recommend (likely Zustand or Context)
- "Create state management for a multi-step form wizard" → Recommend Zustand, generate store with step tracking
- "Add global state for the current user's preferences" → Analyze scope, likely Redux if app-wide
