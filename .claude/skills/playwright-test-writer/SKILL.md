---
name: playwright-test-writer
description: >
  Generates high-quality Playwright E2E tests following POM patterns, code standards, and
  project conventions. Use when creating new test spec files, adding test cases to existing
  suites, writing smoke tests, or building data-driven serial test workflows.
---

# Playwright Test Writer

Generate production-ready Playwright E2E tests that follow the project's established patterns, coding standards, and Page Object Model architecture for the SLA Admin test suite.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Creating new test specifications (`*.spec.ts` files)
- Adding test cases to existing test suites
- Writing smoke tests (`@smoke` tagged tests)
- Creating serial test suites with related test scenarios
- Building data-driven tests with dynamic test data
- Writing CRUD test workflows (create/read/update/delete)
- Writing `beforeAll` hooks with API setup

**If the user mentions "write a test", "add a test case", "test spec", or "smoke test" — use this skill.**

## Critical Safety Rules

**NEVER hardcode test data that creates permanent state changes.**

Tests use `support.generateRandomValue()` to generate unique data. Hardcoded emails, names, or IDs cause test conflicts when running in parallel or repeatedly.

**NEVER:**
- Use `test.describe()` instead of `baseTest.describe.serial()` — this project requires serial execution
- Use Playwright's built-in `test` instead of the project's `baseTest` fixture
- Skip `expectLoaded()` calls after navigation — this causes flaky tests
- Leave `console.log()` or debug code in final test files
- Hardcode credentials or tokens — use the established patterns (`"test@admin.com"`, `"12345"`)

**What IS safe:**
- Creating new `*.spec.ts` files in `e2e/src/tests/`
- Using `baseTest.describe.serial()` for all test suites
- Using `app` and `api` fixtures from `baseTest`
- Tagging tests with `{ tag: "@smoke" }`

## Architecture / Overview

The SLA Admin E2E test framework is built on Playwright with custom fixtures and a Page Object Model.

| Component | Purpose | Location |
|-----------|---------|----------|
| `baseTest` | Custom Playwright fixture providing `app` + `api` | `e2e/src/fixtues/baseTest.ts` |
| `AppPages` | Registry of all Page Objects, accessed via `app` fixture | `e2e/src/app/index.ts` |
| `SLAController` | API client for test data setup, accessed via `api` fixture | `e2e/src/api/index.ts` |
| `support` | Utility functions (`generateRandomValue()`) | `e2e/src/app/support/support.ts` |
| Test specs | Test files using `baseTest.describe.serial()` | `e2e/src/tests/*.spec.ts` |

### Test Execution Flow
```
baseTest fixture setup
  ├─ app = new AppPages(page)     # UI interactions
  └─ api = new SLAController(request, baseURL)  # API calls
       ↓
beforeAll (optional API setup)
       ↓
Serial test execution (test 1 → test 2 → test 3...)
       ↓
Results: JUnit XML + HTML Report
```

### Available Page Objects (via `app`)
- `app.loginPage` — Login page with `loginAsUser()`, `expectLoaded()`
- `app.machinesManagementPage` — Machines list and detail pages
- `app.usersManagementPage` — User management with table, modals

### Available API Controllers (via `api`)
- `api.authController.loginAs(email, password)` — Get access token
- `api.machinesController.getAllMachines(token)` — Fetch machines
- `api.machinesController.getMachineById(token, id)` — Fetch machine details
- `api.machinesController.updateMachineById(token, data)` — Update machine

## Operations

### Writing a New Test Suite

**Steps:**
1. Create file at `e2e/src/tests/<featureName>.spec.ts`
2. Import `baseTest`, `expect`, and `support` (if needed)
3. Generate unique test data at the top of the file
4. Use `baseTest.describe.serial()` for the suite
5. Add `beforeAll` for API setup if needed
6. Write tests following AAA pattern (Arrange-Act-Assert)
7. Tag critical tests with `{ tag: "@smoke" }`

**Template — Basic Test Suite:**
```typescript
import { expect } from "@playwright/test";
import { support } from "src/app/support/support";
import { baseTest } from "src/fixtues/baseTest";

const randomValue = support.generateRandomValue();
const testData = {
	name: `Test Entity ${randomValue}`,
	email: `test${randomValue}@example.com`,
};

baseTest.describe.serial("Feature Name Tests", () => {
	baseTest(
		"Should perform expected action",
		{
			tag: "@smoke",
		},
		async ({ app }) => {
			// Arrange: Navigate and authenticate
			await app.loginPage.goto("/login");
			await app.loginPage.expectLoaded();
			await app.loginPage.loginAsUser("test@admin.com", "12345");

			// Act: Perform the action
			await app.machinesManagementPage.expectLoaded();

			// Assert: Verify the result
			await expect(
				await app.machinesManagementPage.table.getTableRowByName("item"),
			).toBeVisible();
		},
	);
});
```

**Important notes:**
- ALWAYS use `baseTest`, never Playwright's `test`
- ALWAYS use `.describe.serial()` — never parallel
- ALWAYS call `expectLoaded()` after any page navigation
- Test data with `support.generateRandomValue()` goes at file top, outside the suite

### Writing a Test with API Setup

**Template:**
```typescript
import { expect } from "@playwright/test";
import { baseTest } from "src/fixtues/baseTest";

baseTest.describe.serial("Feature Tests with API Setup", () => {
	baseTest.beforeAll(async ({ api }) => {
		const { access_token } = (
			await api.authController.loginAs("test@admin.com", "12345")
		).data;

		// Prepare test data via API
		const machines = (
			await api.machinesController.getAllMachines(access_token)
		).data;
	});

	baseTest(
		"Test using pre-configured data",
		async ({ app }) => {
			// Test implementation
		},
	);
});
```

### Writing Serial CRUD Tests (Shared State)

**Template:**
```typescript
import { expect } from "@playwright/test";
import { support } from "src/app/support/support";
import { baseTest } from "src/fixtues/baseTest";

const randomValue = support.generateRandomValue();
const testEntity = {
	name: `Test Entity ${randomValue}`,
	email: `test${randomValue}@example.com`,
	password: "12345",
};

baseTest.describe.serial("Entity CRUD Tests", () => {
	baseTest(
		"Should create new entity",
		{ tag: "@smoke" },
		async ({ app }) => {
			await app.loginPage.goto("/login");
			await app.loginPage.expectLoaded();
			await app.loginPage.loginAsUser("test@admin.com", "12345");
			// ... create entity
			await expect(
				await app.somePage.table.getTableRowByName(testEntity.name),
			).toBeVisible();
		},
	);

	baseTest(
		"Should update existing entity",
		async ({ app }) => {
			await app.loginPage.goto("/login");
			await app.loginPage.expectLoaded();
			await app.loginPage.loginAsUser("test@admin.com", "12345");
			// ... update entity created in previous test
		},
	);

	baseTest(
		"Should delete entity",
		{ tag: "@smoke" },
		async ({ app }) => {
			await app.loginPage.goto("/login");
			await app.loginPage.expectLoaded();
			await app.loginPage.loginAsUser("test@admin.com", "12345");
			// ... delete entity
			await expect(
				await app.somePage.table.getTableRowByName(testEntity.name),
			).not.toBeVisible();
		},
	);
});
```

**Important notes:**
- Serial execution guarantees test 2 runs after test 1
- Shared `testEntity` const allows later tests to reference data from earlier tests
- Each test still logs in independently (no shared browser state between tests)

## Configuration / Key Locations

```
e2e/
├── playwright.config.ts              # Test config (timeouts, retries, reporters)
├── biome.json                        # Code quality (double quotes, tabs, import sorting)
├── tsconfig.json                     # TypeScript config (src/* path aliases)
├── src/
│   ├── tests/                        # TEST FILES GO HERE
│   │   ├── usersTest.spec.ts         # User management CRUD tests
│   │   └── machinesManagement.spec.ts # Machine management tests
│   ├── fixtues/
│   │   └── baseTest.ts               # Custom fixtures (app, api)
│   └── app/
│       ├── support/support.ts        # generateRandomValue() utility
│       └── index.ts                  # AppPages registry
└── test-results/                     # Generated output
    ├── junit/                        # JUnit XML results
    └── html/                         # HTML report
```

**Test execution commands:**
```bash
npm run tests:ui          # Interactive UI mode (development)
npm run tests:single      # Smoke tests only (CI — @smoke tag)
npm run tests:all         # All tests (CI mode)
npx playwright test <file> --headed   # Single file, see browser
npx playwright test -g "test name"    # Run by test name pattern
npx playwright test --grep "@smoke"   # Run smoke tests
```

## Assertion Patterns

### Standard Assertions
```typescript
// Visibility
await expect(element).toBeVisible();
await expect(element).not.toBeVisible();

// Text (from Page Object method returning string)
await expect(await app.page.getModalText()).toBe("Expected Text");

// Element text (direct locator)
await expect(element).toHaveText("Expected Text");
```

### Soft Assertions (Page Objects only)
```typescript
// Used ONLY in expectLoaded() methods inside Page Objects
await expect.soft(this.pageHeader).toBeVisible();
```

## Common Tasks

### Standard Login Flow (used in nearly every test)
```typescript
await app.loginPage.goto("/login");
await app.loginPage.expectLoaded();
await app.loginPage.loginAsUser("test@admin.com", "12345");
await app.machinesManagementPage.expectLoaded(); // Landing page after login
```

### Module Navigation
```typescript
await app.machinesManagementPage.leftSideNavbar.selectModuleByName("User Management");
await app.usersManagementPage.expectLoaded();
```

### Logout Flow
```typescript
await app.currentPage.topNavigation.logout();
await app.loginPage.expectLoaded();
```

### Modal Verification
```typescript
await expect(
	await app.somePage.someModal.getModalText(),
).toBe("Expected modal message");
```

### Table Row Verification
```typescript
// Verify row exists
await expect(
	await app.somePage.table.getTableRowByName(entityName),
).toBeVisible();

// Verify row was removed
await expect(
	await app.somePage.table.getTableRowByName(entityName),
).not.toBeVisible();
```

## Troubleshooting

```bash
# Test file not found
# Ensure file is in e2e/src/tests/ and ends with .spec.ts

# "Cannot find module 'src/...'"
# Use path alias imports, not relative: import { baseTest } from "src/fixtues/baseTest"

# "Property does not exist on type"
# Check AppPages registry — is the page registered? Check method exists in Page Object.

# "Timeout waiting for element"
# Ensure expectLoaded() is called before interacting with page elements.

# Type check
npx tsc --noEmit

# Lint check
npx biome check e2e/src/tests/
```

## Decision Framework

When the user requests a new test:

1. **Is it a single standalone test?** -> Create a new `.spec.ts` with `baseTest.describe.serial()` containing one test
2. **Is it a multi-step workflow (CRUD)?** -> Create a serial suite with shared test data, one test per step
3. **Does it need API data setup?** -> Add a `beforeAll` hook using the `api` fixture
4. **Is it a critical path test?** -> Tag with `{ tag: "@smoke" }`
5. **Does the required Page Object exist?** -> If not, use the `playwright-pom-generator` skill first
6. **Unsure about test scope?** -> Ask the user what behavior they want to verify

## Code Quality Checklist

Before finalizing any test:
- [ ] Uses `baseTest.describe.serial()` (not `test.describe`)
- [ ] Imports from `src/*` path aliases
- [ ] Test data uses `support.generateRandomValue()` for uniqueness
- [ ] Follows AAA pattern (Arrange-Act-Assert)
- [ ] Calls `expectLoaded()` after every navigation
- [ ] Uses `await expect()` for all assertions
- [ ] Double quotes for strings
- [ ] Tab indentation
- [ ] No `any` types
- [ ] No `console.log()` in final code
- [ ] Descriptive test names
- [ ] Critical tests tagged `{ tag: "@smoke" }`

## Example Requests

- "Write a test for user login" -> Create a smoke test verifying login flow with `loginAsUser()` and `expectLoaded()`
- "Add a test for deleting a machine" -> Add a test to a serial suite that navigates to machine, deletes it, verifies removal
- "Create CRUD tests for products" -> Create a serial suite with create/update/delete tests using shared `randomValue` data
- "Write smoke tests for the main user flows" -> Create `@smoke` tagged tests covering login, navigation, and critical CRUD operations
- "Add an API setup hook to prepare test data" -> Add `beforeAll` with `api.authController.loginAs()` and API data preparation

---

**Remember**: Quality and readability are paramount. Write tests that are easy to understand, maintain, and debug. Every test should tell a clear story of what's being tested and why.
