---
name: frontend-page-architector
description: >
  Architect a complete frontend flow following the FlowListPage → FlowEntityDetailsPage → Views → Cards
  pattern established by MachinesListPage and BottlesListPage. Use when creating a new entity management
  flow (list + details + tabs + editable cards). Asks questions first, then generates all required files.
---

# Frontend Page Architector

Generate a complete frontend entity management flow following the established SLA-Admin pattern:

```
{Entity}ListPage → {Entity}DetailsPage → {Tab}View → {FieldGroup}Card
```

This is the same pattern used by Machines (with live API) and Bottles (with mocked Redux state).

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating a new entity management section (list + details pages)
- Adding a new "flow" with tabs inside a details page
- Building a Redux slice + views + cards for a new domain entity
- Structuring a new section that follows the FlowListPage → DetailsPage → Views → Cards hierarchy

**Trigger phrases:** "new flow", "new page for", "create [entity] management", "add [entity] list", "entity details page", "tab-based details"

## Phase 1 — Gather Requirements (MANDATORY — Ask Before Generating)

**You MUST ask these questions before writing any code.** Ask all required questions in a single message. Do not guess.

### Required Questions

**Q1 — Entity name**
What is the entity called? (e.g., "Product", "Campaign", "Supplier")
- This becomes `{Entity}` throughout — affects file names, types, Redux slice name, routes

**Q2 — Flow purpose**
Describe what this flow manages in 1–3 sentences. What are the key fields/data?
- Helps determine tab structure and card breakdown

**Q3 — Data source**
Is the backend API already implemented, or do we need mocked Redux state for now?
- **API ready**: slice starts with `initialState = null`, fetch on mount with `useApiCall`
- **Backend pending**: slice starts with mocked `initialState` object matching entity fields; fetch commented out with `//TODO`

**Q4 — Tab structure**
What tabs should the details page have? (e.g., "General Information, Specifications, History")
- First tab is always the editable one with the Edit/Save button
- Other tabs can be read-only views

**Q5 — Card breakdown per tab**
For each tab, what logical groups of fields should appear as separate Cards?
- Example: "General Info tab → [Entity Information Card, Image Card, Notes Card]"
- Example: "Specs tab → [Dimensions Card, Materials Card]"

**Q6 — Entity fields**
List the fields that belong to this entity (name, type, and which card they go in).
- Needed to define the TypeScript type and the `initialState` (if mocked)
- Example: `name: string`, `status: 'Active' | 'Inactive'`, `capacity: string`

**Q7 — Figma design**
Is there a Figma design available for this flow? If yes, please share the link or paste screenshots.
- If a design exists, it takes priority for layout, field order, labels, and visual structure
- If no design exists, the generated code will follow the existing Machines/Bottles pattern as a default

**Q8 — Existing similar code**
Is there existing code (a similar flow, types, or API service) I should reference or align with?
- Paste relevant snippets or file paths if available

**Q9 — Route path**
What URL path should this flow use? (e.g., `/products`, `/campaigns`)
- List page: `/{path}`
- Details page: `/{path}/info/:id`

### Optional Questions (ask if not clear from answers above)

**Q10 — Create action**
Should the list page have a "+ Add {Entity}" button that opens a modal? Or is it read-only?

**Q11 — Summary cards**
What count/summary cards should appear at the top of the list page? (Total, Active, Inactive, etc.)

---

## Phase 2 — Plan the File Structure

After gathering answers, present the planned file structure for user confirmation before writing anything:

```
frontend/src/
├── app/
│   ├── pages/
│   │   ├── {Entity}ListPage.tsx
│   │   └── {Entity}DetailsPage.tsx
│   ├── components/{Entity}s/
│   │   ├── views/
│   │   │   ├── GeneralInfoView.tsx        ← editable, with ref + save
│   │   │   └── {SecondTab}View.tsx        ← read-only (if applicable)
│   │   └── cards/
│   │       ├── {FieldGroup1}Card.tsx
│   │       └── {FieldGroup2}Card.tsx
│   └── store/
│       └── {camelEntity}Slice.ts
└── App.tsx                                 ← add route entries
```

Also show what will be updated:
- `frontend/src/app/store/index.ts` — register new reducer
- `frontend/src/App.tsx` — add route entries
- `frontend/src/app/support/types.ts` — add `{Entity}Details` type

---

## Phase 3 — Generate Files

Generate files in this order. Use templates as the source of truth for code patterns.

### Step 1: TypeScript Type

Add to `frontend/src/app/support/types.ts`:

```ts
export interface {{EntityName}}Details {
	id: string;
	name: string;
	status: 'Active' | 'Inactive';
	// ... other fields from Q6
}
```

### Step 2: Redux Slice

File: `frontend/src/app/store/{{camelEntity}}Slice.ts`

Use **frontend-flow-entity-slice** template:
- If API ready: `initialState = null` pattern
- If backend pending: mocked `initialState` object with all fields from Q6

Then update `frontend/src/app/store/index.ts`:
```ts
import {{camelEntity}}Reducer from './{{camelEntity}}Slice';
// Add to combineReducers: {{camelEntity}}: {{camelEntity}}Reducer
```

### Step 3: Card Components

File: `frontend/src/app/components/{{EntityName}}s/cards/{{FieldGroup}}Card.tsx`

Use **frontend-flow-card-component** template for each card:
- Text inputs: `name` attribute, `disabled={!isEditMode}`, `defaultValue`
- Selects: controlled `useState` + `<input type="hidden">` for FormData capture
- Prop type: `Partial<Pick<{{EntityName}}Details, 'field1' | 'field2'>>`
- **No inline `style={{}}` props** — all styles go in `_product-cards.scss`. Use the `frontend-scss-writer` skill when new CSS classes are needed. Colors must use variables from `_variables.scss`.

### Step 4: View Components

File: `frontend/src/app/components/{{EntityName}}s/views/GeneralInfoView.tsx`

Use **frontend-flow-view-component** template:
- Two-component split: Guard (exported) + Form (internal)
- `useRef<HTMLFormElement>` + `useImperativeHandle` + `useCallback` on `submitForm`
- If API ready: real `useApiCall` call in `submitForm`
- If backend pending: stubbed `update{{EntityName}} = () => console.log(...)` with TODO comment

For each additional tab view:
File: `frontend/src/app/components/{{EntityName}}s/views/{{TabName}}View.tsx`
- Read-only views: no ref, no save logic, just `useSelector` + display cards

### Step 5: Details Page

File: `frontend/src/app/pages/{{EntityName}}DetailsPage.tsx`

Use **frontend-flow-details-page** template:
- `useRef<{{EntityName}}GeneralInfoViewHandle>` for the editable view
- `handleEditSave`: toggle pattern → calls `ref.save()` on second click
- `renderSelectedTabContent`: switch over tab names
- If API ready: `useEffect` → `fetchDetails(id)` → `dispatch(setDetails(response.data))`
- If backend pending: fetch commented out with `//TODO: Uncomment when BE is implemented`

### Step 6: List Page

File: `frontend/src/app/pages/{{EntityName}}ListPage.tsx`

Use **frontend-flow-list-page** template:
- `useApiCall` + `useTransition` for list fetch
- List data in local `useState` (NOT Redux)
- Summary `SingleCard` components for counts (from Q10)
- `CustomTable` + `navigate` on row edit
- If create action needed (Q9): `ButtonCard` + Modal

### Step 7: Route Registration

Update `frontend/src/App.tsx` — add route entries:
```tsx
<Route path="/{{routePath}}" element={<PrivateRoute><{{EntityName}}sListPage /></PrivateRoute>} />
<Route path="/{{routePath}}/info/:id" element={<PrivateRoute><{{EntityName}}DetailsPage /></PrivateRoute>} />
```

---

## Phase 4 — Post-Generation Checklist

After writing all files, confirm:

- [ ] `{{camelEntity}}Slice.ts` created and registered in `store/index.ts`
- [ ] `{{EntityName}}Details` type added to `types.ts`
- [ ] All card components created with correct `name` attributes
- [ ] `GeneralInfoView` uses Guard + Form two-component split
- [ ] `{{EntityName}}DetailsPage` has `useRef<ViewHandle>` pointing to `GeneralInfoView`
- [ ] `{{EntityName}}ListPage` fetches list into local state (not Redux)
- [ ] Routes added to `App.tsx`
- [ ] If backend pending: all API calls commented out with `//TODO` markers
- [ ] No `style={{}}` inline props in any card or view component — styles in `_product-cards.scss`, colors via `_variables.scss`

---

## Architecture Reference

The established pattern:

### Data Flow

```
ListPage            DetailsPage           View              Card
─────────           ─────────────         ────────          ────────
fetchList()   →     fetchById(id)   →     useSelector()  →  props
local state         dispatch(set...)      Redux store       isEditMode
navigate(id)        ref.save()      ←     useImperativeHandle
                    isEditMode      →     forwardRef
```

### Redux Responsibility Split

| Data | Where |
|------|-------|
| List of entities | Local state in ListPage |
| Selected entity details | Redux slice |
| Edit mode flag | Local state in DetailsPage |
| Form field values | Uncontrolled inputs (FormData) |
| Select values | Local state in Card (for FormData capture) |

### Key Technical Patterns

**FormData collection** (why `name` attributes matter):
```tsx
// In view's save():
const formData = new FormData(formRef.current);
const value = formData.get('fieldName') as string;
// Works for: <Input name="fieldName" />, <input type="hidden" name="fieldName" value={x} />
// Does NOT work for: <Select> (needs hidden input workaround)
```

**Imperative ref chain** (DetailsPage → View → Form):
```
DetailsPage.handleEditSave()
  → generalInfoViewRef.current.save()       // DetailsPage calls View
    → submitForm(new FormData(formRef.current))  // View collects form data
      → updateEntity(fields)                 // View calls API
      → dispatch(setEntityDetails(merged))   // View updates Redux
```

**Guard + Form type safety**:
```tsx
// Guard reads from Redux (nullable)
const details = useSelector(selectEntityDetails); // EntityDetails | null
if (!details) return <skeleton />;
// Form receives non-nullable prop — zero null checks needed inside
return <EntityForm entityDetails={details} />;
```

---

## Required Templates

| Template | Purpose | Location |
|----------|---------|----------|
| **frontend-flow-entity-slice** | Redux slice (null or mocked initial state) | `.claude/templates/frontend-flow-entity-slice.md` |
| **frontend-flow-list-page** | List page with fetch, counts, table, optional modal | `.claude/templates/frontend-flow-list-page.md` |
| **frontend-flow-details-page** | Details page with tabs, edit mode, ref | `.claude/templates/frontend-flow-details-page.md` |
| **frontend-flow-view-component** | Guard + Form view with useImperativeHandle | `.claude/templates/frontend-flow-view-component.md` |
| **frontend-flow-card-component** | Leaf card components with isEditMode | `.claude/templates/frontend-flow-card-component.md` |

**All code MUST come from these templates.** Do not write inline patterns not present in templates.

---

## Example Interaction

**User**: "Create a Suppliers management flow"

**You ask**:
> Before I generate the files, I need a few details:
>
> 1. **Entity name**: Is it "Supplier" (singular)? What's the plural — "Suppliers"?
> 2. **Flow purpose**: What does a Supplier entity represent? Key fields?
> 3. **Data source**: Is the backend API ready, or should I use mocked Redux state for now?
> 4. **Tab structure**: What tabs should the Supplier details page have?
> 5. **Card breakdown**: For each tab, what logical card groups?
> 6. **Entity fields**: List the fields (name + type)?
> 7. **Figma design**: Is there a Figma design available? If yes, please share the link or screenshots.
> 8. **Similar existing code**: Any similar flow or types I should reference?
> 9. **Route path**: What URL path? (e.g., `/suppliers`)
> 10. **Create action**: Should the list have a "+ Add Supplier" button?
> 11. **Summary cards**: What counts to show? (Total, Active, Inactive?)

**User answers** → You plan the file structure → User confirms → You generate all files.
