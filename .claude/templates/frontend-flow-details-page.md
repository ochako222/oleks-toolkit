# Frontend Flow Details Page Template

Entity details page with tab navigation, edit/save mode control, and imperative refs to views. Based on `MachineDetailsPage` and `BottleDetailsPage`.

## Pattern: Details Page (full implementation)

```tsx
import { Row } from 'antd';
import { useEffect, useRef, useState } from 'react';
import { useDispatch } from 'react-redux';
import { useLocation, useParams } from 'react-router-dom';
import { api } from 'src/api';
import { useApiCall } from 'src/hooks/useApiCall';
import { CustomTabs } from '../components/BaseComponents/Tabs';
import {
	{{EntityName}}GeneralInfoView,
	type {{EntityName}}GeneralInfoViewHandle,
} from '../components/{{EntityName}}s/views/GeneralInfoView';
import { {{EntityName}}SecondView } from '../components/{{EntityName}}s/views/SecondView';
import { set{{EntityName}}Details } from '../store/{{camelEntity}}Slice';

const tabs = ['General Information', '{{SecondTabName}}'];

export const {{EntityName}}DetailsPage = () => {
	const location = useLocation();
	const [currentView, setView] = useState('General Information');
	const dispatch = useDispatch();
	const { id } = useParams();

	const [isEditMode, setIsEditMode] = useState(false);
	const [isSaving, setIsSaving] = useState(false);
	const generalInfoViewRef = useRef<{{EntityName}}GeneralInfoViewHandle>(null);

	// Edit / Save toggle handler — delegates save to the active view via ref
	const handleEditSave = async () => {
		if (!isEditMode) {
			setIsEditMode(true);
		} else {
			if (generalInfoViewRef.current) {
				setIsSaving(true);
				try {
					await generalInfoViewRef.current.save();
					setIsEditMode(false);
				} catch (error) {
					console.error('Error saving changes:', error);
				} finally {
					setIsSaving(false);
				}
			}
		}
	};

	// Render tab content
	const renderSelectedTabContent = (param: string) => {
		switch (param) {
			case 'General Information':
				return (
					<{{EntityName}}GeneralInfoView
						ref={generalInfoViewRef}
						isEditMode={isEditMode}
					/>
				);
			case '{{SecondTabName}}':
				return <{{EntityName}}SecondView />;
			default:
				return null;
		}
	};

	// Fetch entity details and populate Redux store
	const { execute: fetch{{EntityName}}Details } = useApiCall(
		api.{{camelEntity}}sService.get{{EntityName}}DetailsById
	);

	useEffect(() => {
		fetch{{EntityName}}Details(id as string).then((response) => {
			if (response?.data) {
				dispatch(set{{EntityName}}Details(response.data));
			}
		});
	}, [dispatch, id]);

	return (
		<>
			<Row className="layout-title">
				<h2>{{EntityName}} Dashboard</h2>
				<div>{`{{ENTITY_LABEL}} ID ${location.pathname.split(':').pop()}`}</div>
			</Row>

			<Row className="product-card-row">
				<CustomTabs
					currentView={currentView}
					setView={setView}
					tabs={tabs}
					actionButton={{
						visible: currentView === 'General Information',
						text: isEditMode ? 'Save Changes' : 'Edit Information',
						onClick: handleEditSave,
						loading: isSaving,
					}}
				/>
			</Row>

			{renderSelectedTabContent(currentView)}
		</>
	);
};
```

## Pattern: Details Page (backend pending — mocked Redux state)

When the BE is not ready, comment out the fetch and rely on the mocked `initialState` in the slice:

```tsx
export const {{EntityName}}DetailsPage = () => {
	const location = useLocation();
	const [currentView, setView] = useState('General Information');
	const dispatch = useDispatch();
	const { id } = useParams();

	const [isEditMode, setIsEditMode] = useState(false);
	const [isSaving, setIsSaving] = useState(false);
	const generalInfoViewRef = useRef<{{EntityName}}GeneralInfoViewHandle>(null);

	const handleEditSave = async () => {
		if (!isEditMode) {
			setIsEditMode(true);
		} else {
			if (generalInfoViewRef.current) {
				setIsSaving(true);
				try {
					await generalInfoViewRef.current.save();
					setIsEditMode(false);
				} catch (error) {
					console.error('Error saving changes:', error);
				} finally {
					setIsSaving(false);
				}
			}
		}
	};

	const renderSelectedTabContent = (param: string) => {
		switch (param) {
			case 'General Information':
				return (
					<{{EntityName}}GeneralInfoView
						ref={generalInfoViewRef}
						isEditMode={isEditMode}
					/>
				);
			// ... other tabs don't need ref or isEditMode unless they have editable content
			default:
				return null;
		}
	};

	//TODO: Uncomment when BE is implemented
	// const { execute: fetch{{EntityName}}Details } = useApiCall(
	//   api.{{camelEntity}}sService.get{{EntityName}}DetailsById
	// );
	// useEffect(() => {
	//   fetch{{EntityName}}Details(id as string).then((response) => {
	//     if (response?.data) {
	//       dispatch(set{{EntityName}}Details(response.data));
	//     }
	//   });
	// }, [dispatch, id]);

	return (
		<>
			<Row className="layout-title">
				<h2>{{EntityName}} Dashboard</h2>
				<div>{`{{ENTITY_LABEL}} ID ${location.pathname.split(':').pop()}`}</div>
			</Row>

			<Row className="product-card-row">
				<CustomTabs
					currentView={currentView}
					setView={setView}
					tabs={tabs}
					actionButton={{
						visible: currentView === 'General Information',
						text: isEditMode ? 'Save Changes' : 'Edit Information',
						onClick: handleEditSave,
						loading: isSaving,
					}}
				/>
			</Row>

			{renderSelectedTabContent(currentView)}
		</>
	);
};
```

## Key Conventions

- **`isEditMode` / `isSaving`**: Controlled at page level, passed as `isEditMode` prop into the active view
- **`useRef<ViewHandle>`**: Points to the editable view (typically `GeneralInfoView`). Calls `.save()` on it imperatively
- **`handleEditSave`**: Toggle pattern — first click enters edit mode; second click triggers `ref.save()`, then exits on success
- **`renderSelectedTabContent`**: Switch statement mapping tab labels to view components
- **`actionButton.visible`**: Scoped to the "General Information" tab (or whichever tab has editable content)
- **Dispatch on fetch**: `dispatch(set{{EntityName}}Details(response.data))` — populates Redux store, not local state
- **`useLocation`**: Used to extract entity ID from pathname (supports `:id` style routes)
- **Tab refs**: Only the first (editable) tab view gets a `ref`. Read-only views don't need refs
