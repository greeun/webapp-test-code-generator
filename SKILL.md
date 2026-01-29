---
name: webapp-test-code-generator
description: |
  Generate executable test code from webapp-test-docs-writer test documents. Converts TC-* test cases to Playwright/Jest test files with proper assertions, fixtures, and test data setup.
  Triggers: "test code", "implement tests", "generate tests", "테스트 코드 생성", "테스트 구현", "TC-* 구현"
---

# Webapp Test Code Generator

Generate executable test code from webapp-test-docs-writer test documents.

## Language Detection

Detect user's language and respond accordingly.
- English request → English comments/output
- Korean request → Korean comments/output

## Workflow

```
1. Parse Test Doc → 2. Detect Framework → 3. Generate Code → 4. Write Files
```

## Step 1: Parse Test Document

Read the test document (Markdown file or context) and extract:

| Element | Source | Usage |
|---------|--------|-------|
| Test ID | `TC-[TYPE]-[NNN]` | Test name |
| Title | `## TC-*: [Title]` | describe/test name |
| Preconditions | Preconditions section | beforeEach setup |
| Test Steps | Steps table | Test actions |
| Expected Results | Expected column | Assertions |
| Test Data | JSON block | Test fixtures |
| Test Type | TYPE in ID | File location |

### Expected Input Format

```markdown
## TC-API-001: User login with valid credentials

| Item | Content |
|------|---------|
| **Priority** | Critical |
| **Preconditions** | - User exists in DB<br>- API server running |
| **Test Data** | `{"email": "test@example.com", "password": "Pass123!"}` |

### Test Steps
| # | Step | Expected Result |
|---|------|-----------------|
| 1 | POST /api/auth/login with credentials | 200 OK |
| 2 | Check response body | Contains accessToken |
| 3 | Verify token validity | Token is valid JWT |
```

## Step 2: Detect Framework

Check project for test framework configuration:

| Signal | Framework | File Pattern |
|--------|-----------|--------------|
| `playwright.config.ts` | Playwright | `*.spec.ts` |
| `jest.config.*` | Jest | `*.test.ts` |
| `vitest.config.*` | Vitest | `*.test.ts` |
| TC-E2E-*, TC-API-* | Playwright | E2E/API tests |
| TC-UNIT-*, TC-INT-* | Jest/Vitest | Unit/Integration |

**Default mapping:**
- `E2E`, `API`, `SEC` → Playwright
- `UNIT`, `INT` → Jest (or Vitest if configured)

## Step 3: Generate Code

### Playwright Template (E2E/API)

```typescript
import { test, expect } from '@playwright/test';

test.describe('[Feature] - [Scenario]', () => {
  test.beforeEach(async ({ page }) => {
    // Preconditions setup
  });

  test('TC-API-001: [Title]', async ({ request }) => {
    // Step 1: [Action]
    const response = await request.post('/api/auth/login', {
      data: { email: 'test@example.com', password: 'Pass123!' }
    });

    // Expected: 200 OK
    expect(response.status()).toBe(200);

    // Step 2: Check response body
    const body = await response.json();
    expect(body).toHaveProperty('accessToken');
  });
});
```

### Jest Template (Unit/Integration)

```typescript
import { describe, test, expect, beforeEach } from '@jest/globals';
// or: import { describe, test, expect, beforeEach } from 'vitest';

describe('[Feature] - [Scenario]', () => {
  beforeEach(() => {
    // Preconditions setup
  });

  test('TC-UNIT-001: [Title]', () => {
    // Test implementation
  });
});
```

### Code Generation Rules

1. **Test ID → Test Name**: `TC-API-001` → `test('TC-API-001: ...')`
2. **Preconditions → beforeEach**: Setup code in beforeEach hook
3. **Test Steps → Sequential actions**: One step = one action block
4. **Expected Results → expect()**: Each expected result = assertion
5. **Test Data → Fixtures**: Inline data or separate fixture file

### Mapping Test Steps to Code

| Step Pattern | Playwright | Jest |
|-------------|------------|------|
| Navigate to [URL] | `page.goto(url)` | N/A |
| Click [element] | `page.click(selector)` | `fireEvent.click()` |
| Enter [value] in [field] | `page.fill(selector, value)` | `userEvent.type()` |
| POST/GET/PUT/DELETE [endpoint] | `request.post/get/put/delete()` | `fetch()` / supertest |
| Check [condition] | `expect(...).toBe/toHave()` | `expect(...).toBe/toEqual()` |
| Wait for [element/response] | `page.waitFor*()` | `waitFor()` |

## Step 4: Write Files

### File Naming Convention

| Test Type | Directory | File Name |
|-----------|-----------|-----------|
| E2E | `tests/e2e/` or `e2e/` | `[feature].spec.ts` |
| API | `tests/api/` | `[feature].api.spec.ts` |
| Unit | `tests/unit/` or `__tests__/` | `[feature].test.ts` |
| Integration | `tests/integration/` | `[feature].integration.test.ts` |

### Output Structure

```
tests/
├── e2e/
│   └── login.spec.ts          # TC-E2E-* tests
├── api/
│   └── auth.api.spec.ts       # TC-API-* tests
├── unit/
│   └── validators.test.ts     # TC-UNIT-* tests
└── fixtures/
    └── test-data.json         # Shared test data
```

## Quick Reference

### Assertion Mapping

| Expected Result Pattern | Playwright | Jest |
|------------------------|------------|------|
| Status code 200 | `expect(response.status()).toBe(200)` | `expect(res.status).toBe(200)` |
| Contains [text] | `expect(page.locator(...)).toContainText()` | `expect(text).toContain()` |
| Element visible | `expect(locator).toBeVisible()` | `expect(el).toBeInTheDocument()` |
| Redirect to [URL] | `expect(page.url()).toBe(url)` | N/A |
| Response has [field] | `expect(body).toHaveProperty(field)` | `expect(obj).toHaveProperty()` |
| Error message shown | `expect(locator).toHaveText(msg)` | `expect(screen.getByText())` |

### Common Patterns

**API Test with Auth:**
```typescript
test('TC-API-002: Authenticated request', async ({ request }) => {
  // Login first
  const loginRes = await request.post('/api/auth/login', {
    data: { email: 'user@test.com', password: 'pass' }
  });
  const { accessToken } = await loginRes.json();

  // Authenticated request
  const response = await request.get('/api/protected', {
    headers: { Authorization: `Bearer ${accessToken}` }
  });
  expect(response.ok()).toBeTruthy();
});
```

**E2E Form Submission:**
```typescript
test('TC-E2E-001: Submit contact form', async ({ page }) => {
  await page.goto('/contact');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="message"]', 'Hello');
  await page.click('button[type="submit"]');
  await expect(page.locator('.success')).toBeVisible();
});
```

## Execution

After generating test files, provide run commands:

```bash
# Playwright
npx playwright test [file]

# Jest
npm test -- [file]

# Vitest
npx vitest run [file]
```
