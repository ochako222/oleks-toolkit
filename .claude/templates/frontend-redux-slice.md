# Redux Toolkit Slice Template

Code pattern for creating an RTK slice following existing SLA-Admin project conventions.

## Pattern: Simple Slice (state + actions + selectors)

```ts
import { createSlice, type PayloadAction } from '@reduxjs/toolkit';
import type { RootState } from '.';

// 1. Define state interface
interface {{Name}}State {
	{{stateName}}: {{StateType}};
	{{anotherState}}: {{AnotherType}};
}

// 2. Initial state
const initialState: {{Name}}State = {
	{{stateName}}: {{defaultValue}},
	{{anotherState}}: {{anotherDefault}},
};

// 3. Create slice
export const {{camelName}}Slice = createSlice({
	name: '{{camelName}}',
	initialState,
	reducers: {
		set{{StateName}}(state, action: PayloadAction<{{StateType}}>) {
			state.{{stateName}} = action.payload;
		},
		set{{AnotherState}}(state, action: PayloadAction<{{AnotherType}}>) {
			state.{{anotherState}} = action.payload;
		},
		reset{{Name}}() {
			return initialState;
		},
	},
});

// 4. Export actions
export const { set{{StateName}}, set{{AnotherState}}, reset{{Name}} } = {{camelName}}Slice.actions;

// 5. Export selectors
export const select{{StateName}} = (state: RootState) => state.{{camelName}}.{{stateName}};
export const select{{AnotherState}} = (state: RootState) => state.{{camelName}}.{{anotherState}};

// 6. Export reducer
export default {{camelName}}Slice.reducer;
```

## Pattern: Entity Slice (nullable object, like machineSlice)

```ts
import { createSlice, type PayloadAction } from '@reduxjs/toolkit';
import type { RootState } from '.';
import type { {{EntityType}} } from 'src/app/support/types';

// State is the entity itself or null
const initialState: {{EntityType}} | null = null;

export const {{camelName}}Slice = createSlice({
	name: '{{camelName}}',
	initialState: initialState as {{EntityType}} | null,
	reducers: {
		set{{EntityName}}: (_state, action: PayloadAction<{{EntityType}}>) => {
			return action.payload;
		},
		update{{EntityName}}: (state, action: PayloadAction<Partial<{{EntityType}}>>) => {
			if (state) {
				return { ...state, ...action.payload };
			}
			return state;
		},
		clear{{EntityName}}: () => {
			return null;
		},
	},
});

export const { set{{EntityName}}, update{{EntityName}}, clear{{EntityName}} } =
	{{camelName}}Slice.actions;

export const select{{EntityName}} = (state: RootState) => state.{{camelName}};

export default {{camelName}}Slice.reducer;
```

## Pattern: Slice with Multiple Concerns (like authSlice)

```ts
import { createSlice, type PayloadAction } from '@reduxjs/toolkit';
import type { RootState } from '.';

interface {{Name}}State {
	{{primaryData}}: {{PrimaryType}} | null;
	isLoading: boolean;
	{{secondaryData}}: {
		{{field1}}: {{FieldType1}} | null;
		{{field2}}: {{FieldType2}} | null;
	};
}

const initialState: {{Name}}State = {
	{{primaryData}}: null,
	isLoading: false,
	{{secondaryData}}: {
		{{field1}}: null,
		{{field2}}: null,
	},
};

export const {{camelName}}Slice = createSlice({
	name: '{{camelName}}',
	initialState,
	reducers: {
		set{{Name}}Data(
			state,
			action: PayloadAction<{
				{{primaryData}}: {{PrimaryType}};
				{{field1}}: {{FieldType1}};
				{{field2}}: {{FieldType2}};
			}>,
		) {
			state.{{primaryData}} = action.payload.{{primaryData}};
			state.{{secondaryData}}.{{field1}} = action.payload.{{field1}};
			state.{{secondaryData}}.{{field2}} = action.payload.{{field2}};
			state.isLoading = false;
		},
		setLoading(state, action: PayloadAction<boolean>) {
			state.isLoading = action.payload;
		},
		reset{{Name}}(state) {
			state.{{primaryData}} = null;
			state.{{secondaryData}}.{{field1}} = null;
			state.{{secondaryData}}.{{field2}} = null;
			state.isLoading = false;
		},
	},
});

export const { set{{Name}}Data, setLoading, reset{{Name}} } = {{camelName}}Slice.actions;

export const select{{PrimaryData}} = (state: RootState) => state.{{camelName}}.{{primaryData}};
export const selectIsLoading = (state: RootState) => state.{{camelName}}.isLoading;
export const select{{SecondaryData}} = (state: RootState) => state.{{camelName}}.{{secondaryData}};

export default {{camelName}}Slice.reducer;
```

## Store Registration

After creating a new slice, register it in `frontend/src/app/store/index.ts`:

```ts
// Add import
import {{camelName}}Reducer from './{{camelName}}Slice';

// Add to combineReducers
const rootReducers = combineReducers({
	machine: machineReducer,
	auth: authReducer,
	{{camelName}}: {{camelName}}Reducer, // ← ADD THIS LINE
});
```

## Typed Hooks (create if missing)

If `frontend/src/app/hooks/useRedux.ts` does not exist, create it:

```ts
import { useDispatch, useSelector } from 'react-redux';
import type { AppDispatch, RootState } from 'src/app/store';

export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<RootState>();
```

## Key Conventions

- **File location**: `frontend/src/app/store/{{camelName}}Slice.ts`
- **Naming**: camelCase slice name + "Slice" suffix (e.g., `notificationSlice.ts`)
- **Slice name field**: Must match the key in `combineReducers` (e.g., `name: 'notification'` → `notification: notificationReducer`)
- **Exports**: Named exports for actions and selectors, default export for reducer
- **Selectors**: Co-located in the slice file, prefixed with `select` (e.g., `selectNotifications`)
- **PayloadAction**: Always type action payloads explicitly
- **Immutability**: Immer is built into RTK — mutate state directly in reducers (it's safe)
- **RootState import**: Always from `'.'` (relative to store directory) or `'src/app/store'`

## Usage in Components

```tsx
import { useDispatch, useSelector } from 'react-redux';
import type { AppDispatch } from 'src/app/store';
import { select{{StateName}}, set{{StateName}} } from 'src/app/store/{{camelName}}Slice';

const MyComponent = () => {
	const dispatch = useDispatch<AppDispatch>();
	const {{stateName}} = useSelector(select{{StateName}});

	const handleUpdate = (newValue: {{StateType}}) => {
		dispatch(set{{StateName}}(newValue));
	};

	return <div>{{{stateName}}}</div>;
};
```

Or with typed hooks (if created):

```tsx
import { useAppDispatch, useAppSelector } from 'src/app/hooks/useRedux';
import { select{{StateName}}, set{{StateName}} } from 'src/app/store/{{camelName}}Slice';

const MyComponent = () => {
	const dispatch = useAppDispatch();
	const {{stateName}} = useAppSelector(select{{StateName}});

	const handleUpdate = (newValue: {{StateType}}) => {
		dispatch(set{{StateName}}(newValue));
	};

	return <div>{{{stateName}}}</div>;
};
```
