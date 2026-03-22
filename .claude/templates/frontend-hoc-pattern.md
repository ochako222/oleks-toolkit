# Template: Higher-Order Component (HOC) Pattern

Reference implementation for the HOC pattern in this project.
Use this when wrapping identical, unchanging cross-cutting behavior across many components.

---

## Core Structure

```tsx
// withLoader.tsx — HOC that shows a loading state while fetching data
import { useState, useEffect, ComponentType } from 'react';
import { Spin } from 'antd';

interface WithLoaderProps {
	url: string;
}

function withLoader<T extends object>(
	WrappedComponent: ComponentType<T & { data: unknown }>,
	url: string,
) {
	return function WithLoaderComponent(props: T) {
		const [data, setData] = useState<unknown>(null);
		const [loading, setLoading] = useState(true);

		useEffect(() => {
			fetch(url)
				.then((res) => res.json())
				.then((json) => {
					setData(json);
					setLoading(false);
				});
		}, []);

		if (loading) return <Spin />;

		return <WrappedComponent {...props} data={data} />;
	};
}

export default withLoader;
```

---

## Usage

```tsx
// Apply to any component that needs the same fetch + loading behavior
const DogImages = withLoader(DogImagesComponent, 'https://dog.ceo/api/breeds/image/random/6');
const CatImages = withLoader(CatImagesComponent, 'https://api.thecatapi.com/v1/images/search?limit=6');

// In render:
<DogImages />
<CatImages />
```

---

## Composing Multiple HOCs

```tsx
// withAuth.tsx — HOC that protects routes requiring authentication
import { ComponentType } from 'react';
import { useSelector } from 'react-redux';
import { Navigate } from 'react-router-dom';
import { selectIsAuthenticated } from 'src/app/store/authSlice';

function withAuth<T extends object>(WrappedComponent: ComponentType<T>) {
	return function WithAuthComponent(props: T) {
		const isAuthenticated = useSelector(selectIsAuthenticated);

		if (!isAuthenticated) return <Navigate to="/login" replace />;

		return <WrappedComponent {...props} />;
	};
}

export default withAuth;
```

```tsx
// withStyles.tsx — HOC that injects a consistent style className
import { ComponentType } from 'react';

function withStyles<T extends object>(
	WrappedComponent: ComponentType<T>,
	className: string,
) {
	return function WithStylesComponent(props: T) {
		return (
			<div className={className}>
				<WrappedComponent {...props} />
			</div>
		);
	};
}

export default withStyles;
```

---

## HOC Naming Convention

| HOC file       | Export name        | Usage                            |
|----------------|--------------------|----------------------------------|
| `withLoader`   | `withLoader`       | Fetch + loading spinner          |
| `withAuth`     | `withAuth`         | Redirect unauthenticated users   |
| `withStyles`   | `withStyles`       | Wrap in styled container         |

Always prefix with `with` and place in `frontend/src/app/hocs/`.

---

## When to Use This Pattern (Decision Checklist)

Apply HOC when ALL of the following are true:
- [ ] The exact same behavior (no per-component customization) is needed on 3+ components
- [ ] The components work fine without the HOC — it adds cross-cutting concerns only
- [ ] No per-component configuration is needed (or it can come from outer scope/props)
- [ ] The behavior is stable and unlikely to differ per usage

**Do NOT use HOC when:**
- The behavior needs minor variations per component → use a Hook
- Only 1–2 components need it → inline the logic or use a Hook
- The HOC would add many props → Hooks are cleaner
- You would need to stack 3+ HOCs on one component → "wrapper hell"
