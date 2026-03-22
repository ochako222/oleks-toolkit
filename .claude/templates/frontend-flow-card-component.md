# Frontend Flow Card Component Template

Leaf-level card components used inside view components. Each card owns one logical group of fields. Based on cards in `Machines/cards/` and `Bottles/cards/`.

## Pattern: Informational Card (mixed inputs + selects)

```tsx
import { Card, Col, Input, Row, Select, Tag } from 'antd';
import { useState } from 'react';
import type { {{EntityName}}Details } from 'src/app/support/types';
import { colorMap } from 'src/app/support/types';

export const {{CardName}}Card = ({
	{{camelEntity}}Info,
	isEditMode,
}: {
	{{camelEntity}}Info: Partial<Pick<{{EntityName}}Details, '{{field1}}' | '{{field2}}' | 'status'>>;
	isEditMode?: boolean;
}) => {
	// Controlled state for selects — needed for hidden input binding
	const [selected{{Field2}}, setSelected{{Field2}}] = useState({{camelEntity}}Info.{{field2}});
	const [selectedStatus, setSelectedStatus] = useState({{camelEntity}}Info.status);

	return (
		<Card
			className="product-card-details"
			title="{{CardTitle}}"
			extra={
				{{camelEntity}}Info.status ? (
					<Tag color={colorMap[{{camelEntity}}Info.status]} key={{{camelEntity}}Info.status}>
						{{{camelEntity}}Info.status}
					</Tag>
				) : null
			}
		>
			{/* Plain text inputs — FormData reads via name attribute */}
			<Row gutter={16} style={{ marginBottom: 16 }}>
				<Col span={12}>
					<div>{{Field1Label}}</div>
					<Input
						defaultValue={{{camelEntity}}Info.{{field1}}}
						name="{{field1}}"
						disabled={!isEditMode}
					/>
				</Col>
				<Col span={12}>
					<div>ID (read-only)</div>
					<Input defaultValue={{{camelEntity}}Info.id} name="id" disabled />
				</Col>
			</Row>

			{/* Select with hidden input for FormData capture */}
			<Row style={{ marginBottom: 16 }}>
				<div>{{Field2Label}}</div>
				<Select
					value={selected{{Field2}}}
					onChange={(value) => setSelected{{Field2}}(value)}
					style={{ width: '100%' }}
					disabled={!isEditMode}
					options={[
						{ value: 'Option1', label: 'Option 1' },
						{ value: 'Option2', label: 'Option 2' },
					]}
				/>
				<input type="hidden" name="{{field2}}" value={selected{{Field2}} ?? ''} />
			</Row>

			<Row>
				<div>Status</div>
				<Select
					value={selectedStatus}
					onChange={(value) => setSelectedStatus(value)}
					style={{ width: '100%' }}
					disabled={!isEditMode}
					options={[
						{ value: 'Active', label: 'Active' },
						{ value: 'Inactive', label: 'Inactive' },
					]}
				/>
				<input type="hidden" name="status" value={selectedStatus ?? ''} />
			</Row>
		</Card>
	);
};
```

## Pattern: Text-only Card (all plain inputs)

```tsx
import { Card, Col, Input, Row } from 'antd';
import type { {{EntityName}}Details } from 'src/app/support/types';

export const {{CardName}}Card = ({
	{{camelEntity}}Info,
	isEditMode,
}: {
	{{camelEntity}}Info: Partial<Pick<{{EntityName}}Details, '{{field1}}' | '{{field2}}' | '{{field3}}'>>;
	isEditMode?: boolean;
}) => {
	return (
		<Card title="{{CardTitle}}">
			<div style={{ marginBottom: 16 }}>
				<div>{{Field1Label}}</div>
				<Input
					defaultValue={{{camelEntity}}Info.{{field1}}}
					name="{{field1}}"
					disabled={!isEditMode}
				/>
			</div>

			<div style={{ marginBottom: 16 }}>
				<div>{{Field2Label}}</div>
				<Input
					defaultValue={{{camelEntity}}Info.{{field2}}}
					name="{{field2}}"
					disabled={!isEditMode}
				/>
			</div>

			<div>
				<div>{{Field3Label}}</div>
				<Input
					defaultValue={{{camelEntity}}Info.{{field3}}}
					name="{{field3}}"
					disabled={!isEditMode}
				/>
			</div>
		</Card>
	);
};
```

## Pattern: Textarea Card (notes / description)

```tsx
import { Card, Row } from 'antd';
import TextArea from 'antd/es/input/TextArea';
import type { {{EntityName}}Details } from 'src/app/support/types';

export const {{CardName}}Card = ({
	{{camelEntity}}Info,
	isEditMode,
}: {
	{{camelEntity}}Info: Partial<Pick<{{EntityName}}Details, 'notes'>>;
	isEditMode?: boolean;
}) => {
	return (
		<Card title="{{CardTitle}}">
			<Row>
				<div>{{FieldLabel}}</div>
				<TextArea
					rows={3}
					name="notes"
					defaultValue={{{camelEntity}}Info.notes ?? ''}
					disabled={!isEditMode}
				/>
			</Row>
		</Card>
	);
};
```

## Pattern: Image / Media Card

```tsx
import { CameraOutlined, UploadOutlined } from '@ant-design/icons';
import { Button, Card } from 'antd';

export const {{CardName}}Card = ({ image_url }: { image_url?: string }) => {
	return (
		<Card title="{{CardTitle}}">
			<div
				style={{
					border: '2px dashed #d9d9d9',
					borderRadius: 8,
					padding: 40,
					textAlign: 'center',
					marginBottom: 16,
				}}
			>
				{image_url ? (
					<img
						src={image_url}
						alt="entity_image"
						style={{ maxWidth: '100%', maxHeight: 200 }}
					/>
				) : (
					<>
						<CameraOutlined
							style={{ fontSize: 48, color: '#1890ff', display: 'block' }}
						/>
						<p style={{ marginTop: 12, color: '#8c8c8c' }}>
							Click or drag file to upload
						</p>
						<p style={{ color: '#bfbfbf', fontSize: 12 }}>
							PNG or JPG, max 5 MB
						</p>
					</>
				)}
			</div>
			<Button icon={<UploadOutlined />} disabled>
				Click to Upload
			</Button>
		</Card>
	);
};
```

## Key Conventions

- **File location**: `frontend/src/app/components/{{EntityName}}s/cards/{{CardName}}Card.tsx`
- **Prop typing**: `Partial<Pick<{{EntityName}}Details, 'field1' | 'field2'>>` — only declare the fields the card actually uses
- **`isEditMode` prop**: controls `disabled` on all inputs. `disabled={!isEditMode}`
- **`name` attributes**: required on all `<Input>` elements so `new FormData(formRef.current)` captures values
- **Selects**: need controlled `useState` + a `<input type="hidden" name="fieldName" value={selected} />` for `FormData` capture (Ant Design `Select` is not a native input)
- **`defaultValue` vs `value`**: use `defaultValue` for text inputs (uncontrolled), `value` for selects (controlled)
- **No edit button per card**: the edit mode is controlled globally at the DetailsPage level and passed down as `isEditMode`
- **Single responsibility**: one card per logical field group (e.g., location fields, configuration fields, dimensions)
- **`className="product-card-details"`**: add to Card for consistent styling with existing Bottles/Machines cards
