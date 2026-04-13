---
name: e2e
description: Generate Playwright E2E tests from a business spec's Gherkin scenarios. Use after /spec and before or during TDD implementation.
---

# /e2e — Spec-Driven E2E Generator

Convert Gherkin scenarios in `specs/NNN-*.spec.md` into runnable Playwright tests.

## When to use

- After `/spec` has produced a business spec with Gherkin scenarios
- When adding new user-facing flows
- When the existing E2E coverage lags behind the spec

## Detect Playwright

```bash
test -f playwright.config.ts || test -f playwright.config.js
```

If missing, tell user to run `npx playwright install` and init config.

## Steps

### Step 1: Read the spec

Parse `specs/NNN-*.spec.md`. Extract:
- Spec ID (NNN)
- Each `### Scenario:` block
- Given / When / Then bullets within each scenario

### Step 2: Generate test file

Create `tests/e2e/NNN-<kebab-title>.spec.ts`:

```typescript
import { test, expect } from "@playwright/test";

/** @spec NNN */
test.describe("NNN: <title>", () => {
  test("Scenario: <scenario name>", async ({ page }) => {
    // Given <precondition>
    await page.goto("/");
    // TODO: set up precondition

    // When <action>
    // TODO: perform action

    // Then <expected>
    // TODO: assert outcome
    await expect(page).toHaveTitle(/.*/);
  });
});
```

Every scenario becomes one `test()`. Every Given/When/Then becomes an inline comment followed by a TODO marker. Do not invent selectors — leave TODOs for the developer to fill in during TDD.

### Step 3: Suggest selector strategy

Output a reminder:

- Prefer **user-visible** locators: `getByRole`, `getByLabel`, `getByText`
- Avoid CSS selectors tied to class names or DOM structure
- Use `data-testid` only as last resort, and tag with `/** @spec NNN */` comment near the React element

### Step 4: Run skeleton

```bash
npx playwright test tests/e2e/NNN-*.spec.ts --reporter=list
```

Expected: tests run, pass trivially (since TODOs are empty asserts) or fail with clear "TODO" message. This is RED for TDD.

### Step 5: Output

```
E2E skeleton generated:
  🎭 tests/e2e/NNN-<name>.spec.ts
  X scenarios from specs/NNN-*.spec.md

Next: fill in Given/When/Then TODOs during TDD implementation.
```

## Rules

- One test file per spec (NNN).
- Every `test.describe` and every `test` MUST have `/** @spec NNN */` tag.
- Never write implementation — only the test skeleton.
- Never invent selectors or routes — leave TODOs.
- If the spec has `Non-Functional Requirements → Accessibility`, append an axe assertion stub using `@axe-core/playwright`.
