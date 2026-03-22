---
name: frontend-performance-memo-patterns
description: >
  Code patterns for React performance optimization with useMemo, useCallback, and memo.
  Covers React Compiler interaction, what it handles automatically, and the specific
  edge cases where manual memoization is still justified. Used by frontend-performance-optimizer.
type: template
---

# Frontend Performance Memo Patterns

> **Critical context for this project**: `babel-plugin-react-compiler` is active (`vite.config.ts`). The React Compiler automatically memoizes values, functions, and component outputs inside every component and hook. In most cases, adding `useMemo`, `useCallback`, or `memo` manually is **redundant** — the compiler already did it. Read the edge cases below carefully before adding any manual memoization.

---

## What React Compiler Handles Automatically

Inside any React component or hook, the compiler automatically memoizes:

```tsx
// ✅ Compiler handles — NO manual useMemo needed
function ProductList({ products, filter }) {
  // Object created in render — compiler stabilizes it
  const options = { matchMode: 'whole-word', text: filter };

  // Array derived from props — compiler caches the result
  const filtered = products.filter(p => p.category === filter);

  // Event handler defined in body — compiler stabilizes the reference
  const handleClick = (id: string) => setSelected(id);

  // Computed value — compiler caches between renders
  const total = filtered.reduce((sum, p) => sum + p.price, 0);

  return <List items={filtered} onSelect={handleClick} />;
}
```

All of the above would historically need `useMemo`, `useCallback`, or `memo`. With the compiler, they don't.

---

## Genuine Edge Cases for Manual Memoization

### Edge Case 1 — Computation Outside a Component or Hook

The compiler only optimizes code **inside** components and hooks. Standalone utility functions are **not** touched.

```tsx
// ❌ NOT optimized by compiler — it's a standalone function
function expensiveSort(items: ProductData[]): ProductData[] {
  return [...items].sort((a, b) => heavyComparison(a, b));
}

// ✅ If this function is called inside a component AND is expensive,
// memoize the CALL SITE inside the component:
function ProductTable({ items }: { items: ProductData[] }) {
  // This call is inside a component → compiler can cache it
  const sorted = expensiveSort(items);
  return <Table rows={sorted} />;
}
// The compiler will cache `sorted` — no manual useMemo needed here either.
// But if the compiler cannot detect the function is pure, add useMemo manually:
function ProductTable({ items }: { items: ProductData[] }) {
  const sorted = useMemo(() => expensiveSort(items), [items]);
  return <Table rows={sorted} />;
}
```

**Rule**: Add `useMemo` only when `console.time` shows ≥ 1ms and the compiler hasn't already stabilized it (verify with React DevTools Profiler).

---

### Edge Case 2 — Computation Shared Across Multiple Components

The compiler does **not** share memoized results between different component instances. If an expensive computation is used in 3+ components, extract it to a shared store or compute it once at a higher level.

```tsx
// ❌ Each component instance runs expensiveProcessing independently
function ComponentA({ data }) {
  const result = expensiveProcessing(data); // compiler caches within A
  return <div>{result}</div>;
}
function ComponentB({ data }) {
  const result = expensiveProcessing(data); // compiler caches within B — separate call
  return <div>{result}</div>;
}

// ✅ Compute once, pass down as prop — or use Redux selector with memoization
function Parent({ data }) {
  const result = useMemo(() => expensiveProcessing(data), [data]);
  return (
    <>
      <ComponentA result={result} />
      <ComponentB result={result} />
    </>
  );
}
```

---

### Edge Case 3 — Stable Reference Required by Non-React APIs

Event listeners, WebSockets, `IntersectionObserver`, `setTimeout` callbacks, and third-party SDKs require stable function references that survive across renders. The compiler stabilizes references within React's render cycle, but if you need a truly stable ref for external APIs, use `useCallback` explicitly.

```tsx
// ✅ Explicit useCallback — needed when an external API stores the reference
function MapComponent({ onMarkerClick }: { onMarkerClick: (id: string) => void }) {
  const stableHandler = useCallback(
    (markerId: string) => onMarkerClick(markerId),
    [onMarkerClick],
  );

  useEffect(() => {
    const map = initExternalMapSDK();
    map.addClickHandler(stableHandler); // SDK stores this reference
    return () => map.removeClickHandler(stableHandler);
  }, [stableHandler]);

  return <div id="map" />;
}
```

---

### Edge Case 4 — memo() for Components Receiving Unstable Props from Outside the Compiler's Scope

The compiler wraps each component's output, but if a parent is **outside** the compiler's reach (e.g., a third-party component calling your component), `memo` provides a safety net.

```tsx
// ✅ Acceptable use of memo — component is called from a non-compiled context
// or the profiler shows it re-renders unnecessarily with identical props
export const HeavyChart = memo(function HeavyChart({
  data,
  width,
  height,
}: HeavyChartProps) {
  // expensive canvas rendering
  return <canvas ref={canvasRef} width={width} height={height} />;
});
```

**When to apply**: Only after React DevTools Profiler confirms the component re-renders with identical props and the render is measurably slow (≥ 16ms = dropped frame).

---

### Edge Case 5 — useMemo for useEffect / Other Hook Dependencies

If an object or array is created inside the component body and used as a dependency of `useEffect`, the compiler may not always stabilize it enough. Use `useMemo` to guarantee stability.

```tsx
// Potentially re-triggers effect on every render without useMemo
function ConnectionManager({ host, port }: ConnectionProps) {
  // ⚠️ Even with compiler, if useEffect can't verify stability, explicitly memoize
  const config = useMemo(
    () => ({ host, port, timeout: 5000 }),
    [host, port],
  );

  useEffect(() => {
    const conn = createConnection(config);
    conn.connect();
    return () => conn.disconnect();
  }, [config]); // stable reference prevents reconnect on every render

  return null;
}
```

---

## Anti-Patterns to Avoid

```tsx
// ❌ Redundant with React Compiler — removes readability, no performance gain
function Component({ items }: { items: string[] }) {
  const count = useMemo(() => items.length, [items]); // trivial computation
  const handleClick = useCallback(() => console.log('clicked'), []); // no deps, already stable
  return <button onClick={handleClick}>{count}</button>;
}

// ✅ Just write naturally — compiler handles it
function Component({ items }: { items: string[] }) {
  const count = items.length;
  const handleClick = () => console.log('clicked');
  return <button onClick={handleClick}>{count}</button>;
}
```

```tsx
// ❌ memo on a component that re-renders for the right reason — adds noise
const SimpleLabel = memo(({ text }: { text: string }) => <span>{text}</span>);

// ✅ Just write it — it's cheap to render
const SimpleLabel = ({ text }: { text: string }) => <span>{text}</span>;
```

---

## How to Measure Before Optimizing

Always measure FIRST. Add memoization only where profiler evidence exists.

```tsx
// Step 1: Time the computation manually
console.time('filter products');
const result = expensiveFilter(products);
console.timeEnd('filter products');
// If < 1ms → don't memoize

// Step 2: Use React DevTools Profiler
// - Record a user interaction
// - Look for components highlighted in the flamegraph
// - Check "Why did this render?" — if props didn't change → add memo
// - Check if a specific computation dominates render time → add useMemo

// Step 3: Verify memoization is working
// - Wrap computation, run profiler again
// - Confirm the highlighted time decreased
```

---

## Decision Table

| Scenario | React Compiler handles? | Manual action needed? |
|----------|------------------------|-----------------------|
| Inline object/array as prop | ✅ Yes | None |
| Event handler defined in component body | ✅ Yes | None |
| Derived array/object from props | ✅ Yes | None |
| Trivial computation (< 1ms) | ✅ Yes | None |
| Computation ≥ 1ms, single component | ✅ Usually | Only if profiler shows issue |
| Computation shared across 3+ components | ❌ No — separate per instance | Lift up + `useMemo` at parent |
| Standalone utility function (outside component) | ❌ No | `useMemo` at call site if expensive |
| Stable ref needed by external (non-React) API | ⚠️ Partial | Explicit `useCallback` |
| Object dependency of `useEffect` | ⚠️ Partial | Explicit `useMemo` to guarantee stability |
| Component receiving unstable props from outside compiler scope | ⚠️ Partial | `memo()` after profiler confirms issue |
