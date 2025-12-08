---
name: e2e-test
description: PR 전 브랜치 변경사항을 분석하고, 사용자 요구사항을 결합하여 Playwright Test Agents로 E2E 테스트를 자동 생성 및 실행합니다. Pull Request 생성 전 UI 테스트가 필요할 때 사용하세요.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebFetch
---

# E2E Test Automation with Playwright Test Agents

이 스킬은 Git 브랜치의 변경사항을 분석하고, Playwright Test Agents를 활용하여 자동으로 E2E 테스트를 생성하고 실행합니다.

## Workflow Overview

1. **변경사항 분석**: Git diff를 통해 변경된 파일과 코드 파악
2. **요구사항 수집**: 사용자로부터 테스트 범위 및 추가 요구사항 확인
3. **Playwright 설정 확인**: 프로젝트의 Playwright 설정 상태 확인
4. **테스트 계획 생성**: Playwright Planner Agent로 마크다운 테스트 계획 작성
5. **테스트 코드 생성**: Generator Agent로 실행 가능한 테스트 코드 생성
6. **테스트 실행**: 생성된 테스트 실행 및 결과 리포트
7. **자동 수정**: 실패한 테스트가 있을 경우 Healer Agent로 자동 수정

## Instructions

### Step 1: Git 변경사항 분석

먼저 현재 브랜치와 base 브랜치(보통 main/master) 사이의 변경사항을 파악합니다:

```bash
# 현재 브랜치 확인
git branch --show-current

# main/master와의 diff 확인
git diff main...HEAD --name-only
git diff main...HEAD --stat
```

변경된 파일들을 분석하여:
- **프론트엔드 파일** (`.tsx`, `.jsx`, `.vue`, `.svelte` 등) 식별
- **라우팅 변경사항** 확인
- **새로운 페이지/컴포넌트** 추가 여부
- **UI 로직 변경** 범위 파악

### Step 2: 사용자와 대화형 요구사항 수집

변경사항 분석 결과를 사용자에게 보여주고, 다음 정보를 수집합니다:

**질문 리스트:**
1. "다음 변경사항을 감지했습니다: [요약]. 어떤 영역의 E2E 테스트를 생성할까요?"
2. "특정 사용자 플로우를 테스트하고 싶으신가요? (예: 로그인 → 대시보드 → 설정)"
3. "회귀 테스트도 포함할까요? (기존 기능이 여전히 작동하는지 확인)"
4. "테스트할 브라우저 환경은? (Chrome, Firefox, Safari, Mobile 등)"
5. "애플리케이션 시작 URL은 무엇인가요? (예: http://localhost:3000)"

### Step 3: Playwright 설정 확인

프로젝트에 Playwright가 설정되어 있는지 확인합니다:

```bash
# package.json에서 playwright 의존성 확인
cat package.json | grep playwright

# playwright.config.ts/js 존재 확인
ls -la | grep playwright.config
```

**Playwright가 없는 경우:**
```bash
# Playwright 설치
npm init playwright@latest

# Test Agents 초기화
npx playwright init-agents --loop=vscode
```

**Playwright가 이미 있는 경우:**
```bash
# Test Agents만 추가
npx playwright init-agents --loop=vscode
```

### Step 4: 테스트 계획 생성 (Planner Agent)

변경사항과 사용자 요구사항을 바탕으로 마크다운 테스트 계획을 작성합니다.

**계획 파일 생성:**
`specs/`로 디렉토리를 만들고 테스트 계획을 마크다운으로 작성:

```markdown
# [Feature Name] E2E Test Plan

## Test Scope
- Changed files: [list]
- User flows to test: [list]
- Regression tests: [yes/no]

## Test Cases

### Test Case 1: [Name]
**Given:** User is on [page]
**When:** User [action]
**Then:** [expected result]

### Test Case 2: [Name]
...
```

**예시:**
```markdown
# User Profile Update E2E Test Plan

## Test Scope
- Changed files: UserProfile.tsx, ProfileAPI.ts
- User flows: Login → Profile → Update → Verify
- Regression: Check existing profile display

## Test Cases

### Test Case 1: Profile Update Success
**Given:** User is logged in and on profile page
**When:** User updates name and email
**Then:**
- Profile is updated successfully
- Success message is displayed
- New data persists after page reload

### Test Case 2: Validation Error Handling
**Given:** User is on profile page
**When:** User enters invalid email format
**Then:**
- Validation error message appears
- Form is not submitted
- User can correct and retry
```

specs/ 디렉토리에 `[feature-name].spec.md` 형식으로 저장합니다.

### Step 5: 테스트 코드 생성 (Generator Agent)

마크다운 계획을 실행 가능한 Playwright 테스트 코드로 변환합니다.

**Playwright Generator 실행:**

Generator Agent는 `.github/agents/generator.mjs`로 정의되어 있습니다. 이를 실행하여 테스트 코드를 생성합니다:

```bash
# Generator Agent 실행 (마크다운 → Playwright 코드)
# 이는 일반적으로 IDE 통합으로 실행되지만, 수동으로도 가능합니다
node .github/agents/generator.mjs
```

또는 직접 테스트 코드를 작성할 수도 있습니다. specs/ 내용을 참고하여 tests/ 디렉토리에 Playwright 테스트를 생성:

```typescript
// tests/user-profile-update.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Profile Update', () => {
  test('should update profile successfully', async ({ page }) => {
    // Given: User is logged in
    await page.goto('http://localhost:3000/login');
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-button"]');

    // Navigate to profile
    await page.goto('http://localhost:3000/profile');

    // When: User updates profile
    await page.fill('[data-testid="name"]', 'New Name');
    await page.fill('[data-testid="email"]', 'newemail@example.com');
    await page.click('[data-testid="save-button"]');

    // Then: Success message appears
    await expect(page.locator('[data-testid="success-message"]')).toBeVisible();

    // Reload and verify persistence
    await page.reload();
    await expect(page.locator('[data-testid="name"]')).toHaveValue('New Name');
  });

  test('should show validation error for invalid email', async ({ page }) => {
    await page.goto('http://localhost:3000/profile');

    // When: Invalid email
    await page.fill('[data-testid="email"]', 'invalid-email');
    await page.click('[data-testid="save-button"]');

    // Then: Validation error
    await expect(page.locator('[data-testid="email-error"]')).toBeVisible();
    await expect(page.locator('[data-testid="email-error"]')).toContainText('Invalid email');
  });
});
```

### Step 6: 테스트 실행

생성된 테스트를 실행하고 결과를 확인합니다:

```bash
# 모든 테스트 실행
npx playwright test

# 특정 테스트만 실행
npx playwright test tests/user-profile-update.spec.ts

# UI 모드로 실행 (디버깅)
npx playwright test --ui

# 헤드리스 모드 해제 (브라우저 보이기)
npx playwright test --headed
```

**결과 분석:**
- ✅ 통과한 테스트 수
- ❌ 실패한 테스트 수
- ⚠️ 건너뛴 테스트 수
- 📊 실행 시간

실패한 테스트의 경우:
- 스크린샷 확인 (`test-results/` 디렉토리)
- 트레이스 파일 분석 (`npx playwright show-trace [trace-file]`)

### Step 7: 자동 수정 (Healer Agent)

테스트 실패 시 Healer Agent를 사용하여 자동으로 수정합니다:

```bash
# Healer Agent 실행
node .github/agents/healer.mjs
```

Healer는:
- 실패한 테스트를 분석
- 현재 UI에서 동등한 요소를 찾음
- 셀렉터나 로직을 자동으로 수정
- 수정 후 재실행

**수동 수정이 필요한 경우:**
- UI가 실제로 변경되어 테스트 로직 자체를 수정해야 할 때
- 테스트 데이터 문제
- 환경 설정 문제 (URL, 인증 등)

### Step 8: 결과 보고

사용자에게 테스트 결과를 종합하여 보고합니다:

```markdown
# E2E 테스트 결과 리포트

## 요약
- 총 테스트: X개
- 성공: Y개 ✅
- 실패: Z개 ❌
- 실행 시간: T초

## 생성된 파일
- 테스트 계획: `specs/[feature-name].spec.md`
- 테스트 코드: `tests/[feature-name].spec.ts`
- 결과 리포트: `test-results/`

## 실패한 테스트 (있는 경우)
### Test Case: [Name]
- 에러: [error message]
- 스크린샷: [path]
- 제안된 수정: [description]

## 다음 단계
- [ ] 실패한 테스트 수정 (있는 경우)
- [ ] 추가 테스트 케이스 검토
- [ ] CI/CD 파이프라인에 테스트 통합
- [ ] Pull Request 생성
```

## Best Practices

### 1. Seed Test 활용
첫 실행 시 간단한 seed 테스트를 만들어 Planner Agent가 참고하도록 합니다:

```typescript
// tests/seed.spec.ts
import { test } from '@playwright/test';

test('basic navigation', async ({ page }) => {
  await page.goto('http://localhost:3000');
  // 기본적인 네비게이션 테스트
});
```

### 2. 점진적 접근
- 먼저 핵심 사용자 플로우 테스트
- 그 다음 엣지 케이스
- 마지막으로 회귀 테스트

### 3. 명확한 셀렉터 사용
- `data-testid` 속성 우선 사용
- 불안정한 클래스명/ID 대신 안정적인 속성 사용

### 4. 대기 전략
- `waitForSelector`, `waitForLoadState` 적극 활용
- 하드코딩된 `setTimeout` 대신 조건부 대기

### 5. 테스트 격리
- 각 테스트는 독립적으로 실행 가능해야 함
- 테스트 간 상태 공유 지양

## Error Handling

### Playwright가 설치되지 않은 경우
```bash
npm init playwright@latest
npx playwright init-agents --loop=vscode
```

### 애플리케이션이 실행되지 않는 경우
테스트 전에 dev 서버가 실행 중인지 확인:
```bash
npm run dev
# 또는
npm start
```

### 테스트가 타임아웃되는 경우
`playwright.config.ts`에서 타임아웃 설정 증가:
```typescript
export default defineConfig({
  timeout: 60000, // 60초
});
```

## Examples

### Example 1: 로그인 기능 변경 시
```
변경 파일: LoginForm.tsx, auth.ts
테스트 플로우:
1. 올바른 자격증명으로 로그인 성공
2. 잘못된 자격증명으로 로그인 실패 검증
3. "비밀번호 찾기" 플로우
4. "회원가입" 버튼 동작
```

### Example 2: 새로운 대시보드 페이지 추가
```
변경 파일: Dashboard.tsx, DashboardAPI.ts, routes.tsx
테스트 플로우:
1. 대시보드 접근 및 로딩
2. 데이터 위젯 표시 확인
3. 필터링/정렬 기능
4. 반응형 레이아웃 (모바일/데스크톱)
```

### Example 3: 결제 플로우 수정
```
변경 파일: Checkout.tsx, PaymentService.ts
테스트 플로우:
1. 장바구니 → 결제 → 완료 전체 플로우
2. 다양한 결제 수단 (카드, 페이팔 등)
3. 결제 실패 시나리오
4. 쿠폰/할인 적용
```

## Integration with CI/CD

생성된 테스트를 GitHub Actions나 다른 CI/CD에 통합:

```yaml
# .github/workflows/e2e-tests.yml
name: E2E Tests
on: [pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run build
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: test-results/
```

## Output Format

스킬 실행 완료 시 다음 정보를 제공합니다:

1. **변경사항 요약**: 분석된 파일과 변경 범위
2. **테스트 계획**: 생성된 마크다운 계획 경로
3. **테스트 코드**: 생성된 `.spec.ts` 파일 경로
4. **실행 결과**: 통과/실패 개수, 실행 시간
5. **다음 단계**: PR 생성 전 확인 사항

이를 통해 사용자는 안심하고 Pull Request를 생성할 수 있습니다.
