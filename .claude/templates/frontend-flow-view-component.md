# Frontend Flow View Component Template

View components that render inside a details page tab. Uses the **Guard + Form two-component pattern** for null safety and clean form logic. Based on `GeneralInfoView` in both `Machines/` and `Bottles/`.

## The Two-Component Pattern

```
DetailsPage
    └── ViewComponent (forwardRef, exported)    ← Guard: reads Redux, null-checks, renders skeleton
            └── EntityForm (forwardRef, internal)   ← Form: receives non-nullable entity, handles save
                    └── Card1, Card2, Card3...       ← Leaf card components
```

**Why two components?**
- The Guard component guarantees type safety: `EntityForm` receives a non-nullable prop, so no `?.` chains are needed in form logic
- Ref is attached to the Form component (the one that actually has the `<form>` element)
- Separation of concerns: data loading vs. form handling

---

## Pattern: Editable View (Guard + Form with save logic)

```tsx
import { Card, Col, message, Row } from 'antd';
import { forwardRef, useCallback, useImperativeHandle, useRef } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { api } from 'src/api';
import { useApiCall } from 'src/hooks/useApiCall';
import {
	select{{EntityName}}Details,
	set{{EntityName}}Details,
} from 'src/app/store/{{camelEntity}}Slice';
import type { {{EntityName}}Details } from 'src/app/support/types';
import { {{FieldGroupCard1}} } from '../cards/{{FieldGroupCard1}}';
import { {{FieldGroupCard2}} } from '../cards/{{FieldGroupCard2}}';

// ─── Handle interface (exported for DetailsPage ref typing) ───────────────────
export interface {{EntityName}}GeneralInfoViewHandle {
	save: () => Promise<void>;
}

// ─── Form component (internal) ───────────────────────────────────────────────

/**
 * {{EntityName}}Form Component
 *
 * Only renders when {{camelEntity}}Details is guaranteed to exist (guard handled by parent).
 * Safely uses {{camelEntity}}Details without null checks.
 */
interface {{EntityName}}FormProps {
	{{camelEntity}}Details: {{EntityName}}Details;
	isEditMode?: boolean;
}

const {{EntityName}}Form = forwardRef<{{EntityName}}GeneralInfoViewHandle, {{EntityName}}FormProps>(
	({ {{camelEntity}}Details, isEditMode }, ref) => {
		const dispatch = useDispatch();
		const formRef = useRef<HTMLFormElement>(null);

		const { execute: update{{EntityName}} } = useApiCall(
			api.{{camelEntity}}sService.update{{EntityName}}ById
		);

		const submitForm = useCallback(
			async (formData: FormData): Promise<void> => {
				// Extract form fields
				const {{field1}} = formData.get('{{field1}}') as string;
				const {{field2}} = formData.get('{{field2}}') as string;
				// ... extract other fields from the form

				// Call API
				const result = await update{{EntityName}}({
					id: {{camelEntity}}Details.id,
					{{field1}},
					{{field2}},
				});

				if (!result) {
					message.error('Failed to update {{EntityName}} details');
					throw new Error('Failed to update {{EntityName}} details');
				}

				// Merge updated fields with existing entity and push to Redux
				const updated{{EntityName}}Details: {{EntityName}}Details = {
					...{{camelEntity}}Details,
					{{field1}},
					{{field2}},
				};

				dispatch(set{{EntityName}}Details(updated{{EntityName}}Details));
				message.success('{{EntityName}} details updated successfully');
			},
			[dispatch, {{camelEntity}}Details, update{{EntityName}}]
		);

		useImperativeHandle(
			ref,
			() => ({
				save: async () => {
					if (formRef.current) {
						const formData = new FormData(formRef.current);
						await submitForm(formData);
					}
				},
			}),
			[submitForm]
		);

		return (
			<form ref={formRef}>
				<Row gutter={16}>
					<Col span={12}>
						<{{FieldGroupCard1}}
							{{camelEntity}}Info={{{camelEntity}}Details}
							isEditMode={isEditMode}
						/>
					</Col>
					<Col span={12}>
						<{{FieldGroupCard2}}
							{{camelEntity}}Info={{{camelEntity}}Details}
							isEditMode={isEditMode}
						/>
					</Col>
				</Row>
			</form>
		);
	}
);

// ─── Guard component (exported) ───────────────────────────────────────────────

/**
 * {{EntityName}}GeneralInfoView Component (Guard)
 *
 * Reads entity details from Redux. Renders loading skeleton until data arrives.
 * Once data is confirmed, delegates to {{EntityName}}Form (type-safe, no null checks needed).
 */
interface {{EntityName}}GeneralInfoViewProps {
	isEditMode?: boolean;
}

export const {{EntityName}}GeneralInfoView = forwardRef<
	{{EntityName}}GeneralInfoViewHandle,
	{{EntityName}}GeneralInfoViewProps
>(({ isEditMode }, ref) => {
	const {{camelEntity}}Details = useSelector(select{{EntityName}}Details);

	if (!{{camelEntity}}Details) {
		return (
			<Row gutter={16}>
				<Col span={12}>
					<Card loading={true} />
				</Col>
				<Col span={12}>
					<Card loading={true} />
				</Col>
			</Row>
		);
	}

	return (
		<{{EntityName}}Form
			ref={ref}
			{{camelEntity}}Details={{{camelEntity}}Details}
			isEditMode={isEditMode}
		/>
	);
});
```

## Pattern: Read-only View (no form, no save logic)

For tabs that display data but have no edit capability (e.g., a "Bottles" sub-list inside a Machine):

```tsx
import { Card, Col, Row } from 'antd';
import { useSelector } from 'react-redux';
import { select{{EntityName}}Details } from 'src/app/store/{{camelEntity}}Slice';
import { {{DisplayCard}} } from '../cards/{{DisplayCard}}';

export const {{EntityName}}{{ViewName}}View = () => {
	const {{camelEntity}}Details = useSelector(select{{EntityName}}Details);

	if (!{{camelEntity}}Details) {
		return (
			<Row gutter={16}>
				<Col span={12}>
					<Card loading={true} />
				</Col>
			</Row>
		);
	}

	return (
		<Row gutter={16}>
			<Col span={24}>
				<{{DisplayCard}} {{camelEntity}}Info={{{camelEntity}}Details} />
			</Col>
		</Row>
	);
};
```

## Pattern: Backend pending (mocked Redux, stub submitForm)

When backend endpoint doesn't exist yet, stub the API call:

```tsx
const submitForm = useCallback(
	async (formData: FormData): Promise<void> => {
		// TODO: replace with the real API call when BE is implemented
		// const { execute: update{{EntityName}} } = useApiCall(api.{{camelEntity}}sService.update{{EntityName}}ById);
		const update{{EntityName}} = () => console.log('{{EntityName}} updated (mocked)');

		const {{field1}} = formData.get('{{field1}}') as string;

		const updated{{EntityName}}Details: {{EntityName}}Details = {
			...{{camelEntity}}Details,
			{{field1}},
		};

		dispatch(set{{EntityName}}Details(updated{{EntityName}}Details));
		message.success('{{EntityName}} details updated successfully');
	},
	[dispatch, {{camelEntity}}Details]
);
```

## Key Conventions

- **File location**: `frontend/src/app/components/{{EntityName}}s/views/{{ViewName}}View.tsx`
- **Handle interface**: exported, named `{{EntityName}}{{ViewName}}ViewHandle`, with `save: () => Promise<void>`
- **Two-component split**: Guard (exported, `forwardRef`) + Form (internal, `forwardRef`) — always use this split for editable views
- **`useCallback` on `submitForm`**: wraps the async save logic; listed as `useImperativeHandle` dependency
- **`useImperativeHandle`**: exposes `save()` to parent (DetailsPage). Depends on `[submitForm]`
- **`FormData` API**: field values collected via `new FormData(formRef.current)` — requires `name` attributes on inputs
- **Redux merge pattern**: `{ ...existingDetails, ...updatedFields }` — never replace entire state with only form fields
- **Error handling**: throw inside `submitForm` so `isSaving` cleanup runs in the DetailsPage `finally` block
- **Skeleton count**: match the number of Card columns in the loaded view for visual consistency
