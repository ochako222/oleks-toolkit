# Template: Render Props Pattern

Reference implementation for the Render Props pattern in this project.
Use this when sharing stateful logic between components while keeping full control over rendering.

---

## Core Structure

```tsx
// Input.tsx — component with internal state, delegates rendering to caller
import { useState } from 'react';

interface InputProps {
	render: (value: string) => React.ReactNode;
}

function Input({ render }: InputProps) {
	const [value, setValue] = useState('');

	return (
		<>
			<input
				value={value}
				onChange={(e) => setValue(e.target.value)}
				placeholder="Type something..."
			/>
			{render(value)}
		</>
	);
}

export default Input;
```

---

## Usage — Named `render` Prop

```tsx
// Parent controls what gets rendered with the shared value
<Input
	render={(value) => (
		<>
			<Kelvin value={((value as unknown as number) + 273.15).toFixed(2)} />
			<Fahrenheit value={((value as unknown as number) * 9) / 5 + 32} />
		</>
	)}
/>
```

---

## Usage — Children as Function

```tsx
// Alternative syntax: pass function as children
interface DataProviderProps {
	children: (data: string[]) => React.ReactNode;
	url: string;
}

function DataProvider({ children, url }: DataProviderProps) {
	const [data, setData] = useState<string[]>([]);

	useEffect(() => {
		fetch(url)
			.then((res) => res.json())
			.then(setData);
	}, [url]);

	return <>{children(data)}</>;
}
```

```tsx
// Usage:
<DataProvider url="/api/machines">
	{(machines) => <MachineTable rows={machines} />}
</DataProvider>
```

---

## Real-World Pattern — Hover Tracker

```tsx
// HoverProvider.tsx — tracks hover state, delegates visual to consumer
interface HoverProviderProps {
	render: (isHovered: boolean) => React.ReactNode;
}

function HoverProvider({ render }: HoverProviderProps) {
	const [isHovered, setIsHovered] = useState(false);

	return (
		<div
			onMouseEnter={() => setIsHovered(true)}
			onMouseLeave={() => setIsHovered(false)}
		>
			{render(isHovered)}
		</div>
	);
}
```

```tsx
// Consumer decides what "hovered" looks like — no naming conflicts
<HoverProvider
	render={(isHovered) => (
		<img
			src={isHovered ? '/img/active.png' : '/img/idle.png'}
			alt="hover demo"
		/>
	)}
/>
```

---

## File Location Convention

Place render-prop providers in `frontend/src/app/components/shared/providers/`:

| File                    | Purpose                                       |
|-------------------------|-----------------------------------------------|
| `HoverProvider.tsx`     | Tracks hover state, delegates render          |
| `DataProvider.tsx`      | Fetches data, delegates list render           |
| `ResizeProvider.tsx`    | Tracks window/element resize                  |

---

## When to Use This Pattern (Decision Checklist)

Apply Render Props when ALL of the following are true:
- [ ] Stateful logic (hover, resize, fetch, keyboard events) needs to be shared
- [ ] Different consumers need to render **different UI** with that shared state
- [ ] Prop-name collisions would be a concern with HOC (render props avoid this)
- [ ] The data source must be explicit at the call site for readability

**Do NOT use Render Props when:**
- A Hook can provide the same logic → prefer `useHover()`, `useFetch()` etc.
- Multiple render props would be nested → creates callback hell, use Hooks instead
- The behavior is the same across all consumers → use HOC instead
- The component needs lifecycle methods not available in render functions
