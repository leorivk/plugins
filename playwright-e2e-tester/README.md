# Playwright E2E Tester Plugin

Claude Code 플러그인으로, Pull Request 전 브랜치 변경사항을 분석하고 Playwright Test Agents를 활용하여 E2E 테스트를 자동으로 생성 및 실행합니다.

## 개요

이 플러그인은 다음과 같은 워크플로우를 제공합니다:

1. **Git 변경사항 분석**: 현재 브랜치와 base 브랜치(main/master) 간의 diff를 분석하여 UI 관련 변경사항 파악
2. **대화형 요구사항 수집**: 사용자와 대화하며 테스트 범위, 사용자 플로우, 브라우저 환경 등을 결정
3. **Playwright 자동 설정**: 프로젝트에 Playwright가 없다면 자동으로 설치 및 Test Agents 초기화
4. **테스트 계획 생성**: 분석된 변경사항과 요구사항을 바탕으로 마크다운 테스트 계획 작성
5. **테스트 코드 생성**: Playwright Test Agents를 활용하여 실행 가능한 테스트 코드 자동 생성
6. **테스트 실행 및 리포팅**: 테스트를 실행하고 결과를 상세히 보고
7. **자동 수정**: 실패한 테스트를 Healer Agent로 자동 수정

## 설치

### GitHub 저장소에서 설치 (권장)

```bash
# 1. Claude Code에서 마켓플레이스 추가
/plugin marketplace add leorivk/plugins

# 2. 플러그인 설치
/plugin install playwright-e2e-tester@plugins
```

또는 대화형 메뉴 사용:

```bash
# /plugin 명령어 실행
/plugin

# "플러그인 찾아보기" 선택 후 playwright-e2e-tester 설치
```

### 로컬 개발/테스트용 설치

```bash
# 저장소 클론
git clone https://github.com/leorivk/plugins.git

# 플러그인 디렉토리로 복사
cp -r plugins/playwright-e2e-tester ~/.claude/plugins/
```

## 사용 방법

### 1. Skill 호출

Claude Code 대화에서 Skill을 호출합니다:

**방법 1: Skill 명령어**
```
@e2e-test
```

**방법 2: 자연어**
```
PR 전에 E2E 테스트를 생성하고 실행해줘
```

```
변경한 로그인 페이지에 대한 UI 테스트를 만들어줘
```

### 2. 대화형 프로세스

Skill이 실행되면 Claude가 다음 질문들을 합니다:

- "어떤 영역의 E2E 테스트를 생성할까요?"
- "특정 사용자 플로우를 테스트하고 싶으신가요?"
- "회귀 테스트도 포함할까요?"
- "테스트할 브라우저 환경은?"
- "애플리케이션 시작 URL은?"

### 3. 자동 실행

Claude가 자동으로:

1. Git diff 분석
2. Playwright 설치 확인 (없으면 설치)
3. Test Agents 초기화
4. 테스트 계획 작성 (`specs/*.spec.md`)
5. 테스트 코드 생성 (`tests/*.spec.ts`)
6. 테스트 실행
7. 결과 리포팅

## 예시 시나리오

### 시나리오 1: 새로운 로그인 기능 추가

```
User: PR 전에 로그인 기능 테스트를 생성해줘

Claude:
🔍 변경사항 분석 중...
발견된 변경 파일:
- src/components/LoginForm.tsx
- src/services/auth.ts

어떤 사용자 플로우를 테스트할까요?
1. 올바른 자격증명으로 로그인 성공
2. 잘못된 자격증명으로 로그인 실패
3. 비밀번호 찾기
4. 회원가입 링크

User: 1, 2번만 테스트해줘

Claude:
✅ Playwright 설치 확인 완료
📝 테스트 계획 생성 중... (specs/login.spec.md)
🎭 테스트 코드 생성 중... (tests/login.spec.ts)
🧪 테스트 실행 중...

결과:
✅ 2개 테스트 통과 (3.2초)

생성된 파일:
- specs/login.spec.md
- tests/login.spec.ts
```

### 시나리오 2: 대시보드 UI 변경

```
User: 대시보드 변경사항에 대한 E2E 테스트 만들어줘

Claude:
🔍 변경사항 분석 중...
발견된 변경 파일:
- src/pages/Dashboard.tsx
- src/components/DataWidget.tsx
- src/api/dashboardAPI.ts

회귀 테스트도 포함할까요? (기존 기능이 여전히 작동하는지 확인)

User: 네

Claude:
📱 모바일 반응형도 테스트할까요?

User: 네

Claude:
✅ 테스트 계획 생성 완료
🎭 테스트 코드 생성 중...
- Desktop 테스트
- Mobile 테스트
- 회귀 테스트

🧪 테스트 실행 중...

결과:
✅ 8개 테스트 통과
⚠️  1개 테스트 실패 (모바일 뷰 - 차트 렌더링)

🔧 Healer Agent로 자동 수정 시도 중...
✅ 수정 완료 및 재실행 성공!

최종 결과: 9개 테스트 모두 통과 ✅
```

## 생성되는 파일 구조

```
your-project/
├── .github/
│   └── agents/              # Playwright Test Agents
│       ├── planner.mjs
│       ├── generator.mjs
│       └── healer.mjs
├── specs/                   # 마크다운 테스트 계획
│   ├── login.spec.md
│   └── dashboard.spec.md
├── tests/                   # 생성된 Playwright 테스트
│   ├── login.spec.ts
│   └── dashboard.spec.ts
├── test-results/            # 테스트 결과 (스크린샷, 트레이스 등)
└── playwright.config.ts     # Playwright 설정
```

## 고급 기능

### CI/CD 통합

생성된 테스트는 GitHub Actions나 다른 CI/CD 파이프라인에 바로 통합할 수 있습니다:

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
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: test-results/
```

### 커스터마이징

생성된 테스트 코드는 완전히 편집 가능하며, 프로젝트 요구사항에 맞게 수정할 수 있습니다.

## 필요 조건

- **Node.js**: 16.x 이상
- **Git**: 변경사항 분석을 위해 필요
- **프론트엔드 프로젝트**: React, Vue, Svelte, Angular 등 모든 웹 프레임워크 지원
- **개발 서버**: 테스트 실행 시 애플리케이션이 실행 중이어야 함 (예: `localhost:3000`)

## 지원되는 프레임워크

- ✅ React (Next.js, Create React App, Vite 등)
- ✅ Vue (Nuxt, Vue CLI, Vite 등)
- ✅ Svelte (SvelteKit 등)
- ✅ Angular
- ✅ 기타 모든 웹 애플리케이션

## 트러블슈팅

### Playwright가 설치되지 않음
플러그인이 자동으로 설치를 시도하지만, 실패하는 경우 수동으로 설치:
```bash
npm init playwright@latest
```

### Test Agents가 초기화되지 않음
```bash
npx playwright init-agents --loop=vscode
```

### 테스트 타임아웃
`playwright.config.ts`에서 타임아웃 증가:
```typescript
export default defineConfig({
  timeout: 60000, // 60초
});
```

### 애플리케이션이 실행되지 않음
테스트 전에 dev 서버 실행:
```bash
npm run dev
# 또는
npm start
```

## 플러그인 관리

### 플러그인 비활성화
```bash
/plugin disable playwright-e2e-tester@plugins
```

### 플러그인 재활성화
```bash
/plugin enable playwright-e2e-tester@plugins
```

### 플러그인 완전 제거
```bash
/plugin uninstall playwright-e2e-tester@plugins
```

## 기여

이슈와 Pull Request는 언제나 환영합니다!

## 라이선스

MIT

## 관련 문서

- [Claude Code Skills](https://code.claude.com/docs/ko/skills)
- [Claude Code Plugins](https://code.claude.com/docs/ko/plugins)
- [Playwright Test Agents](https://playwright.dev/docs/test-agents)
- [Playwright 문서](https://playwright.dev/)

## 문의

문제가 발생하거나 질문이 있으시면 이슈를 생성해주세요.
