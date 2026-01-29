---
name: webapp-test-code-generator
description: |
  Generate executable test code from webapp-test-docs-writer test documents. Converts TC-* test cases to Playwright/Jest test files with proper assertions, fixtures, and test data setup.
  webapp-test-docs-writer 테스트 문서에서 실행 가능한 테스트 코드 생성. TC-* 테스트 케이스를 Playwright/Jest 테스트 파일로 변환.
  Triggers: "테스트 코드 생성", "테스트 구현", "test code", "implement tests", "generate tests", "TC-* 구현", "테스트 문서로 코드 작성"
---

# Webapp Test Code Generator

Generate executable test code from webapp-test-docs-writer test documents.
webapp-test-docs-writer 테스트 문서에서 실행 가능한 테스트 코드 생성.

## Language / 언어

Detect user's language and respond accordingly.
- English request → English comments/output
- 한글 요청 → 한글 주석/출력

## Workflow

```
1. Parse Test Doc → 2. Detect Framework → 3. Generate Code → 4. Write Files
   문서 파싱          프레임워크 감지        코드 생성          파일 작성
```

## Step 1: Parse Test Document / 문서 파싱

Read the test document (Markdown file or context) and extract:
테스트 문서를 읽고 다음을 추출:

| Element / 요소 | Source / 출처 | Usage / 용도 |
|---------------|---------------|--------------|
| Test ID | `TC-[TYPE]-[NNN]` | Test name / 테스트명 |
| Title | `## TC-*: [Title]` | describe/test name |
| Preconditions | Preconditions section | beforeEach setup |
| Test Steps | Steps table | Test actions |
| Expected Results | Expected column | Assertions |
| Test Data | JSON block | Test fixtures |
| Test Type | TYPE in ID | File location |

### Expected Input Format / 입력 형식

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

## Step 2: Detect Framework / 프레임워크 감지

Check project for test framework configuration:
프로젝트에서 테스트 프레임워크 설정 확인:

| Signal / 신호 | Framework | File Pattern |
|--------------|-----------|--------------|
| `playwright.config.ts` | Playwright | `*.spec.ts` |
| `jest.config.*` | Jest | `*.test.ts` |
| `vitest.config.*` | Vitest | `*.test.ts` |
| TC-E2E-*, TC-API-* | Playwright | E2E/API tests |
| TC-UNIT-*, TC-INT-* | Jest/Vitest | Unit/Integration |

**Default mapping / 기본 매핑:**
- `E2E`, `API`, `SEC` → Playwright
- `UNIT`, `INT` → Jest (or Vitest if configured)

## Step 3: Generate Code / 코드 생성

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

### Code Generation Rules / 코드 생성 규칙

1. **Test ID → Test Name**: `TC-API-001` → `test('TC-API-001: ...')`
2. **Preconditions → beforeEach**: Setup code in beforeEach hook
3. **Test Steps → Sequential actions**: One step = one action block
4. **Expected Results → expect()**: Each expected result = assertion
5. **Test Data → Fixtures**: Inline data or separate fixture file

### Mapping Test Steps to Code / 단계-코드 매핑

| Step Pattern | Playwright | Jest |
|-------------|------------|------|
| Navigate to [URL] | `page.goto(url)` | N/A |
| Click [element] | `page.click(selector)` | `fireEvent.click()` |
| Enter [value] in [field] | `page.fill(selector, value)` | `userEvent.type()` |
| POST/GET/PUT/DELETE [endpoint] | `request.post/get/put/delete()` | `fetch()` / supertest |
| Check [condition] | `expect(...).toBe/toHave()` | `expect(...).toBe/toEqual()` |
| Wait for [element/response] | `page.waitFor*()` | `waitFor()` |

## Step 4: Write Files / 파일 작성

### File Naming Convention / 파일 명명 규칙

| Test Type | Directory | File Name |
|-----------|-----------|-----------|
| E2E | `tests/e2e/` or `e2e/` | `[feature].spec.ts` |
| API | `tests/api/` | `[feature].api.spec.ts` |
| Unit | `tests/unit/` or `__tests__/` | `[feature].test.ts` |
| Integration | `tests/integration/` | `[feature].integration.test.ts` |

### Output Structure / 출력 구조

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

## Quick Reference / 빠른 참조

### Assertion Mapping / 검증 매핑

| Expected Result Pattern | Playwright | Jest |
|------------------------|------------|------|
| Status code 200 | `expect(response.status()).toBe(200)` | `expect(res.status).toBe(200)` |
| Contains [text] | `expect(page.locator(...)).toContainText()` | `expect(text).toContain()` |
| Element visible | `expect(locator).toBeVisible()` | `expect(el).toBeInTheDocument()` |
| Redirect to [URL] | `expect(page.url()).toBe(url)` | N/A |
| Response has [field] | `expect(body).toHaveProperty(field)` | `expect(obj).toHaveProperty()` |
| Error message shown | `expect(locator).toHaveText(msg)` | `expect(screen.getByText())` |

### Common Patterns / 공통 패턴

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

## Execution / 실행

After generating test files, provide run commands:
테스트 파일 생성 후 실행 명령어 제공:

```bash
# Playwright
npx playwright test [file]

# Jest
npm test -- [file]

# Vitest
npx vitest run [file]
```
