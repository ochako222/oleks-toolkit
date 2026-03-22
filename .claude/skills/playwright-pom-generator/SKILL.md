---
name: playwright-pom-generator
description: >
  Generates high-quality Page Objects, Components, and Modals following project POM patterns
  and best practices. Use when creating new pages, modals, table components, navigation components,
  or any reusable UI element classes for the E2E test framework.
---

# Playwright POM Generator

Create production-ready Page Objects, Components, and Modals that follow the project's established architecture, coding standards, and design patterns for the SLA Admin E2E test suite.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating new Page Objects for application pages
- Building reusable Components (navigation, tables, forms)
- Creating Modal classes for dialogs and popups
- Extending existing Page Objects with new functionality
- Refactoring UI elements into reusable components
- Adding a new page to the AppPages registry
- Creating locators for new UI elements

**If the user mentions "page object", "component", "modal", "POM", "locator", or "AppPages" — use this skill.**

## Critical Safety Rules

**NEVER modify `abstractClasses.ts` without explicit user approval.**

This file defines the core `PageHolder` and `BasePage` abstract classes that every POM class inherits from. Changes here affect ALL existing Page Objects and Components and can break the entire test suite.

```
e2e/src/app/abstractClasses.ts    # PROTECTED - base classes
e2e/src/fixtues/baseTest.ts       # PROTECTED - test fixtures
e2e/src/misc/step.ts              # PROTECTED - step decorator
```

**NEVER delete or rename existing public methods** in Page Objects that are already used in tests — this will break those tests silently.

**What IS safe:**
- Creating new Page Object / Component / Modal files
- Adding new methods to existing classes
- Adding new locators (private) to existing classes
- Registering new pages in `AppPages` (`e2e/src/app/index.ts`)

## Architecture / Overview

The SLA Admin E2E framework uses a Page Object Model with composition. Pages own components, and all classes inherit from two abstract bases.

| Component | Purpose | Location |
|-----------|---------|----------|
| `PageHolder` | Abstract base providing `page` reference | `e2e/src/app/abstractClasses.ts` |
| `BasePage` | Abstract page class with `goto()`, `wait()`, `expectLoaded()` | `e2e/src/app/abstractClasses.ts` |
| `AppPages` | Registry of all page objects, injected via `baseTest` fixture | `e2e/src/app/index.ts` |
| `@step()` | Decorator wrapping methods as Playwright test steps (with `box: true`) | `e2e/src/misc/step.ts` |
| `baseTest` | Custom Playwright fixture providing `app` (AppPages) and `api` (SLAController) | `e2e/src/fixtues/baseTest.ts` |

### Class Hierarchy
```
PageHolder (abstract base — provides this.page)
  ├─ BasePage (abstract — adds goto(), wait(), expectLoaded(), expectPageLoaded())
  │   ├─ LoginPage
  │   ├─ MachinesManagementPage
  │   └─ UserManagementPage
  ├─ AppPages (registry — composes all pages)
  └─ Component Classes (reusable UI elements)
      ├─ LeftNavigationComponent
      ├─ TopNavigationComponent
      ├─ TableComponent
      └─ Modal Classes (LoginModal, UsersModal)
```

### Key Concepts
1. **PageHolder**: Base class providing `page` reference to all POM classes
2. **BasePage**: Abstract class for full pages — requires `expectLoaded()` implementation
3. **Components**: Lighter classes for reusable UI elements (inherit from `PageHolder` directly)
4. **Composition**: Pages compose components as public properties
5. **Encapsulation**: Locators are `private`, methods are `public`
6. **Step Decoration**: All public async methods use `@step()` decorator for Playwright reporting

## Operations

### Creating a Page Object

**Steps:**
1. Create file at `e2e/src/app/<featureName>.page.ts`
2. Extend `BasePage`
3. Define private locators
4. Compose public components
5. Implement `expectLoaded()` with `expect.soft()` assertions
6. Add action methods with `@step()` decorators
7. Register in `AppPages` at `e2e/src/app/index.ts`

**Template:**
```typescript
import { expect } from "@playwright/test";
import type { Page } from "@playwright/test";
import { BasePage } from "src/app/abstractClasses";
import { step } from "src/misc/step";
import { TopNavigationComponent } from "src/app/components/topNavigation.component";
import { LeftNavigationComponent } from "src/app/components/leftNavigation.component";
import { TableComponent } from "src/app/components/table.component";

export class FeatureNamePage extends BasePage {
	// ==========================================
	// LOCATORS (Private)
	// ==========================================
	private pageTitle = this.page.locator('[data-testid="page-title"]');
	private addButton = this.page.locator('button:has-text("Add")');

	// ==========================================
	// COMPONENTS (Public)
	// ==========================================
	topNavigation = new TopNavigationComponent(this.page);
	leftSideNavbar = new LeftNavigationComponent(this.page);
	table = new TableComponent(this.page);

	// ==========================================
	// PAGE VERIFICATION
	// ==========================================
	@step("Verify that Feature Name Page is loaded")
	async expectLoaded() {
		await expect.soft(this.pageTitle).toBeVisible();
		await expect.soft(this.addButton).toBeVisible();
	}

	// ==========================================
	// PAGE ACTIONS
	// ==========================================
	@step("Click on Add button")
	async clickOnAddButton() {
		await this.addButton.click();
	}
}
```

**Important notes:**
- `expectLoaded()` is abstract in `BasePage` — you MUST implement it
- Use `expect.soft()` in `expectLoaded()` (not `expect()`)
- Locators are declared as class properties, not inside methods
- Components are instantiated with `this.page`

### Creating a Component

**Steps:**
1. Create file at `e2e/src/app/components/<name>.component.ts`
2. Extend `PageHolder` (NOT `BasePage`)
3. Define private locators scoped to the component
4. Add action methods with `@step()` decorators

**Template:**
```typescript
import { expect } from "@playwright/test";
import type { Page } from "@playwright/test";
import { PageHolder } from "src/app/abstractClasses";
import { step } from "src/misc/step";

export class SomeComponent extends PageHolder {
	// ==========================================
	// LOCATORS (Private)
	// ==========================================
	private container = this.page.locator('[data-testid="component-container"]');

	// ==========================================
	// COMPONENT ACTIONS
	// ==========================================
	@step("Perform component action")
	async performAction() {
		await this.container.click();
	}
}
```

**Important notes:**
- Components extend `PageHolder`, NOT `BasePage`
- Components do NOT implement `expectLoaded()` (that's a `BasePage` concept)
- Components should be reusable across multiple pages

### Creating a Modal

**Steps:**
1. Create file at `e2e/src/app/components/<name>.modal.ts`
2. Extend `PageHolder`
3. Scope all locators within the modal context (`[role="dialog"]` or similar)
4. Include `getModalText()`, `confirm()`, `cancel()` methods

**Template:**
```typescript
import { expect } from "@playwright/test";
import type { Page } from "@playwright/test";
import { PageHolder } from "src/app/abstractClasses";
import { step } from "src/misc/step";

export class SomeModal extends PageHolder {
	// ==========================================
	// LOCATORS (Private)
	// ==========================================
	private modal = this.page.locator('[role="dialog"]');
	private modalText = this.page.locator('[role="dialog"] .modal-body');
	private confirmButton = this.page.locator('[role="dialog"] button.confirm');
	private cancelButton = this.page.locator('[role="dialog"] button.cancel');

	// Form fields (if modal has a form)
	private nameInput = this.page.locator('[role="dialog"] #name');

	// ==========================================
	// MODAL VERIFICATION
	// ==========================================
	@step("Get modal text")
	async getModalText() {
		return (await this.modalText.textContent()) || "";
	}

	@step("Verify modal is visible")
	async expectVisible() {
		await expect(this.modal).toBeVisible();
	}

	// ==========================================
	// MODAL ACTIONS
	// ==========================================
	@step("Confirm action")
	async confirm() {
		await this.confirmButton.click();
	}

	@step("Cancel action")
	async cancel() {
		await this.cancelButton.click();
	}

	// ==========================================
	// FORM ACTIONS (if applicable)
	// ==========================================
	@step("Fill modal form with parameters")
	async fillFormWithParameters(parameters: { name: string }) {
		await this.nameInput.fill(parameters.name);
		await this.confirmButton.click();
	}
}
```

### Registering in AppPages

After creating any new Page Object, register it in `e2e/src/app/index.ts`:

```typescript
import { YourNewPage } from "./yourNew.page.js";

export class AppPages extends PageHolder {
	// ... existing pages
	yourNewPage = new YourNewPage(this.page);
}
```

**Important notes:**
- Import must use `.js` extension (project uses CommonJS with TypeScript)
- Only Pages are registered — Components and Modals are composed within Pages

## Configuration / Key Locations

```
e2e/src/app/
├── abstractClasses.ts                    # PageHolder + BasePage abstract classes
├── index.ts                              # AppPages registry (all pages registered here)
├── login.page.ts                         # LoginPage
├── machinesManagement.page.ts            # MachinesManagementPage
├── usersManagement.page.ts               # UserManagementPage
├── support/
│   └── support.ts                        # Utility functions (generateRandomValue)
└── components/
    ├── leftNavigation.component.ts       # Left sidebar navigation
    ├── topNavigation.component.ts        # Top bar with logout
    ├── table.component.ts                # Reusable table interactions
    ├── login.modal.ts                    # Login error modal
    └── users.modal.ts                    # User CRUD modal
```

**Key behaviors:**
- All imports use `src/*` path aliases (configured in `tsconfig.json`)
- Biome enforces double quotes, tab indentation, and import organization
- The `@step()` decorator uses `box: true` for nested step reporting in Playwright traces

## Code Quality Standards

### TypeScript Standards
- **Strict Mode**: All code must pass strict type checking
- **No Any Types**: Use proper types for all parameters and return values
- **Async/Await Only**: Never use `.then()` chains
- **Double Quotes**: Use double quotes for all strings
- **Tab Indentation**: Use tabs (not spaces)

### Naming Conventions
- **Classes**: PascalCase (e.g., `LoginPage`, `TableComponent`, `UsersModal`)
- **Files**: camelCase with type suffix (e.g., `login.page.ts`, `table.component.ts`, `users.modal.ts`)
- **Methods**: camelCase, descriptive (e.g., `loginAsUser`, `selectModuleByName`)
- **Locators**: camelCase, descriptive (e.g., `userNameInput`, `submitButton`)
- **Private Members**: Use `private` keyword

### Locator Priority (Best to Worst)
1. `data-testid` (most stable)
2. Role and accessible name (`getByRole`)
3. `aria-label`
4. ID (`#username`)
5. CSS selector (fragile — avoid)
6. Text content (breaks with i18n — use sparingly)

### Step Decorator Best Practices
```typescript
// Descriptive messages
@step("Click on Submit button")
async clickOnSubmitButton() { ... }

// Parameter interpolation
@step("Login as user: {userName}")
async loginAsUser(userName: string, password: string) { ... }
```

## Troubleshooting

```bash
# Type check the e2e project
npx tsc --noEmit -p /home/magicalex/Desktop/dev/SLA-Admin/e2e/tsconfig.json

# Run Biome linter/formatter check
npx biome check /home/magicalex/Desktop/dev/SLA-Admin/e2e/src/app/

# Verify imports resolve correctly
npx tsc --traceResolution -p /home/magicalex/Desktop/dev/SLA-Admin/e2e/tsconfig.json 2>&1 | head -50
```

**Common issues:**
- **"Cannot find module"**: Use `src/*` path alias, not relative paths like `../../`
- **"Property does not exist on type BasePage"**: You're extending `PageHolder` instead of `BasePage`, or vice versa
- **"No overload matches this call"**: Check constructor — Components/Modals take `Page`, not other arguments
- **Missing from AppPages**: Forgot to register the new page in `e2e/src/app/index.ts`

## Decision Framework

When the user requests a new POM class:

1. **Is it a full application page with its own URL/route?** -> Create a Page (extend `BasePage`)
2. **Is it a reusable UI section appearing on multiple pages?** -> Create a Component (extend `PageHolder`)
3. **Is it a dialog/popup/overlay?** -> Create a Modal (extend `PageHolder`)
4. **Is it an existing page that needs new interactions?** -> Add methods to the existing Page Object
5. **Unsure?** -> Ask the user if the element is page-level or reusable, then decide

## Creation Checklists

### When Creating a Page Object
- [ ] Extends `BasePage`
- [ ] All locators are `private`
- [ ] Implements `expectLoaded()` with `expect.soft()` assertions
- [ ] Components instantiated with `this.page`
- [ ] All public methods decorated with `@step()`
- [ ] Proper TypeScript types (no `any`)
- [ ] Follows naming conventions (PascalCase class, camelCase file)
- [ ] Registered in `AppPages` in `index.ts`
- [ ] Imports use `src/*` path aliases

### When Creating a Component
- [ ] Extends `PageHolder` (NOT `BasePage`)
- [ ] All locators are `private`
- [ ] Reusable across multiple pages
- [ ] All public methods decorated with `@step()`
- [ ] Proper TypeScript types (no `any`)
- [ ] Follows naming conventions

### When Creating a Modal
- [ ] Extends `PageHolder`
- [ ] Has `getModalText()` method
- [ ] Has confirmation/cancel methods
- [ ] Scoped locators (within modal context)
- [ ] Form methods if modal contains forms
- [ ] All methods decorated with `@step()`

## Example Requests

- "Create a page object for the products page" -> Create `ProductsManagementPage` extending `BasePage`, register in `AppPages`
- "I need a modal for the delete confirmation dialog" -> Create a Modal class extending `PageHolder` with `getModalText()`, `confirm()`, `cancel()`
- "Make a reusable dropdown component" -> Create a Component extending `PageHolder` with dropdown interaction methods
- "Add a search method to the machines page" -> Add a new `@step()`-decorated method to the existing `MachinesManagementPage`
- "Register the new bottles page in the test framework" -> Import and add to `AppPages` in `e2e/src/app/index.ts`

---

**Remember**: Well-designed Page Objects make tests readable, maintainable, and resilient to UI changes. Invest time in creating quality POM classes for long-term test suite health.
