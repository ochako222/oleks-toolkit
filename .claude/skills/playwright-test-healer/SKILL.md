---
name: playwright-test-healer
description: >
  Debugs and fixes failing Playwright E2E tests. Use when tests fail, timeout, produce
  unexpected results, or are flaky. Analyzes errors, diagnoses root causes in Page Objects
  and test code, provides targeted fixes, and maintains code quality.
---

# Playwright Test Healer

Diagnose and fix failing Playwright E2E tests while maintaining code quality, following project patterns, and preserving test intent for the SLA Admin E2E test suite.

## When This Skill MUST Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Tests are failing with errors or timeouts
- Tests are flaky (passing/failing intermittently)
- Tests produce unexpected results
- Locators are not finding elements
- Assertions are failing
- Navigation or page load issues
- Modal or component interaction failures
- User says "fix", "debug", "heal", "broken test", or "test is failing"

**If a test is failing or producing unexpected results — STOP and use this skill before making changes.**

## Critical Safety Rules

**NEVER make changes to fix a test without first reading and understanding the failing test file AND its Page Objects.**

Blind fixes (adding random waits, force-clicking, skipping assertions) mask real bugs and create technical debt.

**NEVER:**
- Add `{ force: true }` to clicks without understanding WHY the element is blocked
- Add `page.waitForTimeout()` with arbitrary values as a fix — find the proper wait condition
- Delete or weaken assertions to make a test pass — the assertion may be correct and the app may be broken
- Modify `abstractClasses.ts`, `baseTest.ts`, or `step.ts` without explicit user approval

**What IS safe:**
- Updating locator selectors in Page Objects
- Adding proper waits (`waitFor`, `waitForResponse`, `waitForLoadState`)
- Fixing incorrect expected values in assertions
- Adding missing `expectLoaded()` calls in test flows

## Architecture / Overview

The SLA Admin E2E framework has three layers where failures can originate:

| Layer | Files | Common Failure Types |
|-------|-------|---------------------|
| **Tests** | `e2e/src/tests/*.spec.ts` | Wrong expected values, missing awaits, bad test flow |
| **Page Objects** | `e2e/src/app/*.page.ts` | Outdated locators, missing methods, wrong selectors |
| **Components/Modals** | `e2e/src/app/components/*` | Scoping issues, wrong modal selectors, stale locators |
| **Fixtures** | `e2e/src/fixtues/baseTest.ts` | Fixture setup issues (rarely the problem) |
| **API Controllers** | `e2e/src/api/*.controller.ts` | API setup failures in `beforeAll` hooks |

### Key Framework Details
- **Execution**: All suites use `baseTest.describe.serial()` — tests run sequentially, earlier failures cascade
- **Timeouts**: Suite timeout 120s, expect timeout 30s (configured in `playwright.config.ts`)
- **Retries**: 1 retry configured — if first run fails, Playwright retries once
- **Fixtures**: `app` provides `AppPages` (UI), `api` provides `SLAController` (API)
- **Reporting**: JUnit XML + HTML report + list output

## Operations

### Debugging Workflow

**Step 1: Gather Information**

```bash
# Run the specific failing test to see the error
npx playwright test e2e/src/tests/<testFile>.spec.ts --headed

# Run with trace for detailed execution timeline
npx playwright test e2e/src/tests/<testFile>.spec.ts --trace on

# Run in debug mode for step-by-step analysis
npx playwright test e2e/src/tests/<testFile>.spec.ts --debug
```

**Read the error output carefully:**
- What is the error message?
- Which line/step is failing?
- What was the expected vs actual behavior?
- Is it a timeout, assertion failure, or element not found?

**Step 2: Analyze the Test File**
- Read the test structure and flow
- Check test data (hardcoded vs random)
- Identify which Page Object methods are called
- Check assertion expectations

**Step 3: Verify Page Objects**
- Are locators correct for the current UI?
- Is `expectLoaded()` checking the right elements?
- Are components properly composed with `this.page`?
- Do methods have proper `@step()` decorators?

**Step 4: Apply Fix + Verify**
- Make targeted fix (locator, wait, assertion, or logic)
- Run test again to verify
- Check for side effects on related tests

## Common Failure Patterns & Solutions

### Pattern 1: Timeout Waiting for Element

**Error:** `TimeoutError: locator.waitFor: Timeout 30000ms exceeded`

**Solutions:**

```typescript
// A) Locator selector changed in the UI
// BEFORE (broken)
private submitButton = this.page.locator("#submit");
// AFTER (fixed)
private submitButton = this.page.locator('button[type="submit"]');

// B) Element needs explicit wait
// BEFORE
await this.submitButton.click();
// AFTER
await this.submitButton.waitFor({ state: "visible" });
await this.submitButton.click();

// C) Page not fully loaded
// BEFORE
await this.page.goto("/dashboard");
// AFTER
await this.page.goto("/dashboard");
await this.page.waitForLoadState("networkidle");
```

### Pattern 2: Element Not Found

**Error:** `Error: Element not found` or `strict mode violation`

**Solutions:**

```typescript
// A) Selector syntax wrong
// BEFORE
private userName = this.page.locator(".user-name");
// AFTER
private userName = this.page.locator('[data-testid="user-name"]');

// B) Multiple matches — need more specific selector
// BEFORE (matches multiple rows)
async clickEditButton(name: string) {
	await this.page.locator(`tr:has-text("${name}") button`).click();
}
// AFTER (specific action button)
async clickEditButton(name: string) {
	await this.page.locator(`tr`).filter({ hasText: name })
		.locator('svg[data-icon="edit"]').click();
}
```

### Pattern 3: Assertion Failures

**Error:** `expect(received).toBe(expected)`

**Solutions:**

```typescript
// A) Timing issue — data not loaded when assertion runs
// BEFORE
await this.page.click("#save");
await expect(await this.getMessage()).toBe("Saved successfully");
// AFTER
await this.page.click("#save");
await this.page.waitForResponse(resp => resp.url().includes("/api/save"));
await expect(await this.getMessage()).toBe("Saved successfully");

// B) Promise not awaited
// BEFORE
await expect(this.getModalText()).toBe("Expected text");
// AFTER
await expect(await this.getModalText()).toBe("Expected text");

// C) Wrong expected value — UI text changed
// BEFORE
await expect(await modal.getText()).toBe("Delete user?");
// AFTER — verify actual text first, then update
await expect(await modal.getText()).toBe("Are you sure you want to delete this user?");
```

### Pattern 4: Flaky Tests (Intermittent Failures)

**Solutions:**

```typescript
// A) Race conditions — not waiting for async operations
// BEFORE
await this.page.click("#load-data");
await expect(await this.table.getRowCount()).toBe(5);
// AFTER
await this.page.click("#load-data");
await this.page.waitForResponse(resp => resp.url().includes("/api/data"));
await expect(await this.table.getRowCount()).toBe(5);

// B) Shared test data conflicts
// BEFORE
const testUser = { email: "test@example.com" };
// AFTER
const randomValue = support.generateRandomValue();
const testUser = { email: `test${randomValue}@example.com` };
```

### Pattern 5: Serial Suite Cascade Failures

**Symptom:** First test fails, all subsequent tests in the suite also fail.

**Root cause:** Serial suites share state. If test 1 doesn't complete its setup (e.g., fails to create a user), tests 2 and 3 that depend on that user will also fail.

**Solution:** Fix the FIRST failing test. The rest will likely pass once the root test works.

### Pattern 6: Modal/Dialog Issues

**Solutions:**

```typescript
// A) Modal not fully rendered before interaction
// BEFORE
await this.page.click("#delete-button");
await this.confirmModal.confirm();
// AFTER
await this.page.click("#delete-button");
await this.page.locator('[role="dialog"]').waitFor({ state: "visible" });
await this.confirmModal.confirm();
```

## Configuration / Key Locations

```
e2e/
├── playwright.config.ts              # Timeouts, retries, browser config
├── src/
│   ├── tests/
│   │   ├── usersTest.spec.ts         # User management tests
│   │   └── machinesManagement.spec.ts # Machine management tests
│   ├── app/
│   │   ├── abstractClasses.ts        # Base classes (PageHolder, BasePage)
│   │   ├── index.ts                  # AppPages registry
│   │   ├── *.page.ts                 # Page Objects
│   │   └── components/               # Components + Modals
│   ├── fixtues/baseTest.ts           # Test fixtures (app, api)
│   └── api/                          # API controllers for test setup
└── test-results/                     # Output: JUnit, HTML, traces
```

**Key config values (playwright.config.ts):**
- Suite timeout: `120000ms` (2 minutes)
- Expect timeout: `30000ms` (30 seconds)
- Retries: `1`
- Workers: `1` (sequential)
- Video: `on-first-retry`
- Screenshot: `only-on-failure`

## Troubleshooting

```bash
# Run single test file in headed mode to see what's happening
npx playwright test e2e/src/tests/usersTest.spec.ts --headed

# Run specific test by name
npx playwright test -g "should create user"

# Run with debug mode (step-by-step with inspector)
npx playwright test e2e/src/tests/usersTest.spec.ts --debug

# Generate and view trace
npx playwright test e2e/src/tests/usersTest.spec.ts --trace on
npx playwright show-trace test-results/trace.zip

# Run in UI mode (interactive)
npm run tests:ui

# Type check for compilation errors
npx tsc --noEmit

# Lint check
npx biome check e2e/src/
```

## Decision Framework

When a test is failing:

1. **Is it the FIRST test in a serial suite?** -> Fix this test first — cascade failures affect everything after it
2. **Is the error a timeout on a locator?** -> Check if the selector still matches the current UI, update the Page Object locator
3. **Is the error an assertion mismatch?** -> Verify the expected value is correct, check for timing issues (missing `await` or missing API wait)
4. **Does the test pass locally but fail in CI?** -> Look for race conditions, add proper waits instead of timeouts
5. **Does it fail intermittently?** -> Check for shared test data (use `generateRandomValue()`), check for animation timing
6. **Is it a `beforeAll` API setup failure?** -> Check API controller methods and auth token flow
7. **Unsure?** -> Run with `--trace on`, inspect the trace, and identify which step diverges from expected behavior

## Quick Reference: Error -> Solution

| Error Message | Likely Cause | Quick Fix |
|--------------|--------------|-----------|
| `Timeout waiting for element` | Element not found/visible | Update selector, add waitFor |
| `Element is not visible` | Element hidden or loading | Wait for visibility state |
| `strict mode violation` | Multiple elements match | Make selector more specific |
| `Navigation timeout` | Page not loading | Check URL, add waitForLoadState |
| `expect(received).toBe(expected)` | Wrong value or timing | Verify value, add API wait |
| `Cannot find module` | Import path wrong | Use `src/*` path alias |
| `Property does not exist` | Method not in Page Object | Check PO implementation |
| `locator.click: Target closed` | Page/tab closed unexpectedly | Check navigation flow |

## Example Requests

- "My user test is failing with a timeout" -> Read the test, identify the failing locator, check the Page Object, update the selector
- "Tests pass locally but fail in CI" -> Look for race conditions and timing issues, replace arbitrary waits with proper conditions
- "The delete confirmation test is broken" -> Check modal locators, verify `getModalText()` returns correct text, check scoping
- "All tests after the first one are failing" -> Fix the first test in the serial suite — cascade failures
- "Test is flaky — sometimes passes, sometimes fails" -> Check for shared test data and race conditions, use `generateRandomValue()`

---

**Remember**: The goal is not just to make tests pass, but to make them reliable, maintainable, and meaningful. Every fix should improve overall test quality. Fix root causes, not symptoms.
