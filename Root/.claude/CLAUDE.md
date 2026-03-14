# Club Vegas Slots - 전체 시스템 아키텍처

**기본 언어**: 한국어

---

## 1. 철학

| 원칙 | 설명 |
|------|------|
| **Context First** | 코드 작성 전에 충분한 질문셋/맥락을 만든다 |
| **Structure First** | Architect가 구조를 먼저 보고, Executor가 구현한다 |
| **Change as Data** | 브랜치/커밋/리뷰/레트로를 모두 학습 가능한 데이터로 남긴다 |
| **Human in the Loop** | 스펙/설계/결과 승인 3지점에서 항상 사람이 개입한다 |

### 핵심 원칙

```
1. 실행 전에 충분히 계획하세요
2. 계획 전에 충분히 질문하세요
3. 권장 사양에 ⭐ 표시하세요
```

### 대안 시도 전 질문 원칙

**정석적인 방법이 실패했을 때, 대안을 시도하기 전에 반드시 사용자에게 질문하세요.**

```
⚠️ 중요: AI가 임의로 대안을 시도하지 말 것!
```

### 컴팩트 후 재확인 원칙

**컨텍스트 요약(Compact) 이후 작업이 불명확하면 반드시 사용자에게 재확인하세요.**

```
⚠️ 중요: 맥락을 잃었다고 느끼면 추측하지 말고 질문할 것!
```

| 상황 | 행동 |
|------|------|
| 현재 작업 목표가 불명확 | "현재 진행 중인 작업이 무엇인지 확인해주세요" |
| 이전 결정사항이 기억나지 않음 | "이전에 결정된 사항을 다시 알려주세요" |
| 파일/코드 맥락을 잃음 | "어떤 파일/기능을 수정 중이었는지 확인해주세요" |
| 다음 단계가 불분명 | "다음에 무엇을 해야 하는지 알려주세요" |

**워크플로우:**
```
컴팩트 발생 → 작업 명확성 확인 → 불명확하면 질문 → 확인 후 진행
```

### 🚀 병렬 처리 원칙 (컨텍스트 최적화)

**독립적인 작업은 반드시 병렬로 실행하세요.**

```
⚠️ 핵심: 의존성 없는 작업 = 병렬 / 의존성 있는 작업 = 순차
```

#### 병렬 실행 필수 상황

| 상황 | 병렬 실행 방법 |
|------|---------------|
| 여러 파일 읽기 | 단일 메시지에 여러 Read 호출 |
| 여러 검색 수행 | 단일 메시지에 여러 Grep/Glob 호출 |
| 독립적인 분석 | 단일 메시지에 여러 Task 호출 |
| MCP 도구 조합 | 의존성 없으면 한 번에 호출 |

#### 병렬 에이전트 호출

```markdown
✅ 올바른 예 (병렬):
단일 메시지에서:
- Task(architect): "클라이언트 구조 분석"
- Task(architect): "서버 구조 분석"

❌ 잘못된 예 (순차):
메시지 1: Task(architect): "클라이언트 구조 분석"
[결과 대기]
메시지 2: Task(architect): "서버 구조 분석"
```

#### 워크플로우 내 병렬화

| 워크플로우 | 병렬 가능 구간 |
|-----------|---------------|
| full-feature | architect(클라이언트) ∥ architect(서버) |
| refactor | critic(리스크) ∥ architect(구조) |
| exploration | 여러 영역 동시 탐색 |

#### 컨텍스트 절약 효과

```
순차 호출 (6턴): A → B → C → D → E → F
병렬 호출 (2턴): [A,B,C] → [D,E,F]
절약: ~67% 컨텍스트 감소
```

#### 🔑 병렬 호출 시 컨텍스트 전달 원칙 (필수)

**각 Subagent가 동일한 이해를 가질 수 있도록 컨텍스트를 아낌없이 전달하세요.**

```
⚠️ 핵심: Subagent는 독립된 세션 → 메인 에이전트의 맥락을 모름!
```

##### 반드시 포함해야 할 컨텍스트

| 항목 | 설명 | 예시 |
|------|------|------|
| **목표** | 최종 달성 목표 | "새 보너스 기능 구현" |
| **배경** | 왜 이 작업이 필요한지 | "기존 FreeSpin과 다른 Jackpot 연동 필요" |
| **제약사항** | 지켜야 할 규칙/제한 | "기존 BonusController 패턴 준수" |
| **관련 파일** | 참조해야 할 파일 경로 | "games/src/slot_game/343.ts 참조" |
| **기대 결과** | 산출물 형태 | "설계 문서 또는 구현 코드" |
| **현재 상태** | 지금까지 결정된 사항 | "Q1-1에서 옵션 A 선택됨" |

##### 올바른 예시

```markdown
✅ 좋은 병렬 호출:

Task(architect): """
[목표] 게임 343의 새 Jackpot 보너스 기능 설계
[배경] 기존 FreeSpin(34300)과 별도로 Jackpot(34301) 추가 요청
[제약사항]
- 기존 BonusController 패턴 준수
- Server-Authoritative RNG 유지
[관련 파일] games/src/slot_game/343.ts, 343.interface.ts
[기대 결과] 서버 측 보너스 로직 설계
[현재 상태] Planner 질문셋 완료, 옵션 A(독립 보너스) 선택됨

클라이언트 설계는 별도 에이전트가 병렬로 진행 중
"""

Task(architect): """
[목표] 게임 343의 새 Jackpot 보너스 기능 설계
[배경] 기존 FreeSpin(34300)과 별도로 Jackpot(34301) 추가 요청
[제약사항]
- 기존 FeatureController 패턴 준수
- FSM 상태 전이 명확히
[관련 파일] client/Assets/Contents/.../343/FSM/
[기대 결과] 클라이언트 측 FSM/피처 설계
[현재 상태] Planner 질문셋 완료, 옵션 A(독립 보너스) 선택됨

서버 설계는 별도 에이전트가 병렬로 진행 중
"""
```

```markdown
❌ 나쁜 병렬 호출:

Task(architect): "서버 보너스 설계해줘"
Task(architect): "클라이언트 보너스 설계해줘"

→ 각 에이전트가 다른 해석을 할 수 있음!
```

##### 병렬 호출 체크리스트

```markdown
병렬 Task 호출 전:
1. [ ] 목표와 배경이 명시되었는가?
2. [ ] 제약사항이 포함되었는가?
3. [ ] 관련 파일 경로가 있는가?
4. [ ] 현재까지의 결정 사항이 포함되었는가?
5. [ ] 다른 병렬 작업의 존재를 알려주었는가?
```

---

## 2. 필수 워크플로우

### 새 기능/씬 요청 시

```
사용자 요청 → 질문셋으로 스펙 확인 → task-id 생성 → 브랜치 생성 → 구현
```

**바로 구현하지 마세요!** 먼저:
1. 요구사항이 명확한지 확인
2. 불명확하면 질문셋 템플릿으로 질문
3. 스펙 확정 후 task-id 생성
4. `agent/<task-id>` 브랜치에서 작업

### 간단한 버그/수정 요청 시

- 명확한 한 줄 수정: 바로 진행 가능
- 여러 파일/복잡한 변경: 위 워크플로우 따름

---

## 3. 역할 (Subagents)

`.claude/agents/` 폴더에 정의된 역할들:

**범용 에이전트:**

| 역할 | Subagent | 용도 |
|------|----------|------|
| Planner | `planner` | 요구사항/질문셋 정리, 스펙 문서 |
| Architect | `architect` | 구조 설계, 기술 부채 관리 |
| Executor | `executor` | 코드/씬/프리팹 구현, 모든 구현 작업 |
| Reviewer | `reviewer` | 코드 리뷰, 품질 검증 |
| Critic | `critic` | 리스크 분석, 대안 제시 |
| Meta Agent | `meta-agent` | .claude/ 설정 개선/생성 |

**게임 제작 에이전트 (슬롯 파이프라인):**

| 역할 | Subagent | 용도 |
|------|----------|------|
| 기획자 | `slot-planner` | 기획서 → 개발용 기획 문서 변환 |
| 메쓰가이 | `math-guy` | 릴셋/확률/RTP 수학 모델 설계 |
| 서버개발자 | `slot-server-dev` | games 레포 서버 게임 로직 구현 |
| 클라개발자 | `slot-client-dev` | Unity 컨텐츠 로직, 피처 컨트롤러 구현 |

### Subagent 호출 방법

**Task tool**로 subagent를 호출합니다:

```
Task tool 사용:
- subagent_type: "planner" (또는 architect, executor 등)
- prompt: "작업 내용 설명"
```

### 언제 Subagent를 호출해야 하나

| 상황 | 호출할 Subagent |
|------|-----------------|
| 새 기능 요청, 요구사항 불명확 | `planner` |
| 구조 설계, 파일/폴더 결정 | `architect` |
| 코드/씬/프리팹 구현, 테스트 작성 | `executor` |
| 코드 리뷰, PR 검토 | `reviewer` |
| 리스크 분석, 주요 결정 전 | `critic` |
| 설정 문제/개선, 새 설정 추가 | `meta-agent` |

**새 기능 요청 시 권장 순서:**
```
planner → critic → architect → critic → executor → reviewer
```

### Subagent 출력 전달 규칙

Subagent가 반환한 출력을 사용자에게 전달할 때:

| Subagent | 전달 방식 |
|----------|-----------|
| planner | **원본 그대로** 전달 (요약 금지) |
| architect | 설계 핵심 + 세부사항 |
| critic | 리스크 전체 |
| reviewer | 리뷰 결과 전체 |

**Planner 출력 특별 규칙:**

> 🚨 **필수**: Planner 출력은 **전체 내용을 그대로** 사용자에게 전달해야 합니다.
> 메인 에이전트가 요약하거나 선택적으로 전달하면 **Human in the Loop 원칙 위반**입니다.

| 반드시 해야 할 것 | 절대 하지 말 것 |
|-------------------|-----------------|
| 모든 질문 (Q1-1, Q1-2...) 전달 | 일부 질문만 선택 |
| 모든 옵션 (A, B, C...) 전달 | 권장 옵션만 추출 |
| 각 옵션의 장단점 표 전달 | 장단점 생략 |
| 권장 이유 전달 | 이유 생략 |

**전달 방법:**
1. Planner Task 결과를 받으면
2. "Planner 질문셋 (원본)" 섹션으로 구분하여
3. **결과 전체를 그대로 복사**하여 전달
4. 요약본이 필요하면 원본 **아래에** 별도로 추가

**옵션 제시 규칙:**

> 🚨 **필수**: 사용자에게 옵션을 제시할 때 **개별 옵션 설명을 포함**해야 합니다.
> 옵션 조합(A+C)만 표시하고 개별 옵션(A, B, C) 설명을 생략하면 **정보 손실**입니다.

| 반드시 해야 할 것 | 절대 하지 말 것 |
|-------------------|-----------------|
| 개별 옵션(A, B, C) 각각 설명 | 조합(A+C)만 표시 |
| 각 옵션의 장단점 포함 | 권장 조합만 추출 |
| 옵션 선택 이유 제공 | 이유 생략 |

**예시:**
```
❌ 잘못된 예:
| 옵션 | 설명 |
| A+C | executor-code + 스킬 강화 |

✅ 올바른 예:
개별 옵션:
- A: executor-code.md에 Unity C# 절차 추가
- B: CLAUDE.md에 전역 규칙 추가
- C: unity-compile 스킬 강화

권장 조합: A+C (executor-code + 스킬 강화)
```

---

## 4. 아키텍처 개요

```
클라이언트 (Unity)                    서버 (Node.js)
┌─────────────────┐                  ┌─────────────────┐
│ FSM/BT ↔ Blackboard │──HTTP REST──→│ LOGIC Server    │
│ SlotMachine     │                  │ ↓               │
│ MetaSystem      │                  │ Games Engine    │
└─────────────────┘                  │ ↓               │
                                     │ MySQL/Redis/Dynamo│
                                     └─────────────────┘
```

**핵심 특징**:
- FSM/BT + Blackboard 데이터 주도형 게임 로직
- Server-Authoritative RNG
- 384개 게임 (슬롯 356, 비디오포커 20, 케노 10, 테이블 4)

---

## 5. 프로젝트 구조

```
club-vegas-slots/
├── client/                    # Unity 클라이언트
│   └── Assets/
│       ├── SlotMaker/         # 635 Action, 74 Condition
│       └── Contents/          # 게임별 FSM/BT
├── server/                    # LOGIC 서버 (Node.js)
│   └── src/LOGIC/             # API, 인증, 트랜잭션
├── games/                     # Games 엔진
│   └── src/slot_game/         # 356개 게임 로직
├── .claude/
│   ├── agents/                # Subagent 정의
│   ├── skills/                # 자동 트리거 스킬
│   ├── commands/              # 슬래시 명령
│   └── workflows/             # 워크플로우 프리셋
└── docs/tasks/                # 태스크 문서
```

### Git 저장소 위치 (중요!)

> ⚠️ **주의**: `club-vegas-slots/`는 Git 저장소가 아닙니다!
> 각 서브폴더가 독립된 Git 저장소입니다.

| 폴더 | Git 경로 | 용도 |
|------|----------|------|
| `client/` | `club-vegas-slots/client/.git` | Unity 클라이언트 |
| `server/` | `club-vegas-slots/server/.git` | LOGIC 서버 |
| `games/` | `club-vegas-slots/games/.git` | Games 엔진 |
| `client_sub/` | `club-vegas-slots/client_sub/.git` | 클라이언트 서브모듈 |

**Git 작업 시 주의사항:**
```bash
# ❌ 잘못된 방법 - club-vegas-slots 루트에서 git 명령 실행
cd club-vegas-slots && git status  # fatal: not a git repository

# ✅ 올바른 방법 - 각 서브폴더에서 git 명령 실행
cd club-vegas-slots/games && git status
cd club-vegas-slots/server && git status
cd club-vegas-slots/client && git status
```

**커밋 메시지 규칙:**
- **반드시 영어로 작성** (한국어 커밋 메시지 금지)
- 게임별 프리픽스: **대문자 게임 타이틀만** 사용 (숫자 ID 금지)
  - ✅ `[BBD]` [BGA]` 등 대문자 게임 타이틀 사용 (VALUE에 있음)
  - ❌ `[343]`, `[123]` 등 숫자 ID 사용 금지
- 예시: `[BBD] Fix sticky symbol logic in Super Free Game`

---

## 6. Skills (자동 트리거)

`.claude/skills/` 폴더에 정의된 스킬들:

### 공통 스킬

| # | Skill | 트리거 조건 |
|---|-------|-------------|
| 1 | `unity-compile` | .cs 파일 수정 시 (컴파일 절차) |
| 2 | `unity-csharp` | .cs 파일 작성 시 (코딩 패턴) |
| 3 | `question-set` | 새 기능/요구사항 정리 시 |
| 4 | `task-create` | 새 작업/브랜치 생성 시 |
| 5 | `git-guard` | 커밋/푸시/브랜치 작업 시 |
| 6 | `prefab-check` | .prefab 파일 수정 시 |
| 7 | `code-index` | 심볼 검색, 구조 분석, 코드 탐색 시 |
| 8 | `cost-analysis` | 2개 이상 대안 비교, 설계 완료 후 |
| 9 | `solution-research` | full-feature 워크플로우, 사용자 요청 시 |
| 10 | `mcp-server-check` | MCP 도구 호출 실패 시 |

### 빌드/개발

| Skill | 용도 | 키워드 |
|-------|------|--------|
| `/new-game` | 새 게임 개발 가이드 | 새 게임, 게임 추가 |
| `/new-api` | API 추가 가이드 | API 추가, 엔드포인트 |

### NodeCanvas (FSM/BT)

| Skill | 용도 | 키워드 |
|-------|------|--------|
| `/fsm-edit` | FSM 편집/생성 | FSM, 상태 머신 |
| `/bt-edit` | BT 편집/생성 | BT, Behavior Tree |
| `/new-action` | Custom Action 작성 | 액션 만들어, Condition |

### 게임 로직

| Skill | 용도 | 키워드 |
|-------|------|--------|
| `/bonus-flow` | 보너스 생명주기 | 보너스, 프리스핀 |
| `/spin-flow` | 스핀 전체 흐름 | 스핀 흐름, 게임 플로우 |
| `/feature-controller` | Feature 스크립트 | 피처, Controller |
| `/symbol-behaviour` | SymbolBehaviour | 심볼 동작, Scatter |
| `/contents-event` | Feature 간 이벤트 통신 | 이벤트, SendContentEvent |
| `/run-simulation` | 시뮬레이션 실행/검증 | 시뮬레이션, RTP 검증 |

### 데이터/매핑

| Skill | 용도 | 키워드 |
|-------|------|--------|
| `/blackboard-path` | Blackboard 경로 | 데이터 경로, 값 위치 |
| `/schema-sync` | Schema.json 동기화 | 스키마, CustomData |
| `/game-mapping` | 게임 ID 매핑 | 게임 ID, 보너스 ID |

---

## 7. Commands (슬래시 명령)

`.claude/commands/` 폴더에 정의:

| # | Command | 용도 |
|---|---------|------|
| 1 | `/sync` | 현재 변경 커밋 + 푸시 |
| 2 | `/finish` | 브랜치 완료 → main 머지 또는 PR |
| 3 | `/status` | 프로젝트 상태 요약 |
| 4 | `/new-task` | 새 task-id + 브랜치 + 문서 생성 |
| 5 | `/context` | 현재 작업 컨텍스트 요약 |
| 6 | `/workflow <name>` | 워크플로우 프리셋 실행 |
| 7 | `/pull-main` | main 브랜치 최신 변경사항 머지 |
| 8 | `/reindex [full]` | 코드 인덱스 서버 시작 + 재구축 |
| 9 | `/start-rmbg` | 배경 제거 서버 시작 |

### 워크플로우 프리셋 (`/workflow`)

`.claude/workflows/` 폴더에 정의:

| # | 프리셋 | 흐름 | 용도 |
|---|--------|------|------|
| 1 | `full-feature` | planner→critic→architect→critic→executor→reviewer | 새 기능 전체 개발 |
| 2 | `quick-fix` | executor→reviewer | 간단한 버그 수정 |
| 3 | `refactor` | critic→architect→executor→reviewer | 리팩토링 |
| 4 | `exploration` | architect만 | 코드베이스 분석 |
| 5 | `design-only` | planner→critic→architect→critic | 설계만 (구현 나중) |

---

## 8. 게임 레퍼런스 (필수)

### 게임 작업 시 레퍼런스 파일 필수 참조

> 🚨 **필수**: 특정 게임 관련 작업 시 `.claude/games/{game_code}.md` 레퍼런스를 **반드시 먼저 확인**하세요.

| 상황 | 행동 |
|------|------|
| 레퍼런스 파일 **있음** | 작업 전 **무조건 읽고** 참조 |
| 레퍼런스 파일 **없음** | 사용자에게 "게임 레퍼런스가 없습니다. 먼저 분석해서 만들까요?" 질문 |

**레퍼런스 파일 위치**: `.claude/games/{game_code}.md`

**파일명 규칙**: gameInfo.json의 `name` 필드 사용 (예: `bbd.md`, `scx.md`)

**현재 보유 레퍼런스:**
| 게임 | 파일 | Game ID |
|------|------|---------|
| Barbarian Destroyer | `.claude/games/bbd.md` | 234 |

**레퍼런스 포함 내용:**
- Game ID, 보너스 ID 맵, 심볼 enum
- 파일 위치 (서버/클라이언트/FSM)
- 핵심 클래스 계층도, ContentEvent 맵
- Blackboard 경로, FSM 구조
- 서버 주요 함수, 상수
- 패턴/Gotchas (수정 시 주의사항)

**워크플로우:**
```
게임 관련 작업 요청 → .claude/games/{code}.md 확인 → 있으면 참조 후 작업 / 없으면 사용자에게 생성 제안
```

---

## 9. 주요 규칙

### 보너스 ID
```
기본: gameId × 100 + 0~9  (예: 343 → 34300, 34301...)
티켓: gameId × 100 + 50   (예: 343 → 34350)
```

### API 엔드포인트
```
POST /v3/slot/spin      # 스핀 요청
POST /v3/slot/claim     # 보너스 클레임
POST /v3/slot/end-turn  # 턴 종료
```

### Blackboard 주요 경로
```
./spin/response/result/reelOutputList  # 릴 결과
./spin/response/bonusList              # 보너스 목록
./bonus/response/                      # 보너스 데이터
/me/credit                             # 유저 잔액
```

### Action/Task 생성 금지

> 🚨 **필수**: NodeCanvas Custom Action/Task를 임의로 생성하지 마세요!

| 반드시 해야 할 것 | 절대 하지 말 것 |
|-------------------|-----------------|
| 기존 635개 Action 먼저 검색 | 새 Action 바로 생성 |
| 기존 74개 Condition 먼저 검색 | 새 Condition 바로 생성 |
| 생성 필요 시 **사용자에게 확인** | 임의로 Action/Task 생성 |

**워크플로우:**
```
Action 필요 → 기존 Action 검색 → 없으면 사용자에게 질문 → 승인 후 생성
```

---

## 10. 빠른 시작

```bash
# 시뮬레이션 실행 (games 폴더에서)
cd games && ts-node src/slot_simulation/{game_code}.ts
```

---

## 11. Extended Thinking

복잡한 결정 시 활용:
- 새로운 시스템 설계
- 기존 구조와의 통합
- 트레이드오프 분석

---

## 12. 대형 프로젝트 운용 규칙

### 팀 에이전트 자동 사용

> **필수**: 다음 조건에 해당하면 **팀 에이전트(Team/Swarm)**로 운용하세요.

| 조건 | 설명 |
|------|------|
| 여러 세션에 걸치는 작업 | 한 컨텍스트에서 완료 불가능한 규모 |
| 3개 이상 영역 동시 진행 | 서버+클라이언트+게임 등 병렬 작업 |
| 마이그레이션/리팩토링 | 대규모 코드 변환, 구조 변경 |
| 사용자가 명시적 요청 | "팀으로", "병렬로", "swarm" 등 |

**팀 운용 워크플로우:**
```
1. TeamCreate → 팀 생성
2. Task → 팀원 스폰 (researcher, architect, executor 등)
3. TaskCreate → 작업 분배
4. 문서화 → docs/tasks/ 에 진행 상황 기록
5. 세션 종료 시 → 진행 상황 문서 업데이트
```

### 세션 간 연속성

대형 프로젝트는 **반드시 문서화**하여 세션 간 연속성을 보장:

| 문서 | 위치 | 내용 |
|------|------|------|
| 프로젝트 계획 | `docs/tasks/{project}/plan.md` | 전체 계획, 단계, 의존성 |
| 진행 상황 | `docs/tasks/{project}/progress.md` | 완료/진행/대기 작업 |
| 결정 로그 | `docs/tasks/{project}/decisions.md` | 주요 결정과 근거 |
| 패턴 가이드 | `docs/tasks/{project}/patterns.md` | 발견된 패턴, 변환 규칙 |

---

## CLAUDE.md 자동 업데이트 규칙

사용자 피드백, 새 패턴 발견, Best Practice 학습 시 관련 CLAUDE.md 파일을 자동으로 업데이트합니다.

| 내용 | 파일 |
|------|------|
| 클라이언트 FSM/BT/피처 패턴 | `client/Assets/Contents/.claude/CLAUDE.md` |
| SlotMaker 프레임워크 패턴 | `client/Assets/SlotMaker/.claude/CLAUDE.md` |
| 서버 API/보너스 패턴 | `server/.claude/CLAUDE.md` |
| 게임 엔진 로직 패턴 | `games/.claude/CLAUDE.md` |

자세한 내용은 각 Skill 또는 하위 CLAUDE.md 파일 참조.
