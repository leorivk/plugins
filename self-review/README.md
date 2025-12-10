# Self-Review Plugin

PR 제출 전 철저한 자동화 리뷰와 반복적 개선을 통해 코드를 준비하는 Claude Code 플러그인입니다.

## 개요

이 플러그인은 리뷰어의 리뷰 공수를 최소화하기 위해 PR을 올리기 전에 자체적으로 포괄적인 코드 리뷰와 수정을 반복합니다.

## 아키텍처

이 플러그인은 **subagent 방식**을 사용하여 구현되었습니다:

```
self-review/
├── .claude-plugin/
│   └── plugin.json          # 플러그인 메타데이터
├── agents/
│   └── orchestrator.md      # 핵심 워크플로우 관리 agent
├── commands/
│   └── self-review.md       # /self-review 명령어 (orchestrator 호출)
└── README.md
```

**Slash Command → Subagent 구조**:
- `/self-review` 명령어를 실행하면 `orchestrator` subagent가 시작됩니다
- Orchestrator는 독립적인 컨텍스트에서 전체 9단계 워크플로우를 관리합니다
- 다른 agents (code-explorer, code-architect, pr-review-toolkit)를 병렬로 호출하여 효율적으로 작업합니다
- Git 작업, 사용자 상호작용, 반복 로직을 모두 orchestrator가 제어합니다

### 주요 기능

- ✅ **자동 브랜치 생성**: 기능에 맞는 브랜치 자동 생성
- ✅ **요구사항 명확화**: 구현 전 모호한 부분을 질문으로 해소
- ✅ **코드베이스 탐색**: code-explorer 에이전트로 기존 코드 패턴 분석
- ✅ **아키텍처 설계**: code-architect 에이전트로 여러 설계 접근법 제시
- ✅ **자동 커밋**: 구현 및 수정 후 자동으로 의미있는 커밋 생성
- ✅ **포괄적 리뷰**: pr-review-toolkit의 6개 전문 에이전트 모두 활용
  - code-reviewer (일반 코드 품질)
  - comment-analyzer (주석 정확성)
  - pr-test-analyzer (테스트 커버리지)
  - silent-failure-hunter (에러 처리)
  - type-design-analyzer (타입 설계)
  - code-simplifier (코드 단순화)
- ✅ **반복적 개선**: 이슈가 없을 때까지 자동 수정 및 재검토 반복
- ✅ **PR 준비 완료**: 리뷰어가 볼 준비가 된 고품질 코드

## 워크플로우 (9단계)

```
1. 요구사항 파악
   └─ 기능 명확화, 문제 정의

2. 브랜치 생성
   └─ feature/xxx 형식으로 자동 생성

3. 코드베이스 탐색
   └─ 2-3개 code-explorer 에이전트로 기존 코드 분석

4. 명확화 질문
   └─ 엣지 케이스, 에러 처리 등 불명확한 부분 질문

5. 아키텍처 설계
   └─ 2-3개 code-architect 에이전트로 여러 접근법 제시

6. 구현 + 커밋
   └─ 선택한 아키텍처로 구현 후 자동 커밋

7. 포괄적 리뷰
   └─ 6개 전문 리뷰 에이전트 병렬 실행
   └─ Critical/High/Medium/Low 우선순위로 이슈 분류

8. 이슈 해결 루프 (🔄 반복)
   └─ 이슈 수정 + 커밋
   └─ 재검토
   └─ 이슈 없을 때까지 반복 (최대 3회)

9. 요약 + PR 준비 완료
   └─ PR 제목/설명 제안
   └─ 다음 단계 안내
```

## 설치

### GitHub 저장소에서 설치 (권장)

```bash
# 1. Claude Code에서 마켓플레이스 추가
/plugin marketplace add leorivk/plugins

# 2. 플러그인 설치
/plugin install self-review@plugins
```

또는 대화형 메뉴 사용:

```bash
# /plugin 명령어 실행
/plugin

# "플러그인 찾아보기" 선택 후 self-review 설치
```

### 로컬 개발/테스트용 설치

```bash
# 저장소 클론
git clone https://github.com/leorivk/plugins.git

# 플러그인 디렉토리로 복사
cp -r plugins/self-review ~/.claude/plugins/
```

## 사용 방법

### 1. Slash Command 호출

```bash
# 기능 설명과 함께
/self-review 사용자 인증 기능 추가

# 또는 설명 없이 (대화형으로 진행)
/self-review
```

### 2. 워크플로우 진행

Claude가 자동으로:
1. 요구사항 명확화
2. 브랜치 생성 (사용자 확인 후)
3. 코드베이스 탐색
4. 추가 질문 (필요시)
5. 아키텍처 설계 및 선택
6. 구현 + 커밋
7. 6개 에이전트로 포괄적 리뷰
8. 이슈 수정 반복
9. PR 준비 완료!

### 3. PR 생성 (수동)

워크플로우 완료 후 사용자가 직접:
```bash
gh pr create
```

## 사용 예시

### 예시 1: 새로운 API 엔드포인트 추가

```
User: /self-review 사용자 프로필 조회 API 추가

Claude:
📋 Phase 1: 요구사항 파악
기능: 사용자 프로필 조회 API 추가
- REST API 엔드포인트를 추가하시나요?
- 인증이 필요한가요?
- 어떤 사용자 정보를 반환해야 하나요?

User: GET /api/users/:id, JWT 인증 필요, 기본 프로필 정보만

Claude:
✓ 이해했습니다.

🌿 Phase 2: 브랜치 생성
제안 브랜치: feature/add-user-profile-api
생성하시겠습니까?

User: 네

Claude:
✓ feature/add-user-profile-api 브랜치 생성 완료

🔍 Phase 3: 코드베이스 탐색
- code-explorer 1: 기존 API 엔드포인트 분석 중...
- code-explorer 2: JWT 인증 미들웨어 분석 중...
- code-explorer 3: User 모델 및 서비스 분석 중...

발견한 패턴:
- API 라우트: src/routes/api/
- 인증 미들웨어: src/middleware/auth.ts
- User 서비스: src/services/UserService.ts
...

[Phase 4-9 계속...]

📊 Phase 7: 포괄적 리뷰

Critical 이슈 (0개): 없음
High Priority 이슈 (2개):
- src/routes/api/users.ts:45 - 에러 처리 누락
- src/routes/api/users.ts:67 - 테스트 케이스 필요

어떻게 진행하시겠습니까?

User: 자동으로 수정해줘

Claude:
🔧 이슈 수정 중...
✓ 에러 처리 추가
✓ 테스트 케이스 작성
✓ 커밋 완료

🔄 재검토 중...
✓ 모든 이슈 해결!

✅ Phase 9: PR 준비 완료

생성된 커밋:
- feat: 사용자 프로필 조회 API 추가
- fix: 코드 리뷰 결과 반영

PR 제목 제안: [Feature]: 사용자 프로필 조회 API 추가
이제 `gh pr create`로 PR을 생성하세요!
```

## Phase별 상세 설명

### Phase 7: 포괄적 리뷰

6개의 전문화된 에이전트가 병렬로 실행됩니다:

1. **code-reviewer**:
   - CLAUDE.md 규칙 준수
   - 버그 탐지
   - 코드 품질 이슈
   - 스타일 가이드 준수

2. **comment-analyzer**:
   - 주석과 코드의 일치성
   - 문서 완전성
   - 오래된 주석 (comment rot) 탐지

3. **pr-test-analyzer**:
   - 테스트 커버리지 분석
   - 엣지 케이스 테스트 확인
   - 테스트 품질 평가

4. **silent-failure-hunter**:
   - catch 블록의 silent failure 탐지
   - 부적절한 에러 처리
   - 에러 로깅 누락

5. **type-design-analyzer**:
   - 타입 캡슐화 평가 (1-10점)
   - 불변성 표현 평가 (1-10점)
   - 타입 유용성 평가 (1-10점)

6. **code-simplifier**:
   - 불필요한 복잡성 식별
   - 코드 명확성 개선 제안
   - 중복 코드 제거

### Phase 8: 이슈 해결 루프

```
🔄 반복 1
├─ 발견: 5 critical, 8 high, 12 medium
├─ 수정 + 커밋
└─ 재검토 → 0 critical, 2 high, 5 medium

🔄 반복 2
├─ 발견: 0 critical, 2 high, 5 medium
├─ 수정 + 커밋
└─ 재검토 → 0 critical, 0 high, 2 medium

✅ 완료: Critical과 High Priority 이슈 모두 해결!
```

최대 3회 반복하며, 각 반복마다 진행 상황을 표시합니다.

## 필요 조건

- **Claude Code**: 설치 필요
- **Git**: 브랜치 생성 및 커밋용
- **pr-review-toolkit**: 자동으로 사용 (Claude Code 공식 플러그인)
- **feature-dev**: code-explorer, code-architect 에이전트 사용 (Claude Code 공식 플러그인)

## 플러그인 관리

### 플러그인 비활성화
```bash
/plugin disable self-review@plugins
```

### 플러그인 재활성화
```bash
/plugin enable self-review@plugins
```

### 플러그인 완전 제거
```bash
/plugin uninstall self-review@plugins
```

## 팁

### 효율적인 사용

1. **구체적인 요구사항**: 초기 요청을 명확히 하면 Phase 4의 질문이 줄어듭니다
2. **자동 수정 활용**: Critical과 High Priority는 자동 수정을 선택하면 빠릅니다
3. **반복 제한**: 3회 반복 후에도 이슈가 남아있다면 수동으로 검토하세요
4. **커밋 메시지 확인**: 각 커밋이 의미있는 단위인지 확인하세요

### 언제 사용하면 좋은가?

**추천**:
- 복잡한 새 기능 개발
- 여러 파일을 수정하는 작업
- 리뷰어의 시간이 제한적인 경우
- 높은 코드 품질이 요구되는 프로젝트

**비추천**:
- 한 줄짜리 버그 수정
- 타이포 수정
- 긴급 핫픽스
- 매우 간단한 작업

## 트러블슈팅

### 에이전트가 너무 오래 걸려요

**원인**: 큰 코드베이스는 탐색에 시간이 걸립니다

**해결**:
- 정상입니다. 에이전트는 병렬로 실행되어 최대한 빠릅니다
- 철저한 분석이 더 나은 결과를 가져옵니다

### Phase 4에서 질문이 너무 많아요

**해결**:
- 초기 요청을 더 구체적으로 작성하세요
- 제약사항을 미리 명시하세요
- "당신이 생각하기에 가장 좋은 방법으로"라고 답변하세요

### 반복이 끝나지 않아요

**해결**:
- 3회 반복 후 자동으로 중단됩니다
- Medium Priority는 무시하고 진행해도 괜찮습니다
- 일부 이슈는 수동으로 수정하는 것이 더 빠를 수 있습니다

## 라이선스

MIT

## 버전

1.0.0
