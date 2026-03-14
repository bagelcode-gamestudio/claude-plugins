---
name: bonus-flow
description: 보너스 생명주기 가이드. 프리스핀, 픽보너스, 라이트닝 등 보너스 구현 시 자동으로 사용됨.
trigger: 보너스, 프리스핀, ClaimBonus, BeginBonus 관련 작업 시
priority: high
---

# 보너스 생명주기 가이드

슬롯 게임의 보너스 처리 흐름을 안내합니다.

## 언제 사용하나요?

- "보너스 구현해줘", "프리스핀 추가해줘" 요청 시
- 보너스 관련 버그 수정 시
- 보너스 FSM/BT 작성 시

## 보너스 생명주기

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 트리거 (Spin 응답에서 bonusList 확인)                     │
│    └─ CheckBonus(bonusId) → true                           │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. BeginBonus (./bonus/ Blackboard 생성)                    │
│    └─ BeginSpinBonus(bonusId)                              │
│    └─ ./bonus/bonusId, ./bonus/spinCount 등 초기화         │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. ClaimBonus (서버에 보너스 결과 요청) - ClaimType 확인!   │
│    └─ ClaimType.ALWAYS: 반드시 호출                        │
│    └─ ClaimType.NEVER: 호출 안함                           │
│    └─ ClaimType.CONDITIONAL: 조건부 호출                   │
│    └─ POST /v3/slot/claim → 보너스 결과 저장               │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. 보너스 실행 (NestedFSM/NestedBT)                         │
│    └─ 프리스핀 루프                                         │
│    └─ Pick 보너스                                           │
│    └─ Wheel 보너스                                          │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. EndBonus (보너스 종료)                                   │
│    └─ EndBonus()                                            │
│    └─ ./bonus/ Blackboard 정리                             │
│    └─ 총 획득금 표시                                        │
└─────────────────────────────────────────────────────────────┘
```

## 보너스 ID 규칙

```
기본: gameId × 100 + sequence
예시:
  - 게임 343 → 34300 (Free Game), 34301 (Pick), 34302 (Lightning)
  - 게임 282 → 28200, 28201, 28202...

티켓형: gameId × 100 + 50 + sequence
  - 게임 343 → 34350 (Ticket Bonus)
  - EQUIV_BONUS_ID로 일반 보너스와 연결
```

## 보너스 타입별 처리

### Free Game (프리스핀)
```
FSM 흐름:
1. CheckBonus(34300) → BeginSpinBonus(34300)
2. ClaimBonus (필요시)
3. NestedFSM: FreeSpin_FSM
   └─ Loop: SpinBonus → UpdateDeck → CalcWin → CheckRetrigger
4. EndBonus → TotalWin 표시
```

### Pick Bonus
```
FSM 흐름:
1. CheckBonus(34301) → BeginSpinBonus(34301)
2. ClaimBonus → 서버에서 모든 선택지 결과 받음
3. NestedBT: PickBonus_BT
   └─ 유저 선택 → 결과 표시 (이미 결정됨)
4. EndBonus
```

### Lightning/Hold & Spin
```
FSM 흐름:
1. CheckBonus(34302) → BeginSpinBonus(34302)
2. Initial: 트리거 심볼 고정
3. Loop: 3회 리스핀 (새 심볼 출현 시 리셋)
4. 모든 위치 채워지면 Grand Jackpot
5. EndBonus
```

## Blackboard 구조

```
./bonus/
├── bonusId: 34300
├── spinCount: 5              # 남은 스핀
├── totalSpinCount: 8         # 총 스핀
├── earnCredit: 50000         # 보너스 획득금
├── response/
│   ├── initialSpinCount: 8
│   ├── result: {...}
│   └── customData: {...}
└── multiplier: 2             # 배수 (있는 경우)
```

## 중요 규칙

1. **ClaimType.ALWAYS면 ClaimBonus 필수 호출**
2. **BT에서 CheckBonus 사용** (스크립트에서 직접 체크 X)
3. **피처는 릴 정지 전에 실행** (타이밍 중요)
4. **EndBonus 전에 모든 연출 완료**

## 🚨 ClaimType 확인 필수

> **중요**: 보너스 구현 전에 반드시 ClaimType을 확인하세요!

### ClaimType 종류

| ClaimType | 의미 | ClaimBonus 호출 |
|-----------|------|-----------------|
| `ALWAYS` | 항상 서버에 claim 필요 | ✅ 필수 |
| `NEVER` (none) | claim 불필요 | ❌ 호출 안함 |
| `CONDITIONAL` | 조건부 claim | ⚠️ 조건 확인 |

### 확인 방법

**1. 서버 코드에서 확인** (권장)
```
games/src/slot_game/{game_name}.ts

// 예시: barbarian_destroyer.ts
export const bonusType = {
  FREE_SPIN: {
    // claimType - none      ← 여기서 확인!
    BONUS_ID: 23400,
  },
  INFINITY_REEL: {
    // claimType - always    ← ClaimBonus 필수!
    BONUS_ID: 23403,
  },
}
```

**2. 사용자에게 질문**
```
ClaimType 확인이 필요합니다:
- 보너스 ID: 23403 (INFINITY_REEL)
- ClaimType이 ALWAYS인가요, NEVER인가요?
- 서버 코드(games/src/slot_game/barbarian_destroyer.ts)에서 확인해주세요.
```

### ClaimType별 FSM/BT 흐름

**ALWAYS (claim 필수)**
```
CheckBonus → BeginSpinBonus → ClaimBonus → 보너스 실행 → EndBonus
```

**NEVER (claim 불필요)**
```
CheckBonus → BeginSpinBonus → 보너스 실행 → EndBonus
```

### 질문 템플릿

```
보너스 구현 전 확인이 필요합니다:

1. 보너스 ID: {bonusId}
2. ClaimType: ALWAYS / NEVER / CONDITIONAL ?
   - 서버 코드 위치: games/src/slot_game/{game}.ts
   - bonusType.{BONUS_NAME}.BONUS_ID 근처 주석 확인

ClaimType을 알려주시거나, 서버 코드를 확인해도 될까요?
```

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `fsm-edit` | FSM에서 보너스 상태 추가 |
| `bt-edit` | 보너스 BT 작성 |
| `blackboard-path` | 보너스 데이터 경로 |
