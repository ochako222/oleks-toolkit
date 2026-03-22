# Frontend Flow Entity Slice Template

Redux Toolkit slice for a single entity in the details page flow. Follows the "replace state" pattern from `machineSlice` and `bottleSlice`.

## Pattern: Entity Details Slice (API-ready, null initial state)

Use when the backend endpoint exists. Initial state is `null`; data is populated via API call.

```ts
import { createSlice } from '@reduxjs/toolkit';
import type { RootState } from '.';
import type { {{EntityType}} } from 'src/app/support/types';

const initialState: {{EntityType}} | null = null;

export const {{camelEntity}}Slicer = createSlice({
	name: '{{camelEntity}}',
	initialState: initialState as {{EntityType}} | null,
	reducers: {
		set{{EntityName}}Details: (_state, action) => {
			return action.payload;
		},
	},
});

export const { set{{EntityName}}Details } = {{camelEntity}}Slicer.actions;

export const select{{EntityName}}Details = (state: RootState) => state.{{camelEntity}};

export default {{camelEntity}}Slicer.reducer;
```

## Pattern: Entity Details Slice (Mocked, backend pending)

Use when the backend is not yet implemented. Initial state is a mocked object. Same reducer — once BE is ready, swap `initialState` for `null`.

```ts
import { createSlice } from '@reduxjs/toolkit';
import type { RootState } from '.';

// TODO: Replace mocked initialState with null when BE is implemented
const initialState = {
	id: '1',
	name: 'Example Entity',
	status: 'Active',
	// ... other fields matching {{EntityType}}
};

export const {{camelEntity}}Slicer = createSlice({
	name: '{{camelEntity}}',
	initialState,
	reducers: {
		set{{EntityName}}Details: (_state, action) => {
			return action.payload;
		},
	},
});

export const { set{{EntityName}}Details } = {{camelEntity}}Slicer.actions;

export const select{{EntityName}}Details = (state: RootState) => state.{{camelEntity}};

export default {{camelEntity}}Slicer.reducer;
```

## Store Registration

After creating the slice, register it in `frontend/src/app/store/index.ts`:

```ts
import {{camelEntity}}Reducer from './{{camelEntity}}Slice';

const rootReducers = combineReducers({
	machine: machineReducer,
	auth: authReducer,
	{{camelEntity}}: {{camelEntity}}Reducer, // ← ADD THIS LINE
});
```

## Key Conventions

- **File location**: `frontend/src/app/store/{{camelEntity}}Slice.ts`
- **Reducer pattern**: `return action.payload` (replace, not mutate) — matches `machineSlice` and `bottleSlice`
- **initialState**: `null` for API-backed data; mocked object when BE is pending
- **Selector**: co-located, prefixed `select`, e.g. `selectBottleDetails`, `selectMachineDetails`
- **Slicer name**: must match the key in `combineReducers`
