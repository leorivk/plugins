# Auto-Dev Plugin

자율적이고 반복적인 개발 워크플로우를 제공하는 Claude Code 플러그인입니다. 여러 이슈를 병렬로 처리하고, 각 이슈는 완료될 때까지 자동으로 개선됩니다.

## 개요

Auto-Dev는 다음을 자동화합니다:

- ✅ **Git Worktree 관리**: 각 작업을 독립적인 환경에서 격리
- ✅ **반복적 개선**: 이슈가 해결될 때까지 자동으로 리뷰-수정 반복
- ✅ **컨텍스트 연속성**: 반복 간 상태 유지로 점진적 개선
- ✅ **병렬 처리**: 여러 이슈를 동시에 처리 (향후 지원)

## 아키텍처

### 디렉토리 구조

```
auto-dev/
├── .claude-plugin/
│   └── plugin.json          # 플러그인 메타데이터
├── agents/
│   ├── orchestrator.md      # 전체 워크플로우 관리
│   ├── worktree-manager.md  # Git worktree 생성/관리
│   └── issue-worker.md      # 단일 이슈 처리 (반복 루프)
├── commands/
│   └── auto-dev.md          # /auto-dev 명령어
├── templates/
│   └── TASK_CONTEXT.md      # 반복 간 컨텍스트 템플릿
└── README.md
```

### 핵심 컴포넌트

#### 1. Orchestrator Agent

전체 워크플로우를 조율하는 마스터 에이전트입니다.

**역할**:

- 사용자 요구사항 파싱
- Worktree Manager 호출하여 격리 환경 생성
- Issue Worker 실행 및 모니터링
- 최종 결과 요약

**MVP 범위**: 단일 이슈 처리

#### 2. Worktree Manager Agent

Git worktree 생성 및 관리를 담당합니다.

**역할**:

- 브랜치명 자동 생성 (컨벤션 기반)
- Git worktree 생성/삭제
- 환경 초기화 (npm install 등)
- TASK_CONTEXT.md 파일 생성

**브랜치 네이밍 패턴**:

```
auto-dev/task-{number}/{description}-{hash}

예시:
- auto-dev/task-1/add-user-auth-a1b2c3d4
- auto-dev/task-2/fix-rate-limiting-b2c3d4e5
```

**Worktree 디렉토리 구조**:

```
프로젝트-root/
├── .git/
├── src/
└── .worktrees/           # 모든 worktree는 여기에
    ├── task-1/
    │   ├── TASK_CONTEXT.md
    │   └── (프로젝트 파일들)
    └── task-2/
        ├── TASK_CONTEXT.md
        └── (프로젝트 파일들)
```

#### 3. Issue Worker Agent

단일 이슈를 완료까지 책임지는 핵심 에이전트입니다.

**역할**:

- TASK_CONTEXT.md 읽기/쓰기
- 반복 루프 실행 (continuous-claude 패턴)
- 코드베이스 탐색 및 설계
- 구현 및 커밋
- 포괄적 리뷰
- 이슈 수정 및 재검토
- PR 준비

**반복 루프 구조**:

```
while (이슈가 남아있고 && 최대 반복 횟수 미만):
    1. TASK_CONTEXT.md 읽기
    2. 현재 상태 파악

    if 반복 == 1:
        - 요구사항 명확화
        - 코드베이스 탐색 (2-3 code-explorer agents 병렬)
        - 아키텍처 설계 (2-3 code-architect agents 병렬)
        - 구현
    else:
        - 이전 반복 이슈 수정

    3. 커밋 생성
    4. 포괄적 리뷰 (6개 pr-review-toolkit agents 병렬)
    5. 리뷰 결과 분석
    6. TASK_CONTEXT.md 업데이트

    if Critical과 High Priority 이슈 없음:
        break

7. PR 생성 제안
```

### TASK_CONTEXT.md 파일

반복 간 컨텍스트를 유지하는 핵심 파일입니다.

**구조**:

```markdown
# Task: [작업 설명]

## Metadata

- Task ID: task-1
- Branch: auto-dev/task-1/add-user-auth-a1b2c3d4
- Current Iteration: 2
- Max Iterations: 5
- Started: 2024-01-15T10:30:00Z

## Goal

[최종 목표 설명]

## Iteration History

### Iteration 1 (Completed)

**What was done**:

- 요구사항 분석 완료
- 코드베이스 탐색: 기존 인증 패턴 발견 (src/auth/)
- 아키텍처 설계: JWT 기반 OAuth 2.0 선택
- 구현: 기본 인증 스캐폴딩 추가

**Commits**:

- `a1b2c3d` feat: Add OAuth authentication scaffolding

**Review Results**:

- Critical: 3 issues
  - auth.ts:45 - 에러 처리 누락
  - auth.ts:67 - 보안: 토큰 검증 미흡
  - auth.ts:89 - SQL injection 취약점
- High: 5 issues
  - 테스트 커버리지 부족
  - 타입 정의 불완전
  - 문서화 누락
  - 에러 메시지 개선 필요
  - 로깅 추가 필요

**Next Actions**:

- [ ] CRITICAL: auth.ts:45 에러 처리 추가
- [ ] CRITICAL: auth.ts:67 토큰 검증 강화
- [ ] CRITICAL: auth.ts:89 SQL injection 방어
- [ ] HIGH: 테스트 케이스 작성
- [ ] HIGH: 타입 정의 완성

### Iteration 2 (In Progress)

**Current Focus**:

- Critical 이슈 3개 수정 중

**Notes**:

- auth.ts:45: try-catch 추가하고 적절한 에러 반환
- auth.ts:67: jwt.verify() 옵션 강화
- auth.ts:89: Prepared statements 사용

## Current Status

- **Phase**: Iteration 2 - Fixing Critical Issues
- **Blocked**: No
- **Ready for Review**: No

## Decision Log

- [Iteration 1] JWT vs Session: JWT 선택 (stateless API 요구사항)
- [Iteration 1] OAuth Provider: OAuth 2.0 표준 준수
- [Iteration 2] Error Handling: Custom error classes 도입

## Related Files

- src/auth/login.ts
- src/auth/register.ts
- src/auth/middleware.ts
- src/models/User.ts
- tests/auth.test.ts
```

**AI 에이전트 활용 가이드라인**:

- **명확한 섹션 구분**: 각 섹션은 특정 목적 (히스토리, 현재 상태, 다음 액션)
- **실행 가능한 체크리스트**: `[ ]` 형식으로 다음 할 일 명시
- **구체적인 컨텍스트**: 파일명, 라인 번호, 구체적 문제 설명
- **의사결정 기록**: 왜 이런 선택을 했는지 기록
- **메타데이터**: 반복 횟수, 시간 등 추적 정보

## 워크플로우

### Phase 1: 초기화

```
User: /auto-dev "사용자 인증 기능 추가"
    ↓
Orchestrator
    ↓
Worktree Manager
    ├─ 브랜치 생성: auto-dev/task-1/add-user-auth-a1b2c3d4
    ├─ Worktree 생성: .worktrees/task-1/
    ├─ 환경 설정: npm install (선택적)
    └─ TASK_CONTEXT.md 초기화
    ↓
Issue Worker 시작
```

### Phase 2: 반복 1 - 구현

```
Issue Worker - Iteration 1:
    ↓
1. 요구사항 명확화
   └─ 사용자와 대화하여 불명확한 부분 질문
    ↓
2. 코드베이스 탐색
   └─ 2-3개 code-explorer agents 병렬 실행
   └─ 기존 패턴, 아키텍처, 관련 코드 분석
    ↓
3. 아키텍처 설계
   └─ 2-3개 code-architect agents 병렬 실행
   └─ 여러 접근법 제시 및 사용자 선택
    ↓
4. 구현
   └─ 선택된 아키텍처로 코드 작성
    ↓
5. 커밋
   └─ git commit -m "feat: Add user authentication"
    ↓
6. 포괄적 리뷰
   └─ 6개 pr-review-toolkit agents 병렬 실행:
       - code-reviewer
       - comment-analyzer
       - pr-test-analyzer
       - silent-failure-hunter
       - type-design-analyzer
       - code-simplifier
    ↓
7. 결과 분석
   ├─ Critical: 3 issues
   ├─ High: 5 issues
   └─ Medium: 8 issues
    ↓
8. TASK_CONTEXT.md 업데이트
   └─ 발견된 이슈, 다음 할 일 기록
```

### Phase 3: 반복 2 - 수정

```
Issue Worker - Iteration 2:
    ↓
1. TASK_CONTEXT.md 읽기
   └─ 이전 반복 결과 파악
   └─ Critical 이슈 3개 확인
    ↓
2. Critical 이슈 수정
   └─ 각 이슈 하나씩 해결
    ↓
3. 커밋
   └─ git commit -m "fix: Address critical auth issues"
    ↓
4. 재리뷰
   └─ 6개 agents 다시 실행
    ↓
5. 결과 분석
   ├─ Critical: 0 issues ✅
   ├─ High: 2 issues
   └─ Medium: 5 issues
    ↓
6. TASK_CONTEXT.md 업데이트
```

### Phase 4: 반복 3 - 마무리

```
Issue Worker - Iteration 3:
    ↓
1. High Priority 이슈 수정
    ↓
2. 커밋
    ↓
3. 최종 리뷰
   ├─ Critical: 0 ✅
   ├─ High: 0 ✅
   └─ Medium: 3 (허용 가능)
    ↓
4. PR 준비 완료
   └─ PR 제목/설명 제안
```

## 사용 방법

### 기본 사용

```bash
# 텍스트 설명으로 시작
/auto-dev "사용자 인증 기능 추가"

# 또는 설명 없이 (대화형)
/auto-dev
```

### 예상 출력

```
🚀 Auto-Dev 시작

📦 Phase 1: 초기화
✓ 브랜치 생성: auto-dev/task-1/add-user-auth-a1b2c3d4
✓ Worktree 생성: .worktrees/task-1/
✓ 환경 설정 완료
✓ TASK_CONTEXT.md 초기화

🔄 Phase 2: Iteration 1
📋 요구사항 명확화 중...
   Q: OAuth 2.0을 사용하시나요, 아니면 자체 인증을 구현하시나요?
   Q: JWT 토큰을 사용하시나요?

[사용자 응답]

🔍 코드베이스 탐색 중 (3 explorers 병렬)...
   ✓ Explorer 1: 기존 인증 패턴 분석
   ✓ Explorer 2: 사용자 모델 구조 파악
   ✓ Explorer 3: API 라우팅 패턴 확인

🏗️ 아키텍처 설계 중 (3 architects 병렬)...
   ✓ Architect 1: Minimal Changes 접근
   ✓ Architect 2: Clean Architecture 접근
   ✓ Architect 3: Pragmatic Balance 접근

   추천: Pragmatic Balance (빠른 구현 + 유지보수성)
   진행하시겠습니까? [y/n]

[사용자 확인]

✍️ 구현 중...
   ✓ src/auth/login.ts 생성
   ✓ src/auth/register.ts 생성
   ✓ src/auth/middleware.ts 생성
   ✓ tests/auth.test.ts 생성

✓ 커밋 완료: feat: Add user authentication scaffolding (a1b2c3d)

📊 포괄적 리뷰 중 (6 agents 병렬)...
   ✓ code-reviewer: 완료
   ✓ comment-analyzer: 완료
   ✓ pr-test-analyzer: 완료
   ✓ silent-failure-hunter: 완료
   ✓ type-design-analyzer: 완료
   ✓ code-simplifier: 완료

📋 리뷰 결과:
   🔴 Critical (3):
      - src/auth/login.ts:45 - 에러 처리 누락
      - src/auth/login.ts:67 - 토큰 검증 미흡
      - src/auth/register.ts:89 - SQL injection 취약점

   🟡 High (5):
      - 테스트 커버리지 부족
      - 타입 정의 불완전
      - 문서화 누락
      - 에러 메시지 개선 필요
      - 로깅 추가 필요

💾 TASK_CONTEXT.md 업데이트 완료

🔄 Phase 3: Iteration 2
🔧 Critical 이슈 수정 중...
   ✓ src/auth/login.ts:45 - try-catch 추가
   ✓ src/auth/login.ts:67 - 토큰 검증 강화
   ✓ src/auth/register.ts:89 - Prepared statements 적용

✓ 커밋 완료: fix: Address critical auth issues (b2c3d4e)

📊 재리뷰 중...

📋 리뷰 결과:
   ✅ Critical (0)
   🟡 High (2)

계속 수정하시겠습니까? [y/n]

[계속...]

🔄 Phase 4: Iteration 3
🔧 High Priority 이슈 수정 중...

[...]

✅ 모든 Critical과 High Priority 이슈 해결!

📝 PR 준비 완료

생성된 커밋:
  - a1b2c3d feat: Add user authentication scaffolding
  - b2c3d4e fix: Address critical auth issues
  - c3d4e5f refactor: Improve auth test coverage

제안 PR 제목: feat: Add user authentication with OAuth 2.0

이제 `gh pr create`로 PR을 생성하거나,
worktree를 유지하고 추가 작업을 계속하세요.
```

## MVP 범위

### 포함된 기능

- ✅ **단일 이슈 처리**: 하나의 작업을 완료까지 처리
- ✅ **Worktree 관리**: 독립적인 환경 생성/관리
- ✅ **반복 루프**: 이슈가 해결될 때까지 자동 반복
- ✅ **컨텍스트 연속성**: TASK_CONTEXT.md로 상태 유지
- ✅ **포괄적 리뷰**: 6개 pr-review-toolkit agents 활용
- ✅ **자동 커밋**: 각 단계마다 의미있는 커밋 생성

### 향후 추가 예정

- ⚡ **병렬 이슈 처리**: 여러 이슈 동시 처리
- ⚡ **Task Splitter**: 한 이슈를 여러 병렬 작업으로 분할
- ⚡ **GitHub 통합**: 이슈 번호로 직접 작업
- ⚡ **자동 PR 생성**: gh pr create 자동 실행
- ⚡ **CI 모니터링**: PR 체크 대기 및 자동 머지

## 필요 조건

- **Claude Code**: 최신 버전
- **Git**: Worktree 지원 (Git 2.5+)
- **pr-review-toolkit**: Claude Code 공식 플러그인
- **feature-dev**: code-explorer, code-architect agents

## 설치

```bash
# 플러그인 설치
/plugin install auto-dev@plugins

# 또는 로컬 개발
cp -r auto-dev ~/.claude/plugins/
```

## 설정

### 최대 반복 횟수

기본값: 5회

수정하려면 `TASK_CONTEXT.md` 템플릿 또는 Issue Worker agent 설정 조정

### Worktree 디렉토리

기본값: `.worktrees/`

변경하려면 Worktree Manager agent 설정 조정

## 트러블슈팅

### Worktree 생성 실패

**원인**: 브랜치명 충돌 또는 권한 문제

**해결**:

- 수동으로 브랜치 삭제: `git branch -D auto-dev/task-1/...`
- Worktree 정리: `git worktree prune`

### 반복이 끝나지 않음

**원인**: 일부 이슈가 계속 발견됨

**해결**:

- 최대 반복 횟수 제한 (기본 5회)
- Medium Priority 이슈는 무시 가능
- 수동으로 남은 이슈 처리

### TASK_CONTEXT.md 손상

**원인**: 파일 수동 편집 또는 충돌

**해결**:

- 템플릿에서 새로 생성
- Git에서 이전 버전 복구

## 라이선스

MIT

## 버전

0.1.0 (MVP)
