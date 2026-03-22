# Zustand Store Template

Code pattern for creating a Zustand store with TypeScript types, actions, and selectors.

## Prerequisites

Zustand must be installed:
```bash
cd frontend && npm install zustand
```

## Pattern: Simple Store (state + actions)

```ts
import { create } from 'zustand';

// 1. Define state interface
interface {{Name}}State {
	{{stateName}}: {{StateType}};
	{{anotherState}}: {{AnotherType}};
}

// 2. Define actions interface
interface {{Name}}Actions {
	set{{StateName}}: (value: {{StateType}}) => void;
	set{{AnotherState}}: (value: {{AnotherType}}) => void;
	reset: () => void;
}

// 3. Define the store type
type {{Name}}Store = {{Name}}State & {{Name}}Actions;

// 4. Initial state (extracted for reset)
const initialState: {{Name}}State = {
	{{stateName}}: {{defaultValue}},
	{{anotherState}}: {{anotherDefault}},
};

// 5. Create the store
export const use{{Name}}Store = create<{{Name}}Store>()((set) => ({
	...initialState,

	set{{StateName}}: (value) => set({ {{stateName}}: value }),
	set{{AnotherState}}: (value) => set({ {{anotherState}}: value }),
	reset: () => set(initialState),
}));

// 6. Selectors (optional — for derived state or performance)
export const select{{StateName}} = (state: {{Name}}Store) => state.{{stateName}};
export const select{{AnotherState}} = (state: {{Name}}Store) => state.{{anotherState}};
```

## Pattern: Store with Computed/Derived State

```ts
import { create } from 'zustand';

interface {{Name}}State {
	items: {{ItemType}}[];
	filter: string;
	sortBy: '{{sort1}}' | '{{sort2}}';
}

interface {{Name}}Actions {
	setItems: (items: {{ItemType}}[]) => void;
	setFilter: (filter: string) => void;
	setSortBy: (sortBy: {{Name}}State['sortBy']) => void;
	addItem: (item: {{ItemType}}) => void;
	removeItem: (id: {{IdType}}) => void;
	reset: () => void;
}

type {{Name}}Store = {{Name}}State & {{Name}}Actions;

const initialState: {{Name}}State = {
	items: [],
	filter: '',
	sortBy: '{{sort1}}',
};

export const use{{Name}}Store = create<{{Name}}Store>()((set) => ({
	...initialState,

	setItems: (items) => set({ items }),
	setFilter: (filter) => set({ filter }),
	setSortBy: (sortBy) => set({ sortBy }),

	addItem: (item) =>
		set((state) => ({ items: [...state.items, item] })),

	removeItem: (id) =>
		set((state) => ({
			items: state.items.filter((item) => item.id !== id),
		})),

	reset: () => set(initialState),
}));

// Derived selectors — use these for computed values
export const selectFilteredItems = (state: {{Name}}Store) => {
	const filtered = state.items.filter((item) =>
		item.name.toLowerCase().includes(state.filter.toLowerCase()),
	);
	return filtered;
};
```

## Pattern: Store with Persist Middleware

```ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface {{Name}}State {
	{{stateName}}: {{StateType}};
}

interface {{Name}}Actions {
	set{{StateName}}: (value: {{StateType}}) => void;
	reset: () => void;
}

type {{Name}}Store = {{Name}}State & {{Name}}Actions;

const initialState: {{Name}}State = {
	{{stateName}}: {{defaultValue}},
};

export const use{{Name}}Store = create<{{Name}}Store>()(
	persist(
		(set) => ({
			...initialState,

			set{{StateName}}: (value) => set({ {{stateName}}: value }),
			reset: () => set(initialState),
		}),
		{
			name: '{{kebab-name}}-storage',
			// Uses localStorage by default. For sessionStorage:
			// storage: createJSONStorage(() => sessionStorage),
		},
	),
);
```

## Key Conventions

- **File location**: `frontend/src/app/stores/{{camelCaseName}}Store.ts`
- **Naming**: camelCase store name + "Store" suffix (e.g., `filterStore.ts`)
- **Hook naming**: `use{{PascalName}}Store` (e.g., `useFilterStore`)
- **Exports**: Named exports only — store hook + selector functions
- **Initial state**: Extract to a `const initialState` for clean reset functionality
- **Interfaces**: Separate `State` and `Actions` interfaces, combine as `Store` type
- **No Provider needed**: Zustand stores are imported and used directly in components

## Usage in Components

```tsx
import { use{{Name}}Store, select{{StateName}} } from 'src/app/stores/{{camelCaseName}}Store';

const MyComponent = () => {
	// Option 1: Direct access
	const {{stateName}} = use{{Name}}Store((state) => state.{{stateName}});
	const set{{StateName}} = use{{Name}}Store((state) => state.set{{StateName}});

	// Option 2: Using selectors
	const {{stateName}} = use{{Name}}Store(select{{StateName}});

	// Option 3: Multiple values (re-renders on any change — use sparingly)
	const { {{stateName}}, {{anotherState}} } = use{{Name}}Store();

	return <div>{{{stateName}}}</div>;
};
```
