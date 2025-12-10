# Claude Code Plugins

Claude Code 플러그인 마켓플레이스입니다.

## 설치 방법

### 1. 마켓플레이스 추가

```bash
/plugin marketplace add leorivk/plugins
```

또는 로컬 경로로:

```bash
/plugin marketplace add /path/to/plugins
```

### 2. 플러그인 설치

**대화형 메뉴 사용:**
```bash
/plugin
```
"플러그인 찾아보기" 선택 후 원하는 플러그인 설치

**직접 설치:**
```bash
/plugin install playwright-e2e-tester@plugins
```

## 사용 가능한 플러그인

### 🎭 Playwright E2E Tester

PR 전 브랜치 변경사항을 분석하고 Playwright Test Agents로 E2E 테스트를 자동 생성 및 실행하는 플러그인

**주요 기능:**
- Git diff 기반 변경사항 자동 분석
- 대화형 테스트 범위 설정
- Playwright Test Agents 통합 (Planner, Generator, Healer)
- 마크다운 테스트 계획 자동 생성
- 실행 가능한 Playwright 테스트 코드 생성
- 테스트 자동 실행 및 결과 리포팅
- 실패한 테스트 자동 수정

**설치:**
```bash
/plugin install playwright-e2e-tester@plugins
```

**사용:**
```bash
@e2e-test
```

자세한 내용은 [Playwright E2E Tester README](./playwright-e2e-tester/README.md)를 참조하세요.

### 🔍 Self-Review

PR 제출 전 철저한 자동화 리뷰와 반복적 개선을 통해 코드를 준비하는 플러그인

**주요 기능:**
- 자동 브랜치 생성 및 커밋
- 코드베이스 탐색 및 아키텍처 설계 (feature-dev 활용)
- 6개 전문 리뷰 에이전트 병렬 실행 (pr-review-toolkit 활용)
  - code-reviewer, comment-analyzer, pr-test-analyzer
  - silent-failure-hunter, type-design-analyzer, code-simplifier
- 이슈가 없을 때까지 자동 수정 및 재검토 반복
- PR 준비 완료 상태로 만들어 리뷰어 공수 최소화

**설치:**
```bash
/plugin install self-review@plugins
```

**사용:**
```bash
/self-review 기능 설명
```

자세한 내용은 [Self-Review README](./self-review/README.md)를 참조하세요.

## 디렉토리 구조

```
plugins/
├── .claude-plugin/
│   └── marketplace.json          # 마켓플레이스 설정
├── playwright-e2e-tester/        # Playwright E2E 테스트 플러그인
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── skills/
│   │   └── e2e-test/
│   │       └── SKILL.md
│   └── README.md
├── self-review/                  # Self-Review 플러그인
│   ├── .claude-plugin/
│   │   └── plugin.json
│   ├── commands/
│   │   └── self-review.md
│   └── README.md
└── README.md                     # 이 파일
```

## 플러그인 개발 가이드

새로운 플러그인을 추가하려면:

1. **플러그인 디렉토리 생성**
   ```bash
   mkdir -p your-plugin-name/.claude-plugin
   mkdir -p your-plugin-name/skills/your-skill
   ```

2. **plugin.json 작성**
   ```json
   {
     "name": "your-plugin-name",
     "description": "플러그인 설명",
     "version": "1.0.0",
     "author": {
       "name": "작성자명"
     }
   }
   ```

3. **Skill 작성** (선택사항)
   - `skills/your-skill/SKILL.md` 파일 작성
   - YAML 프론트매터 + Markdown 지침 형식

4. **marketplace.json에 등록**
   ```json
   {
     "plugins": [
       {
         "name": "your-plugin-name",
         "source": "./your-plugin-name",
         "description": "플러그인 설명"
       }
     ]
   }
   ```

5. **테스트**
   ```bash
   /plugin marketplace add ./plugins
   /plugin install your-plugin-name@plugins
   ```

## 기여

새로운 플러그인 아이디어나 개선사항이 있으시면 이슈나 PR을 생성해주세요!

## 라이선스

MIT

## 관련 문서

- [Claude Code 문서](https://code.claude.com/docs)
- [Claude Skills 가이드](https://code.claude.com/docs/ko/skills)
- [Claude Plugins 가이드](https://code.claude.com/docs/ko/plugins)

## 문의

문제가 발생하거나 질문이 있으시면 이슈를 생성해주세요.
