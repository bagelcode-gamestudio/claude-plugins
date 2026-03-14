# Skill Reference Guide

스킬 빠른 참조 가이드입니다. 다음에 다시 찾아보지 않고 바로 사용할 수 있도록 핵심 정보만 정리했습니다.

---

## 스킬 카테고리별 요약

### 🎰 게임 로직 (슬롯 개발 핵심)

| 스킬 | 트리거 키워드 | 핵심 용도 |
|------|--------------|----------|
| `bonus-flow` | 보너스, 프리스핀, ClaimBonus | 보너스 생명주기 (Begin→Claim→Execute→End) |
| `spin-flow` | 스핀 흐름, 게임 플로우 | 클라이언트-서버 스핀 전체 흐름 |
| `feature-controller` | 피처, Controller, Module | FeatureController/Module 스크립트 작성 |
| `symbol-behaviour` | 심볼 동작, Scatter, Wild | SymbolBehaviour 라이프사이클 |
| `blackboard-path` | 데이터 경로, Blackboard | Blackboard 경로 체계 |
| `game-mapping` | 게임 ID, bonusId | 게임 ID, 보너스 ID 매핑 규칙 |

### 🔧 NodeCanvas (FSM/BT)

| 스킬 | 트리거 키워드 | 핵심 용도 |
|------|--------------|----------|
| `fsm-edit` | FSM, 상태 머신 | FSM 에셋 편집/생성 |
| `bt-edit` | BT, Behavior Tree | BT 에셋 편집 (WaitUntil 필수 규칙!) |
| `new-action` | 새 액션, 커스텀 액션 | SlotActionTask/SlotConditionTask 작성 |

### 🏗️ 빌드/개발

| 스킬 | 트리거 키워드 | 핵심 용도 |
|------|--------------|----------|
| `build-server` | 서버 빌드, npm run logic | LOGIC 서버 빌드/실행 |
| `build-games` | 게임 빌드, npm run dist-ts | Games 엔진 빌드 |
| `new-game` | 새 게임, 게임 추가 | 새 게임 10단계 체크리스트 |
| `new-api` | API 추가, 엔드포인트 | 서버 API 추가 절차 |
| `run-simulation` | 시뮬레이션, RTP 확인 | 게임 로직 검증 |

### 📝 공통/프로세스

| 스킬 | 트리거 키워드 | 핵심 용도 |
|------|--------------|----------|
| `question-set` | 새 기능, 요구사항 정리 | 구조화된 질문셋 템플릿 |
| `task-create` | 새 작업, 브랜치 생성 | task-id 생성 + 문서 + 브랜치 |
| `cost-analysis` | 대안 비교, 설계 완료 | 다차원 비용 분석 (c_eff 공식) |
| `solution-research` | 기존 솔루션 검색 | "바퀴 재발명" 방지 리서치 |

### 🛡️ 품질/검증

| 스킬 | 트리거 키워드 | 핵심 용도 |
|------|--------------|----------|
| `unity-csharp` | .cs 파일 작성 | Unity C# 코딩 패턴 (OnValidate, fake null 등) |
| `prefab-check` | .prefab 수정 | Override/Missing Reference 확인 |
| `git-guard` | 커밋, 푸시 | 브랜치 규칙, 커밋 메시지 검증 |
| `schema-sync` | 스키마, CustomData | 서버-클라이언트 Schema.json 동기화 |

### 🔍 검색/인덱스

| 스킬 | 트리거 키워드 | 핵심 용도 |
|------|--------------|----------|
| `code-index` | 클래스 찾기, 구조 분석 | 코드 심볼/참조/계층 검색 |
| `clip-index` | 클립 찾기, 애니메이션 | 애니메이션 클립 태그/카테고리 검색 |
| `mcp-server-check` | MCP 도구 실패 | MCP 서버 상태 확인/시작 안내 |

---

## 핵심 스킬 상세

### 1. bonus-flow (보너스 생명주기)

**보너스 흐름**:
```
CheckBonus → BeginSpinBonus → [ClaimBonus] → NestedFSM/BT 실행 → EndBonus
```

**보너스 ID 규칙**:
```
기본: gameId × 100 + 0~9  (예: 343 → 34300, 34301...)
티켓: gameId × 100 + 50   (예: 343 → 34350)
```

**Blackboard 경로**:
```
./bonus/bonusId        # 현재 보너스 ID
./bonus/spinCount      # 남은 스핀
./bonus/earnCredit     # 보너스 획득금
./bonus/response/      # 서버 응답 데이터
```

---

### 2. bt-edit (BT 편집)

**⚠️ 필수 규칙: WaitUntil은 반드시 자식 노드 필요!**

```json
// 올바른 WaitUntil 구조
{
  "nodes": [
    {"$type": "NodeCanvas.BehaviourTrees.WaitUntil", "$id": "5",
     "_condition": {"eventName": {"_value": "EndFeature"}}},
    {"$type": "NodeCanvas.BehaviourTrees.ActionNode", "$id": "8",
     "_action": {"status": "Success", "$type": "NodeCanvas.Tasks.Actions.RaiseStatus"}}
  ],
  "connections": [
    {"_sourceNode": {"$ref": "5"}, "_targetNode": {"$ref": "8"}}
  ]
}
```

**Feature BT 표준 패턴**:
```
BinarySelector(CheckBonus) → Sequencer → SendContentEvent → WaitUntil → RaiseStatus
```

---

### 3. blackboard-path (데이터 경로)

**주요 경로 빠른 참조**:

| 용도 | 경로 |
|------|------|
| 현재 베팅 | `./betCredit` |
| 스핀 획득금 | `./spin/earnCredit` |
| 릴 결과 | `./spin/response/result/reelOutputList` |
| 보너스 목록 | `./spin/response/bonusList` |
| 커스텀 데이터 | `./spin/response/customData` |
| 현재 덱 | `./spin/deck` |
| 보너스 ID | `./bonus/bonusId` |
| 남은 스핀 | `./bonus/spinCount` |
| 유저 잔액 | `/me/credit` |

**코드 접근**:
```csharp
var bb = ContentBlackboard.Get();
var value = bb.GetValue<long>("./spin/earnCredit");
bb.SetValue("./customData/myValue", data);
```

---

### 4. feature-controller (피처 스크립트)

**클래스 선택 가이드**:

| 클래스 | 용도 |
|--------|------|
| `FeatureController` | 단순, 동기적 피처 |
| `FeatureControllerAsync` | UniTask 기반, 복잡 연출 |
| `FeatureModule` | 여러 이벤트 조합 |
| `FeatureModuleAsync` | 코디네이터 역할 |

**FeatureControllerAsync 템플릿**:
```csharp
public class MyFeatureController : FeatureControllerAsync
{
    protected override string ON_FEATURE_BEGIN_EVENT => "MyFeature_Begin";
    protected override string ON_FEATURE_END_EVENT => "MyFeature_End";

    protected override async UniTask OnPlay()
    {
        var bb = ContentBlackboard.Get();
        // 연출 실행
        effect.SetActive(true);
        await WaitForSeconds(1.5f);
        effect.SetActive(false);
        // OnPlay 완료 시 자동으로 END_EVENT 발송
    }
}
```

**⚠️ 중요: Script에서 CheckBonus 하지 마세요! BT에서 처리!**

---

### 5. unity-csharp (Unity C# 패턴)

**필수 패턴 1: RequireComponent + OnValidate**
```csharp
[RequireComponent(typeof(Camera))]
public class UICamera : MonoBehaviour
{
    [SerializeField, HideInInspector] private Camera _camera;

    void OnValidate()
    {
        if (_camera == null) _camera = GetComponent<Camera>();
    }

    void Awake()
    {
        if (_camera == null) _camera = GetComponent<Camera>(); // fallback
    }
}
```

**필수 패턴 2: Unity fake null 주의**
```csharp
// ❌ 금지 - Unity destroyed 상태 감지 못함
public static Camera? Camera => _instance?._camera;

// ✅ 올바름 - != null로 명시적 체크
public static Camera? Camera => _instance != null ? _instance._camera : null;
```

---

### 6. question-set (질문셋 템플릿)

**필수 형식**:
```markdown
# 📋 [기능명] - 스펙 확정 질문셋

## 분석 축 (1~3개)
1. [축1명] (1-5)
2. [축2명] (1-5)

## 1. [카테고리명]

**Q1-1. [질문]**
- A) [S] [옵션 1] - Safe (안전한 선택)
- B) [B] [옵션 2] ⭐ (권장) - Balanced
- C) [I] [옵션 3] - Ideal (이상적이지만 고비용)

**축 기반 평가:**
| 옵션 | 성격 | [축1] | [축2] | 종합 |
|------|------|-------|-------|------|
| A    | Safe | 1     | 2     | 3    |

**권장 이유:** [축 기반 설명]
```

---

### 7. cost-analysis (비용 분석)

**c_eff 공식**:
```
c_eff = (effort × 1.0) + (risk × 1.5) + (uncertainty × 1.2)
        - (reversibility × 1.0) - available_resource
```

**비용 요소 (1-5 스케일)**:

| 요소 | 1 (낮음) | 5 (높음) |
|------|----------|----------|
| effort | 몇 줄 수정 | 대규모 리팩토링 |
| risk | 로컬 영향 | 시스템 전체 영향 |
| uncertainty | 검증된 패턴 | 완전히 새로운 시도 |
| reversibility | git revert 가능 | 롤백 불가 |

**c_eff 해석**:
| 범위 | 해석 |
|------|------|
| 0-4 | 저비용, 진행 권장 |
| 5-8 | 중간, 검토 후 진행 |
| 9-12 | 고비용, 대안 탐색 |
| 13+ | 매우 고비용, 재설계 |

---

### 8. task-create (태스크 생성)

**task-id 형식**:
```
{priority}-{date}-{initials}-{keyword}
예: H1-2025-12-03-ds-tentacle-ai
```

| 요소 | 설명 |
|------|------|
| priority | H(High), M(Mid), L(Low) + 숫자 |
| date | 환경 정보의 Today's date 사용 |
| initials | git config user.name에서 추출 |
| keyword | kebab-case 키워드 |

**워크플로우**:
1. question-set으로 요구사항 확정
2. task-id 생성
3. `docs/tasks/<task-id>.md` 생성
4. `agent/<task-id>` 브랜치 생성
5. 작업 시작

---

### 9. run-simulation (시뮬레이션)

**실행 명령어**:
```bash
cd games
npx ts-node src/slot_simulation/{game_code}.ts -- -s 1000000 -r 0 -t 4
```

**파라미터**:
| 파라미터 | 설명 | 기본값 |
|----------|------|--------|
| `-s` | 스핀 횟수 | 1,000,000 |
| `-r` | RTP 인덱스 (0=LOW, 1=MID, 2=HIGH) | 0 |
| `-t` | 스레드 수 | CPU 코어 수 |

**결과 확인**:
```
RTP :
total       base        free        bonusonbase bonusonfree
0.960123    0.421234    0.312456    0.112345    0.114088
```
- total: 전체 RTP (목표값과 ±0.5% 이내 확인)

---

### 10. build-server / build-games

**서버 빌드**:
```bash
cd server
npm run dist-ts && npm run logic
```

**Games 엔진 빌드**:
```bash
cd games
npm run dist-ts
```

**서버 포트**:
| 서버 | 포트 |
|------|------|
| LOGIC | 3000 |
| ADMIN | 3001 |
| RANK | 3002 |

---

## 스킬 연관 관계도

```
게임 개발 워크플로우:

[새 기능 요청]
    │
    ├─▶ question-set (요구사항 정리)
    │       │
    │       ▼
    ├─▶ task-create (task-id + 브랜치)
    │       │
    │       ▼
    ├─▶ solution-research (기존 솔루션 검색)
    │       │
    │       ▼
    └─▶ [구현 시작]
            │
            ├─▶ fsm-edit / bt-edit (FSM/BT 작업)
            │       │
            │       ├─▶ new-action (커스텀 Action 필요시)
            │       │
            │       └─▶ bonus-flow / spin-flow (보너스/스핀 참조)
            │
            ├─▶ feature-controller (피처 스크립트)
            │       │
            │       └─▶ symbol-behaviour (심볼 동작)
            │
            ├─▶ unity-csharp (C# 코딩 패턴)
            │       │
            │       └─▶ prefab-check (Prefab 수정 시)
            │
            └─▶ [서버 수정 시]
                    │
                    ├─▶ new-api (API 추가)
                    │
                    ├─▶ schema-sync (Schema 동기화)
                    │
                    ├─▶ build-server / build-games (빌드)
                    │
                    └─▶ run-simulation (검증)
```

---

## MCP 서버 포트 참조

| 서버 | 포트 | 시작 명령 |
|------|------|----------|
| unity-bridge | 17902 | Unity Editor에서 자동 |
| code-index | 17910 | `/reindex` |
| clip-index | 17901 | `cd tools/clip-index-server && npm run start:http` |
| remove-bg | 17920 | `/start-rmbg` |

---

## 파일 위치 참조

| 타입 | 경로 |
|------|------|
| **클라이언트 FSM/BT** | `client/Assets/Contents/Contents Group 1/{GameName}/FSM/` |
| **클라이언트 스크립트** | `client/Assets/Contents/Contents Group 1/{GameName}/Scripts/` |
| **서버 게임 로직** | `games/src/slot_game/{game_name}.ts` |
| **서버 인터페이스** | `games/src/slot_game/{game_name}.interface.ts` |
| **API 라우터** | `server/src/LOGIC/routes/` |
| **시뮬레이션** | `games/src/slot_simulation/{game_code}.ts` |
| **SlotMaker Actions** | `client/Assets/SlotMaker/NodeCanvas/Tasks/Actions/` |
| **SlotMaker Conditions** | `client/Assets/SlotMaker/NodeCanvas/Tasks/Conditions/` |

---

## 🔄 Workflows (워크플로우 프리셋)

`/workflow <name>`으로 실행하는 워크플로우 프리셋입니다.

### 워크플로우 요약

| 프리셋 | 흐름 | 사용 조건 |
|--------|------|-----------|
| `full-feature` | planner → critic → architect → critic → executor → reviewer | 새 기능 전체 개발 |
| `quick-fix` | executor → reviewer | 영향 파일 3개 이하, 설계 변경 없음 |
| `refactor` | critic → architect → executor → reviewer | 구조 변경, 성능 최적화, 기술 부채 해소 |
| `design-only` | planner → critic → architect → critic | 설계 문서만 작성 (구현 나중) |
| `exploration` | architect만 | 코드베이스 탐색/분석 |

### full-feature (새 기능 전체 개발)

**DAG 흐름**:
```
[M0: 요청] → [M1: 스펙 확정] ⇄ critic#1
                ↓
            [M2: 설계 완료] ⇄ critic#2
                ↓
            [M3: 구현 완료] ⇄ reviewer
                ↓
            [M4: 검증] → [M5: 머지]
```

**Milestone 산출물**:
| Milestone | 산출물 |
|-----------|--------|
| M1 | `docs/tasks/<task-id>.md` |
| M2 | `docs/design/<task-id>.md` |
| M4 | `docs/reviews/<task-id>.md` |

**분기 조건**: c_eff > 8이면 이전 단계로 회귀

---

### quick-fix (간단한 버그 수정)

**사용 조건 (모두 만족)**:
1. 영향 파일 3개 이하
2. 명확한 버그/수정 사항
3. 설계 변경 없음
4. 새로운 의존성 없음

**격상 조건** → `full-feature`로:
- 영향 범위가 예상보다 큼
- 근본 원인이 설계 문제
- 새로운 패턴/구조 필요

---

### refactor (리팩토링)

**Critic 선행**: 리스크를 먼저 분석

**체크리스트**:
- [ ] 정말 필요한가?
- [ ] 더 작은 범위로 가능한가?
- [ ] 롤백 계획은?

**산출물**: `docs/design/<task-id>.md`, `docs/retro/<task-id>.md`

---

### design-only (설계만)

**용도**: 구현 일정 미정, 팀 논의 전 문서화, 기술 스파이크

**산출물**:
- `docs/tasks/<task-id>.md` - 확정된 스펙
- `docs/design/<task-id>.md` - 상세 설계
- `docs/critic/<task-id>.md` - 리스크/대안 분석

**나중에 구현 시**: `executor → reviewer`로 직접 시작 (설계 있으므로)

---

### exploration (탐색)

**분석 유형**:
- **A. 전체 구조**: 폴더 구조, 주요 모듈, 의존성
- **B. 특정 모듈**: 책임 범위, 공개 API, 내부 구조
- **C. 기술 부채**: 중복 코드, 복잡한 의존성, 오래된 패턴

**다음 단계 연결**:
| 발견 | 권장 워크플로우 |
|------|----------------|
| 개선 필요 | `refactor` |
| 새 기능 아이디어 | `full-feature` |
| 버그 발견 | `quick-fix` |

---

## 🤖 Agents (서브에이전트)

Task tool로 호출하는 서브에이전트입니다.

### 에이전트 요약

| Agent | 역할 | 호출 시점 | 도구 |
|-------|------|----------|------|
| `planner` | 요구사항/질문셋 정리, 스펙 문서 | 새 기능, 요구사항 불명확 | Read, Write, Glob |
| `architect` | 구조 설계, 기술 부채 관리 | 구조 결정, 설계 필요 | Read, Grep, Glob |
| `executor` | 코드/씬/프리팹 구현 | 모든 구현 작업 | Read, Write, Edit, Bash, Grep, Glob |
| `critic` | 전략/리스크/대안 분석 | 주요 결정 전 | Read, Grep, Glob |
| `reviewer` | 코드/Unity/테스트 검증 | 구현 완료 후 | Read, Grep, Glob, Bash |
| `meta-agent` | .claude/ 설정 개선/생성 | 설정 문제/개선 | Read, Write, Edit, Glob, Grep |

### planner (플래너)

**핵심 책임**: 요구사항 수집 → 질문셋 작성 → task-id 생성 → 스펙 문서 작성

**질문셋 필수 요소**:
- 분석 축 (1~3개, 1-5 스케일)
- 옵션 성격: `[S]` Safe, `[B]` Balanced, `[I]` Ideal
- 권장 표시: `⭐ (권장)`
- 반드시 **권장 이유** 포함

**산출물**: `docs/tasks/<task-id>.md`

**🚨 출력 규칙**: 메인 에이전트는 planner 출력을 **전체 원본 그대로** 사용자에게 전달

---

### architect (아키텍트)

**핵심 책임**: 구조 분석 → 설계 결정 → 부채 관리 → 설계 문서 작성

**설계 문서 필수 섹션**:
- 개요 / 영향 범위 / 구조 / 인터페이스 / 의존성 / **트레이드오프**

**대안 제시 형식**:
```markdown
**Q1. [결정 사항]**
- A) [방안] - [설명]
- B) [방안] ⭐ (권장) - [설명]

**권장 이유:** [설명]
```

**Critic 자동 트리거**: 영향 파일 10개+, 새 의존성, 인터페이스 변경

**산출물**: `docs/design/<task-id>.md`

---

### executor (구현 담당)

**핵심 책임**: 코드/Unity 구현 → 테스트 작성 → 커밋 관리

**브랜치 규칙**:
1. prompt에 브랜치 명시 → 해당 브랜치
2. 현재 `agent/*` 형태 → **현재 브랜치 유지**
3. main에서 새 task → `agent/<task-id>` 생성

**Unity 필수 절차**:
```
.cs 수정 → unity.compilation.request → unity.compilation.await → unity.editor.console.read
```

**커밋 메시지**:
```
<type>(<scope>): <subject>

task-id: <task-id>
```

**산출물**: 코드, 씬/프리팹, 테스트, 커밋

---

### critic (크리틱)

**핵심 역할**: Devil's Advocate - 불편한 진실을 말함

**전략적 질문 (필수)**:
1. 이게 정말 필요한가?
2. 더 간단한 방법은 없는가?
3. 6개월 후에도 이해할 수 있는가?
4. 우리가 놓치고 있는 관점은?
5. 실패한다면 어떤 이유일까? (Pre-mortem)

**Anti-pattern 주의**:
| 패턴 | 증상 |
|------|------|
| 과도한 보수성 | 모든 변경에 "리스크" |
| 무비판적 동의 | 항상 "좋은 접근입니다" |
| 리스크 과소평가 | "큰 문제 없을 것" |

**산출물**: `docs/critic/<task-id>.md`

---

### reviewer (리뷰어)

**체크리스트**:
- **코드**: 네이밍, 중복, 에러 처리, 문서화
- **Unity**: 컴파일/런타임 에러, 씬 저장, Prefab
- **테스트**: 단위/통합/수동 테스트

**인라인 Retro 트리거**:
- 같은 파일 3회+ 수정
- 컴파일 에러 2회+
- 런타임 에러 발견
- 설계와 다른 구현

**산출물**: `docs/reviews/<task-id>.md`, (조건부) `docs/retro/<task-id>.md`

---

### meta-agent (메타 에이전트)

**용도**: .claude/ 설정 분석, 개선, 생성

**작업 유형**:
| 유형 | 작업 |
|------|------|
| 개선 | 출력 형식, 누락 정보, 역할 혼동, 트리거 오류 수정 |
| 생성 | 새 command, agent, skill, workflow 추가 |

**원칙**:
- 기존 스타일 따름
- 모든 수정은 사용자 승인 후
- CLAUDE.md 동기화 필수

---

## ⌨️ Commands (슬래시 명령)

### 명령어 요약

| 명령 | 용도 | 핵심 동작 |
|------|------|----------|
| `/sync` | GitHub 동기화 | 커밋 + 푸시 |
| `/finish` | 브랜치 완료 | 리뷰 확인 → 머지 또는 PR |
| `/status` | 프로젝트 상태 | Git + 태스크 + Unity 상태 요약 |
| `/new-task` | 새 태스크 | task-id + 브랜치 + 문서 생성 |
| `/context` | 컨텍스트 요약 | 현재 작업 상태 + 관련 문서 |
| `/workflow <name>` | 워크플로우 실행 | 프리셋 로드 + 첫 단계 실행 |
| `/pull-main` | main 최신화 | fetch + merge origin/main |
| `/reindex [full]` | 코드 인덱스 | 서버 시작 + 인덱싱 |
| `/reindex-clips [full]` | 클립 인덱스 | 클립 서버 + 인덱싱 |
| `/start-rmbg [gpu]` | 배경 제거 서버 | Python 서버 시작 |
| `/setup-rmbg` | RMBG 설정 | venv + 의존성 설치 |

---

### /sync (GitHub 동기화)

**흐름**: `git status` → 변경 요약 → 커밋 메시지 제안 → 승인 → 커밋 + 푸시

**주의**:
- main/master에서는 경고
- force push 금지

---

### /finish (브랜치 완료)

**완료 체크리스트**:
| 항목 | 필수 |
|------|------|
| 리뷰 문서 (`docs/reviews/<task-id>.md`) | ✅ |
| 컴파일 에러 없음 | ✅ |
| 태스크 문서 갱신 | ✅ |
| 레트로 문서 (문제 있었을 경우) | 조건부 |

**리뷰 미완료 시**:
- A) reviewer 호출
- B) 간략 리뷰 (mini-review)
- C) 건너뛰기 (사유 필요)

**반영 방식**: 바로 머지 / PR 생성 / 취소

---

### /new-task (새 태스크)

**질문**:
1. 우선순위 (H/M/L)
2. 담당자 이니셜
3. 키워드 (kebab-case)

**생성물**:
- task-id: `{priority}-{date}-{initials}-{keyword}`
- 브랜치: `agent/<task-id>`
- 문서: `docs/tasks/<task-id>.md`

---

### /context (컨텍스트 요약)

**출력 정보**:
- 브랜치, task-id, 미커밋 변경, 최근 커밋
- 관련 문서 존재 여부 (스펙/설계/리뷰/레트로/크리틱)
- 스펙 요약 (있을 경우)
- 권장 다음 단계

---

### /reindex (코드 인덱스)

**모드**:
- `/reindex` - incremental (변경 파일만)
- `/reindex full` - 전체 재구축

**자동 시작**: 서버 미실행 시 자동으로 시작 (`npm run dev -- http 17910`)

**인덱싱 옵션**:
```bash
npm run index              # 기본 (Tree-sitter + SCIP + Embedding)
npm run index -- --full    # 전체 재구축
npm run index -- --no-embeddings  # 빠른 (임베딩 없이)
```

**소요 시간**: 심볼 ~10초, SCIP ~5초, 임베딩 ~1-3분

---

### /start-rmbg (배경 제거 서버)

**모드**:
- `/start-rmbg` - CPU 모드
- `/start-rmbg gpu` - GPU 모드
- `/start-rmbg status` - 상태 확인만

**리소스**:
| 모드 | VRAM/RAM | 첫 추론 |
|------|----------|---------|
| CPU | ~1GB RAM | ~5초 |
| GPU | ~1GB VRAM | ~1초 |

---

## 📂 산출물 경로 참조

| 문서 타입 | 경로 |
|-----------|------|
| 스펙 문서 | `docs/tasks/<task-id>.md` |
| 설계 문서 | `docs/design/<task-id>.md` |
| 리뷰 문서 | `docs/reviews/<task-id>.md` |
| 레트로 문서 | `docs/retro/<task-id>.md` |
| 크리틱 문서 | `docs/critic/<task-id>.md` |
| 메타 개선 | `docs/meta/<날짜>-<개선내용>.md` |

---

## 🔗 전체 워크플로우 관계도

```
[사용자 요청]
    │
    ├── 새 기능? ──────────▶ /workflow full-feature
    │                              │
    │                              ├─▶ planner (질문셋)
    │                              ├─▶ critic (스펙 검토)
    │                              ├─▶ architect (설계)
    │                              ├─▶ critic (설계 검토)
    │                              ├─▶ executor (구현)
    │                              └─▶ reviewer (리뷰)
    │
    ├── 간단한 버그? ──────▶ /workflow quick-fix
    │                              │
    │                              ├─▶ executor
    │                              └─▶ reviewer
    │
    ├── 리팩토링? ─────────▶ /workflow refactor
    │                              │
    │                              ├─▶ critic (리스크 분석)
    │                              ├─▶ architect (설계)
    │                              ├─▶ executor (구현)
    │                              └─▶ reviewer (리뷰)
    │
    ├── 설계만? ───────────▶ /workflow design-only
    │                              │
    │                              ├─▶ planner
    │                              ├─▶ critic
    │                              ├─▶ architect
    │                              └─▶ critic
    │
    └── 코드 분석? ────────▶ /workflow exploration
                                   │
                                   └─▶ architect (분석 모드)

[작업 완료]
    │
    ├─▶ /sync (커밋 + 푸시)
    │
    └─▶ /finish (리뷰 확인 → 머지/PR)
```

---

*Last updated: 2026-01-13*
