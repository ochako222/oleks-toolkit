---
name: frontend-pattern-advisor
description: >
  Evaluate whether HOC or Render Props patterns should be applied when building or refactoring
  React components. Automatically triggered during any frontend development task. Shows a
  decision block with reasoning before writing code — or explicitly confirms no pattern is needed.
---

# Frontend Pattern Advisor

**AUTOMATICALLY INVOKE THIS SKILL** at the start of every frontend development task before writing any component, view, card, or hook code. The decision must be shown to the user as a visible block.

---

## When to Invoke

Invoke this skill whenever the task involves ANY of the following:

- Creating a new React component, view, card, or page
- Refactoring an existing component
- Extracting shared logic from multiple components
- Adding cross-cutting behavior (auth guards, loading states, data fetching, hover tracking, etc.)
- Building utility or wrapper components

---

## How to Run the Evaluation

### Step 1 — Identify Candidates

Read the task. Look for signals that a pattern might apply:

**HOC signals:**
- Same behavior needed across 3+ independent components
- Cross-cutting concerns: auth guards, loading spinners, style wrappers, analytics tracking
- Components should work standalone without the behavior
- No per-component variation in the behavior

**Render Props signals:**
- Stateful logic (hover, resize, keyboard, drag, timer) needs to be shared
- Different consumers need to render completely different UI using that shared state
- Prop naming collisions would be a concern with HOC
- The data source must be visible at the call site

**No pattern signals:**
- Logic is used in only 1–2 places → inline or use a custom Hook
- Hooks (`useHover`, `useFetch`, etc.) would solve it more cleanly
- The component is a leaf node with no shared behavior
- Adding a pattern would increase complexity without reducing duplication

### Step 2 — Apply the Decision Checklists

Reference the checklist sections in the templates:

- HOC checklist → `.claude/templates/frontend-hoc-pattern.md` (section: "When to Use This Pattern")
- Render Props checklist → `.claude/templates/frontend-render-props-pattern.md` (section: "When to Use This Pattern")

Mark each checklist item as met or not met based on the task at hand.

### Step 3 — Output the Decision Block

**Always** show this decision block to the user before writing any code. Use one of the three formats below.

---

## Decision Block Formats

### Format A — HOC Chosen

```
## Pattern Decision: Higher-Order Component (HOC)

**Decision:** Apply HOC pattern.

**Why:**
- [Specific reason from the checklist, tied to this task]
- [e.g., "The same loading + fetch behavior is needed on DogImages, CatImages, and BirdImages — 3 independent components"]
- [e.g., "The components work fine standalone; the HOC only adds the cross-cutting concern"]

**What will be done:**
- Create `withLoader` HOC in `frontend/src/app/hocs/withLoader.tsx`
- Wrap DogImagesComponent, CatImagesComponent, BirdImagesComponent with it
- Each wrapped component receives `data` as a prop; the HOC owns fetch + loading state

**Template used:** `frontend-hoc-pattern.md`
```

---

### Format B — Render Props Chosen

```
## Pattern Decision: Render Props

**Decision:** Apply Render Props pattern.

**Why:**
- [Specific reason from the checklist, tied to this task]
- [e.g., "Three consumers (ImageCard, TextCard, ButtonCard) need to react to hover state but each renders completely different UI — Render Props lets each control its own output"]
- [e.g., "With HOC, the `isHovered` prop name could collide; Render Props avoids this entirely"]

**What will be done:**
- Create `HoverProvider` in `frontend/src/app/components/shared/providers/HoverProvider.tsx`
- Each consumer receives `isHovered: boolean` via the `render` prop and decides its own visual output

**Template used:** `frontend-render-props-pattern.md`
```

---

### Format C — No Pattern Needed

```
## Pattern Decision: No Pattern Needed

**Decision:** Proceed without HOC or Render Props.

**Why:**
- [Specific reason this task doesn't qualify]
- [e.g., "Only one component needs this behavior — a custom Hook is the simpler, cleaner solution"]
- [e.g., "The logic is trivial enough to inline; extracting it would add unnecessary indirection"]

**Approach:** [Describe the simpler path — inline logic / custom Hook / props / etc.]
```

---

## Rules

1. **Never skip this evaluation.** Every frontend task gets a decision block — even if the answer is Format C.
2. **Show the block before writing code.** Do not write any component code before the decision is visible.
3. **Be specific.** Reference the actual component names and real reasons from the task, not generic descriptions.
4. **Prefer Hooks over both patterns** when a Hook would be equally expressive — both HOC and Render Props have been largely superseded by Hooks in React 18+. Only choose them when Hooks cannot cleanly solve the problem.
5. **One pattern at a time.** If both might apply, pick the one that better fits and explain why the other was rejected.
6. **Use the templates for code.** When a pattern is chosen, all code must follow the structure in the corresponding template.

---

## Decision Summary Table

| Scenario | HOC | Render Props | Hook | None |
|----------|-----|--------------|------|------|
| Same behavior on 3+ independent components, no variation | ✓ | | | |
| Different UI per consumer, shared state | | ✓ | | |
| Shared logic, same UI structure | | | ✓ | |
| Auth guard / redirect | ✓ | | | |
| Hover / resize / keyboard tracking, varied output | | ✓ | | |
| Hover / resize / keyboard tracking, same output | | | ✓ | |
| Used in 1–2 places only | | | ✓ | |
| Single-use, trivial logic | | | | ✓ |
| Data fetching + loading | ✓ or Hook | | ✓ | |

---

## Templates

| Template | Location |
|----------|----------|
| `frontend-hoc-pattern` | `.claude/templates/frontend-hoc-pattern.md` |
| `frontend-render-props-pattern` | `.claude/templates/frontend-render-props-pattern.md` |
