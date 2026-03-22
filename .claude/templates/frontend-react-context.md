# React Context Template

Code pattern for creating a React Context with Provider and custom hook. Based on AnySkin Cosmetics project patterns.

## Pattern: Simple Context (single value + setter)

```tsx
import { createContext, useCallback, useContext, useMemo, useState, type ReactNode } from 'react';

// 1. Define the context shape
interface {{Name}}ContextType {
	{{stateName}}: {{StateType}};
	set{{StateName}}: (value: {{StateType}}) => void;
}

// 2. Default value (used when consuming outside Provider)
const defaultValue: {{Name}}ContextType = {
	{{stateName}}: {{defaultStateValue}},
	set{{StateName}}: () => {
		// No-op function
	},
};

// 3. Create context
export const {{Name}}Context = createContext<{{Name}}ContextType>(defaultValue);

// 4. Provider component
type {{Name}}ProviderProps = { children: ReactNode };

export const {{Name}}Provider: React.FC<{{Name}}ProviderProps> = ({ children }) => {
	const [{{stateName}}, set{{StateName}}State] = useState<{{StateType}}>({{defaultStateValue}});

	const set{{StateName}} = useCallback((value: {{StateType}}) => {
		set{{StateName}}State(value);
	}, []);

	const contextValue = useMemo(
		() => ({ {{stateName}}, set{{StateName}} }),
		[{{stateName}}, set{{StateName}}],
	);

	return (
		<{{Name}}Context.Provider value={contextValue}>
			{children}
		</{{Name}}Context.Provider>
	);
};

// 5. Custom hook (safe accessor)
export function use{{Name}}(): {{Name}}ContextType {
	const context = useContext({{Name}}Context);
	if (context === defaultValue) {
		throw new Error('use{{Name}} must be used within a {{Name}}Provider');
	}
	return context;
}
```

## Pattern: Complex Context (multiple values, reset, derived state)

```tsx
import { createContext, useCallback, useContext, useMemo, useState, type ReactNode } from 'react';

// 1. Define data types
export interface {{DataType}} {
	{{field1}}: {{FieldType1}};
	{{field2}}: {{FieldType2}};
}

const default{{DataType}}: {{DataType}} = {
	{{field1}}: {{defaultField1}},
	{{field2}}: {{defaultField2}},
};

// 2. Define context shape
interface {{Name}}ContextType {
	{{dataName}}: {{DataType}};
	set{{DataName}}: React.Dispatch<React.SetStateAction<{{DataType}}>>;
	{{booleanFlag}}: boolean;
	set{{BooleanFlag}}: (value: boolean) => void;
	reset{{DataName}}: () => void;
}

const defaultValue: {{Name}}ContextType = {
	{{dataName}}: default{{DataType}},
	set{{DataName}}: () => {},
	{{booleanFlag}}: false,
	set{{BooleanFlag}}: () => {},
	reset{{DataName}}: () => {},
};

// 3. Create context
export const {{Name}}Context = createContext<{{Name}}ContextType>(defaultValue);

// 4. Provider component
type {{Name}}ProviderProps = { children: ReactNode };

export const {{Name}}Provider: React.FC<{{Name}}ProviderProps> = ({ children }) => {
	const [{{dataName}}, set{{DataName}}] = useState<{{DataType}}>(default{{DataType}});
	const [{{booleanFlag}}, set{{BooleanFlag}}State] = useState(false);

	const set{{BooleanFlag}} = useCallback((value: boolean) => {
		set{{BooleanFlag}}State(value);
	}, []);

	const reset{{DataName}} = useCallback(() => {
		set{{DataName}}(default{{DataType}});
		set{{BooleanFlag}}State(false);
	}, []);

	const contextValue = useMemo(
		() => ({
			{{dataName}},
			set{{DataName}},
			{{booleanFlag}},
			set{{BooleanFlag}},
			reset{{DataName}},
		}),
		[{{dataName}}, {{booleanFlag}}, set{{BooleanFlag}}, reset{{DataName}}],
	);

	return (
		<{{Name}}Context.Provider value={contextValue}>
			{children}
		</{{Name}}Context.Provider>
	);
};

// 5. Custom hook
export function use{{Name}}(): {{Name}}ContextType {
	const context = useContext({{Name}}Context);
	if (context === defaultValue) {
		throw new Error('use{{Name}} must be used within a {{Name}}Provider');
	}
	return context;
}
```

## Key Conventions

- **File location**: `frontend/src/app/contexts/{{Name}}Context.tsx`
- **Naming**: PascalCase context name + "Context" suffix (e.g., `ThemeContext.tsx`)
- **Exports**: Named exports only — `{{Name}}Context`, `{{Name}}Provider`, `use{{Name}}`
- **Stability**: Wrap setters with `useCallback`, wrap value object with `useMemo`
- **Safety**: Custom hook checks for usage outside Provider — throws `Error` or logs `console.warn`
- **Default value**: Create a `defaultValue` const with no-op functions for type safety
- **No-op pattern**: Use `() => { /* No-op function */ }` or `() => {}` for default setters
- **Provider props**: Always `{ children: ReactNode }` typed as a separate type

## Provider Placement

After creating the context, the Provider must be added to the component tree. Common locations:

- **App-wide**: Wrap in `frontend/src/App.tsx` or `frontend/src/main.tsx`
- **Feature-scoped**: Wrap in the feature's root page component
- **Route-scoped**: Wrap inside a specific route's layout component
