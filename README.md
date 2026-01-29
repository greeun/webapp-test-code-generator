# webapp-test-code-generator

webapp-test-docs-writer로 작성된 테스트 문서를 실행 가능한 Playwright/Jest 테스트 코드로 변환합니다.

## 개요

```
┌─────────────────────────┐      ┌──────────────────────────┐      ┌─────────────────────┐
│  webapp-test-docs-writer │  →  │ webapp-test-code-generator │  →  │   실행 가능한 테스트   │
│  (테스트 문서 작성)        │      │   (테스트 코드 생성)        │      │   코드 파일           │
└─────────────────────────┘      └──────────────────────────┘      └─────────────────────┘
```

## 설치

```bash
# 심볼릭 링크 생성 (이미 완료된 경우 생략)
ln -sf "$(pwd)/webapp-test-code-generator" ~/.claude/skills/webapp-test-code-generator

# 확인
ls -la ~/.claude/skills/webapp-test-code-generator
```

## 사용법

### 기본 사용

```
테스트 문서 tests/docs/login-test-cases.md를 기반으로 테스트 코드 생성해줘
```

```
implement tests from tests/docs/api-test-cases.md
```

### 트리거 키워드

| 한글 | English |
|------|---------|
| 테스트 코드 생성 | test code |
| 테스트 구현 | implement tests |
| TC-* 구현 | generate tests |
| 테스트 문서로 코드 작성 | - |

## 지원 테스트 유형

| 테스트 ID | 유형 | 프레임워크 | 출력 파일 |
|----------|------|-----------|----------|
| TC-E2E-* | E2E | Playwright | `*.spec.ts` |
| TC-API-* | API | Playwright | `*.api.spec.ts` |
| TC-UNIT-* | Unit | Jest/Vitest | `*.test.ts` |
| TC-INT-* | Integration | Jest/Vitest | `*.integration.test.ts` |
| TC-SEC-* | Security | Playwright | `*.security.spec.ts` |

## 입력 형식

webapp-test-docs-writer가 생성하는 표준 형식:

```markdown
## TC-API-001: 유효한 자격증명으로 로그인

| Item / 항목 | Content / 내용 |
|-------------|----------------|
| **Priority / 우선순위** | Critical |
| **Preconditions / 전제조건** | - 사용자가 DB에 존재<br>- API 서버 실행 중 |
| **Test Data / 테스트 데이터** | `{"email": "test@example.com", "password": "Pass123!"}` |

### Test Steps / 테스트 단계

| # | Step / 단계 | Expected Result / 예상 결과 |
|---|-------------|----------------------------|
| 1 | POST /api/auth/login with credentials | 200 OK |
| 2 | Check response body | Contains accessToken |
| 3 | Verify token validity | Token is valid JWT |
```

## 출력 예시

### API 테스트 (Playwright)

```typescript
import { test, expect } from '@playwright/test';

test.describe('Login - 정상 로그인', () => {
  test('TC-API-001: 유효한 자격증명으로 로그인', async ({ request }) => {
    const credentials = {
      email: 'test@example.com',
      password: 'Pass123!'
    };

    // Step 1: POST /api/auth/login
    const response = await request.post('/api/auth/login', {
      data: credentials
    });
    expect(response.status()).toBe(200);

    // Step 2: Check response body
    const body = await response.json();
    expect(body).toHaveProperty('accessToken');

    // Step 3: Verify token validity
    expect(body.accessToken.split('.').length).toBe(3);
  });
});
```

### E2E 테스트 (Playwright)

```typescript
import { test, expect } from '@playwright/test';

test.describe('Login Page - E2E', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('TC-E2E-001: 정상 로그인 후 메인 페이지 이동', async ({ page }) => {
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'Pass123!');
    await page.click('button[type="submit"]');
    await expect(page).toHaveURL('/main');
  });
});
```

### Unit 테스트 (Jest)

```typescript
import { describe, test, expect } from '@jest/globals';
import { validateEmail } from '../utils/validators';

describe('Email Validator', () => {
  test('TC-UNIT-001: 유효한 이메일 형식 검증', () => {
    expect(validateEmail('test@example.com')).toBe(true);
  });

  test('TC-UNIT-002: 잘못된 이메일 형식 검증', () => {
    expect(validateEmail('invalid-email')).toBe(false);
  });
});
```

## 프레임워크 자동 감지

프로젝트 설정 파일로 프레임워크를 자동 감지합니다:

| 설정 파일 | 프레임워크 |
|----------|-----------|
| `playwright.config.ts` | Playwright |
| `jest.config.js/ts` | Jest |
| `vitest.config.ts` | Vitest |

## 파일 구조

```
project/
├── tests/
│   ├── docs/                    # 테스트 문서 (입력)
│   │   └── login-test-cases.md
│   ├── e2e/                     # E2E 테스트
│   │   └── login.spec.ts
│   ├── api/                     # API 테스트
│   │   └── auth.api.spec.ts
│   ├── unit/                    # 단위 테스트
│   │   └── validators.test.ts
│   └── integration/             # 통합 테스트
│       └── auth.integration.test.ts
└── playwright.config.ts
```

## 실행 명령어

```bash
# Playwright
npx playwright test tests/e2e/login.spec.ts

# Jest
npm test -- tests/unit/validators.test.ts

# Vitest
npx vitest run tests/unit/validators.test.ts
```

## 워크플로우 예시

```bash
# 1. 테스트 문서 작성
"로그인 기능 테스트 케이스 작성해줘"
# → webapp-test-docs-writer 활성화
# → tests/docs/login-test-cases.md 생성

# 2. 테스트 코드 생성
"작성된 테스트 문서로 테스트 코드 생성해줘"
# → webapp-test-code-generator 활성화
# → tests/e2e/login.spec.ts 생성

# 3. 테스트 실행
npx playwright test tests/e2e/login.spec.ts
```

## 관련 스킬

| 스킬 | 역할 |
|------|------|
| webapp-test-docs-writer | 테스트 시나리오/케이스 문서 작성 |
| **webapp-test-code-generator** | 테스트 코드 생성 (현재 스킬) |
| smart-test-runner | 테스트 실행 및 실패 재시도 |
| webapp-testing | Playwright 기반 웹앱 테스트 |

## 팁

1. **테스트 문서 품질**: 명확한 전제조건, 구체적인 테스트 데이터(JSON), 단계별 예상 결과 포함
2. **커스터마이징**: 생성된 코드에 인증 헬퍼, 공통 fixture, 환경별 설정 추가 가능
3. **연속 사용**: webapp-test-docs-writer → webapp-test-code-generator 순서로 사용
