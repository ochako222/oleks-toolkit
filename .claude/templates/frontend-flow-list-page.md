# Frontend Flow List Page Template

Entry point for a flow. Fetches list data, shows summary cards and a table, and navigates to the details page. Based on `MachinesListPage` and `BottlesListPage`.

## Pattern: List Page with API data + optional Create modal

```tsx
import { Col, message, Row, Spin } from 'antd';
import { useEffect, useState, useTransition } from 'react';
import { useNavigate } from 'react-router-dom';
import { api } from 'src/api';
import type { Create{{EntityName}}Params } from 'src/api/{{camelEntity}}s.service';
import { ButtonCard, SingleCard } from 'src/app/components/BaseComponents/Cards';
import { CustomTable } from 'src/app/components/BaseComponents/Table';
import { Add{{EntityName}}Modal } from 'src/app/components/{{EntityName}}s/Add{{EntityName}}Modal';
import {
	Total{{EntityName}}Icon,
	Active{{EntityName}}Icon,
	Inactive{{EntityName}}Icon,
} from 'src/app/components/{{EntityName}}s/{{EntityName}}Icons';
import type { {{EntityName}}Counts, {{EntityName}}TableData, {{EntityName}}ListItem } from 'src/app/support/types';
import { useApiCall } from 'src/hooks/useApiCall';
import { TableContainer } from '../components/TableContainer';

export const {{EntityName}}sListPage = () => {
	const navigate = useNavigate();
	const [isAddModalOpen, setAddModalOpen] = useState(false);
	const [isPending, startTransition] = useTransition();
	const [{{camelEntity}}List, set{{EntityName}}List] = useState<{{EntityName}}ListItem[]>([]);

	const { execute: fetch{{EntityName}}s } = useApiCall(api.{{camelEntity}}sService.getAll{{EntityName}}s);
	const { execute: create{{EntityName}}, isLoading: addLoading } = useApiCall(
		api.{{camelEntity}}sService.create{{EntityName}}
	);

	useEffect(() => {
		startTransition(async () => {
			const response = await fetch{{EntityName}}s();
			if (response?.status === 'success' && response?.data?.length > 0) {
				set{{EntityName}}List(response.data);
			}
		});
	}, []);

	// Derived counts (or use response.counts if backend provides them)
	const totals: {{EntityName}}Counts | undefined = {{camelEntity}}List
		? {
				Total: {{camelEntity}}List.length,
				Active: {{camelEntity}}List.filter((item) => item.status === 'Active').length,
				Inactive: {{camelEntity}}List.filter((item) => item.status === 'Inactive').length,
			}
		: undefined;

	// Map API data to table shape
	const tableData: {{EntityName}}TableData[] = {{camelEntity}}List.map((item) => ({
		// Map fields from API shape to table columns
		id: String(item.id),
		name: item.name,
		status: item.status ?? 'Active',
		// ... other display fields
	}));

	const handleAdd{{EntityName}} = async (values: Create{{EntityName}}Params) => {
		const response = await create{{EntityName}}(values);
		if (response) {
			message.success('{{EntityName}} added successfully');
			setAddModalOpen(false);
			navigate(`/{{camelEntity}}s/info/${response.data.id}`);
		} else {
			message.error('Failed to add {{EntityName}}. Please try again.');
		}
	};

	return (
		<>
			<Row className="layout-title">
				<h2>{{EntityName}}s Management</h2>
				<div>View and manage your team's {{camelEntity}}s</div>
			</Row>

			{/* Summary cards */}
			<Row gutter={16}>
				<Col span={8}>
					<SingleCard
						title="Total {{EntityName}}s"
						description={totals?.Total?.toString() ?? 'Loading...'}
						icon={() => <Total{{EntityName}}Icon />}
					/>
				</Col>
				<Col span={8}>
					<SingleCard
						title="Active {{EntityName}}s"
						description={totals?.Active?.toString() ?? 'Loading...'}
						icon={() => <Active{{EntityName}}Icon />}
					/>
				</Col>
				<Col span={8}>
					<SingleCard
						title="Inactive {{EntityName}}s"
						description={totals?.Inactive?.toString() ?? 'Loading...'}
						icon={() => <Inactive{{EntityName}}Icon />}
					/>
				</Col>
			</Row>

			{/* Table section */}
			<ButtonCard
				title="{{EntityName}}s List"
				buttonTitle="+ Add {{EntityName}}"
				onClick={() => setAddModalOpen(true)}
			/>

			{tableData ? (
				<TableContainer
					table={() => (
						<CustomTable
							data={tableData}
							onEdit={(record) => navigate(`/{{camelEntity}}s/info/${record.id}`)}
							onDelete={(record) => console.log('Delete:', record)}
						/>
					)}
				/>
			) : (
				<Spin />
			)}

			{/* Create modal */}
			<Add{{EntityName}}Modal
				open={isAddModalOpen}
				onCancel={() => setAddModalOpen(false)}
				onSubmit={handleAdd{{EntityName}}}
				loading={addLoading}
			/>
		</>
	);
};
```

## Pattern: List Page (read-only, no Create modal)

When the flow only needs list + navigate to details (no create action):

```tsx
export const {{EntityName}}sListPage = () => {
	const navigate = useNavigate();
	const [isPending, startTransition] = useTransition();
	const [{{camelEntity}}List, set{{EntityName}}List] = useState<{{EntityName}}ListItem[]>([]);

	const { execute: fetch{{EntityName}}s } = useApiCall(api.{{camelEntity}}sService.getAll{{EntityName}}s);

	useEffect(() => {
		startTransition(async () => {
			const response = await fetch{{EntityName}}s();
			if (response?.status === 'success' && response?.data?.length > 0) {
				set{{EntityName}}List(response.data);
			} else if (!response) {
				message.error('Failed to load {{camelEntity}}s. Please try again.');
			}
		});
	}, []);

	const tableData: {{EntityName}}TableData[] = {{camelEntity}}List.map((item) => ({
		// Map fields from API shape to table columns
		id: String(item.id),
		name: item.name,
		// ... other display fields
	}));

	const totals = { Total: {{camelEntity}}List.length };

	return (
		<>
			<Row className="layout-title">
				<h2>{{EntityName}}s Management</h2>
				<div>View and manage your team's {{camelEntity}}s</div>
			</Row>
			<Row gutter={16}>
				<Col span={8}>
					<SingleCard
						title="Total {{EntityName}}s"
						description={isPending ? 'Loading...' : totals.Total.toString()}
						icon={() => <Total{{EntityName}}Icon />}
					/>
				</Col>
			</Row>
			<TableContainer
				table={() => (
					<CustomTable
						data={tableData}
						onEdit={(record) => navigate(`/{{camelEntity}}s/info/${record.id}`)}
						onDelete={(record) => console.log(record)}
					/>
				)}
			/>
		</>
	);
};
```

## Key Conventions

- **List state**: Always local `useState` — NOT Redux (Redux only holds the single selected entity for the details page)
- **Data fetching**: `useApiCall` + `useTransition` — keeps UI responsive during fetch. Only destructure `execute` from `useApiCall` — **never use `data`** from the hook for list pages; store in local `useState` instead
- **Navigation**: `useNavigate` + `navigate(\`/{{camelEntity}}s/info/${record.id}\`)` on row edit
- **Summary counts**: Derived client-side from the local list state. Use `isPending ? 'Loading...' : count.toString()` for the card description
- **Loading state**: `isPending` from `useTransition` drives the loading text in summary cards — no `<Spin />` needed; the table renders empty while pending
- **Table data mapping**: Transform API shape to display-friendly shape in `tableData` array
- **Create flow**: `ButtonCard` → modal → API call → navigate to new entity's details page
- **No Redux dispatch here** — list data stays local; only entity details go to Redux in the details page
