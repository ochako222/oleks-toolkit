---
name: frontend-performance-optimizer
description: >
  Audit React components for useMemo, useCallback, and memo optimization opportunities.
  React Compiler-aware: checks compiler status first, explains what it already handles,
  then surfaces only the genuine remaining cases where manual memoization adds real value.
  Invoke when the user wants to reduce re-renders or optimize frontend performance.
---

# Frontend Performance Optimizer

Audit the frontend for legitimate `useMemo`, `useCallback`, and `memo` optimization opportunities. **This skill is React Compiler-aware** — it checks the compiler status first and only surfaces findings that the compiler cannot handle automatically.

---

## When to Invoke

Invoke this skill when the user:

- Wants to reduce unnecessary re-renders
- Asks about `useMemo`, `useCallback`, or `memo` usage
- Reports performance problems (sluggish UI, dropped frames)
- Asks for a performance audit of the frontend

---

## Step 1 — Check React Compiler Status

Before scanning anything, determine whether React Compiler is active:

```
grep -n "babel-plugin-react-compiler" frontend/vite.config.ts frontend/package.json
```

**If compiler is active** (found in `vite.config.ts` plugins):
- Set `compilerActive = true`
- The compiler automatically memoizes values, functions, and component outputs inside every component and hook
- Most manual `useMemo`, `useCallback`, `memo` additions are **redundant**
- Only the edge cases in Step 3 are worth investigating

**If compiler is NOT active**:
- Set `compilerActive = false`
- Full manual memoization audit applies — scan every component for re-render patterns

Output the compiler status at the top of the Performance Audit Block (Step 4).

---

## Step 2 — Read the Performance Patterns Template

Read `.claude/templates/frontend-performance-memo-patterns.md` in full. This is the authoritative reference for:
- What the compiler handles automatically (do not flag these)
- The 5 genuine edge cases where manual memoization still adds value
- Anti-patterns to avoid
- The decision table

Use this as your ruleset for the scan.

---

## Step 3 — Scan for Genuine Edge Cases

Use Glob and Grep to scan `frontend/src/`. Focus on the 5 edge cases from the template. Scan only what is necessary — do not read every file.

### 3a — Standalone utility functions called in components (Edge Case 1)

Look for functions defined outside components that perform heavy work and are called during render:

```
grep -rn "\.sort\|\.filter\|\.reduce\|\.map\|\.flatMap" frontend/src/ --include="*.tsx" --include="*.ts"
```

Flag only when:
- The function is defined outside the component body
- The computation involves non-trivial iteration (large arrays, nested loops)
- `console.time` measurement would likely show ≥ 1ms

**Do NOT flag**: Simple `.filter`, `.map`, `.reduce` on small arrays — these are trivially fast.

### 3b — Expensive computations shared across multiple components (Edge Case 2)

Look for the same expensive computation called independently in multiple components:

```
grep -rn "sort\|filter\|reduce" frontend/src/app/components/ --include="*.tsx" -l
```

Flag when: The same expensive derivation appears in 3+ separate component files with no shared parent passing the result down.

### 3c — Stable references for non-React APIs (Edge Case 3)

Look for `useEffect` that sets up external listeners, SDKs, or timers with inline function args:

```
grep -rn "addEventListener\|addListener\|subscribe\|observe\|setInterval\|setTimeout" frontend/src/ --include="*.tsx"
```

Flag when: A function reference is passed to an external API inside `useEffect` without being wrapped in `useCallback`.

### 3d — useEffect with object/array dependencies (Edge Case 5)

Look for `useEffect` dependencies that are objects or arrays created inline:

```
grep -rn "useEffect" frontend/src/ --include="*.tsx" -A 5
```

Flag when: The dependency array contains an identifier pointing to an object or array constructed in the component body, not a primitive.

### 3e — Components with memo opportunities (Edge Case 4)

Look for components that:
- Receive many props (> 5)
- Are rendered inside lists or frequently re-rendering parents
- Contain expensive rendering logic (canvas, SVG, complex DOM trees)

```
grep -rn "\.map(" frontend/src/app/ --include="*.tsx" -B 2 -A 3
```

Flag when: A component rendered inside `.map()` is **not** already wrapped in `memo` AND the parent re-renders frequently (e.g., on every keystroke, scroll, or interval).

---

## Step 4 — Output the Performance Audit Block

**Always** show this block before writing any code or making any changes.

---

### Format A — Compiler Active, Genuine Findings Exist

```
## Performance Audit Report

**React Compiler status:** ACTIVE (babel-plugin-react-compiler in vite.config.ts)

**What the compiler already handles (no action needed):**
- All inline objects and arrays passed as props
- All event handlers defined in component bodies
- All computed values derived from props/state within components
- Trivial computations (< 1ms)

**Genuine optimization opportunities found:**

| # | File | Location | Edge Case | Recommendation |
|---|------|----------|-----------|----------------|
| 1 | `path/to/file.tsx` | line 42 | Expensive computation shared across 3 components | Lift to parent + useMemo |
| 2 | `path/to/file.tsx` | line 87 | useEffect dep is inline object | Wrap with useMemo |
| 3 | `path/to/list.tsx` | line 23 | Component in .map() renders complex SVG | Add memo() |

**Anti-patterns found (redundant manual memoization):**

| File | Location | Issue |
|------|----------|-------|
| `path/to/file.tsx` | line 12 | useMemo on trivial `items.length` — remove |
| `path/to/file.tsx` | line 34 | useCallback with no dependencies on a simple handler — remove |

**Verdict:** [N] genuine optimizations + [M] redundant memoizations to remove.
```

---

### Format B — Compiler Active, No Genuine Findings

```
## Performance Audit Report

**React Compiler status:** ACTIVE (babel-plugin-react-compiler in vite.config.ts)

**What the compiler already handles (no action needed):**
- All inline objects and arrays passed as props
- All event handlers defined in component bodies
- All computed values derived from props/state within components

**Findings:** No genuine optimization opportunities detected.

The compiler is already handling memoization for the scanned components.
Manual useMemo/useCallback/memo additions would be redundant here.

**Recommendation:** If a specific component feels slow, profile it in React DevTools first,
then invoke this skill again with the specific component path for a targeted audit.
```

---

### Format C — Compiler NOT Active, Full Audit

```
## Performance Audit Report

**React Compiler status:** NOT ACTIVE

**Findings — useMemo opportunities:**

| File | Location | Computation | Why memoize |
|------|----------|-------------|-------------|
| `path/to/file.tsx` | line 42 | `products.filter(...)` called every render | Array depends on `products` prop, expensive on large lists |

**Findings — useCallback opportunities:**

| File | Location | Handler | Why memoize |
|------|----------|---------|-------------|
| `path/to/file.tsx` | line 87 | `handleSubmit` passed to memoized child | Child wrapped in memo but handler recreated every render |

**Findings — memo() opportunities:**

| File | Location | Component | Why wrap |
|------|----------|-----------|----------|
| `path/to/list.tsx` | line 23 | `<ProductRow>` inside `.map()` | Renders on every parent re-render even when row data unchanged |

**Verdict:** [N] findings requiring attention.
```

---

## Step 5 — Apply Changes

After the user reviews the audit block and approves, apply only the confirmed findings:

1. **Adding `useMemo`**: Follow Edge Case 1 pattern from `.claude/templates/frontend-performance-memo-patterns.md`
2. **Adding `useCallback`**: Follow Edge Case 3 pattern
3. **Adding `memo()`**: Follow Edge Case 4 pattern
4. **Removing redundant memoization**: Delete the `useMemo`/`useCallback`/`memo` wrapper, keep the inner value/function

**Never add memoization** where the compiler already handles it — this adds complexity and noise without any performance benefit.

---

## Step 6 — Verification Guidance

After changes are applied, tell the user how to verify:

```
## How to Verify the Optimization

1. Open React DevTools → Profiler tab
2. Click "Record" → Perform the action that was slow
3. Stop recording → inspect the flamegraph
4. Look for:
   - Components no longer highlighted (memo working)
   - Shorter render bars (useMemo working)
   - Fewer render counts per interaction

Biome doesn't check for memoization — verification is manual via the Profiler.
```

---

## Rules

1. **Always check compiler status first** — never recommend manual memoization without knowing if the compiler is active
2. **Never flag what the compiler already handles** — trivial computations, inline handlers, inline objects are all covered
3. **Show the audit block before making any changes** — findings are proposals until the user approves
4. **Measure before memoizing** — if there's no profiler evidence of a problem, recommend profiling first rather than preemptive memoization
5. **Remove redundant memoization when found** — if the codebase already has `useMemo`/`useCallback` on trivial things, flag them for removal
6. **One finding = one justification** — every item in the audit must cite which edge case it maps to
7. **Follow Biome formatting** — tabs, single quotes, semicolons, no explicit `any` where avoidable
8. **Import from React** — `import { useMemo, useCallback, memo } from 'react';`

---

## Templates

| Template | Location |
|----------|----------|
| `frontend-performance-memo-patterns` | `.claude/templates/frontend-performance-memo-patterns.md` |
