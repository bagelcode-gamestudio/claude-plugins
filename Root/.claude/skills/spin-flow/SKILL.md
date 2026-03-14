---
name: spin-flow
description: 게임 스핀 전체 플로우 가이드. 스핀 흐름, 게임 플로우 확인 시 자동으로 사용됨.
trigger: 스핀 흐름, 게임 플로우, 스핀 어떻게 동작해 관련 질문 시
priority: medium
---

# 게임 스핀 전체 플로우

슬롯 게임 스핀의 클라이언트-서버 전체 흐름을 설명합니다.

## 언제 사용하나요?

- "스핀 흐름", "스핀 플로우 알려줘" 요청 시
- "스핀 어떻게 동작해", "게임 흐름" 요청 시

## 전체 플로우 요약

```
클라이언트 (FSM)          서버 (LOGIC)           엔진 (Games)
     │                        │                      │
1. BeginSpin              2. 요청 검증              │
     │─── POST /v3/slot/spin ─→│                     │
     │                    3. 엔진 호출 ──────────→ 4. RNG 생성
     │                        │                   5. 당첨 계산
     │                        │←── 결과 반환 ────── 6. 보너스 결정
     │←── HTTP Response ──────│                      │
7. Blackboard 저장            │                      │
8. UpdateDeck                 │                      │
9. StopSlotMachine            │                      │
10. CalcWin                   │                      │
11. CheckBonus                │                      │
12. EndTurn ─── POST /v3/slot/end-turn ─→│          │
```

## 1. 클라이언트 - 스핀 요청

```
FSM State: Begin Spin
├─ Action: BeginSpin.cs
│  └─ Blackboard에 ./spin/ 구조 생성
│
└─ Action: Spin.cs
   └─ MetaSystem.SlotSpin(betCredit, customData, callback)
      └─ HTTP POST /v3/slot/spin
```

**요청 데이터**:
```json
{
  "game_id": 343,
  "bet_credit": 500,
  "contents": { "CustomData343": {...} },
  "is_game_spin": true
}
```

## 2. LOGIC 서버 - 요청 처리

```
route_slot.ts → doSpinV2()
├─ 1) 요청 검증 (파라미터, 세션, 크레딧)
├─ 2) Games 엔진 호출
│     └─ logic_slot.spin() → getGameResultPromise()
└─ 3) DB 업데이트 (크레딧 차감, 턴 기록)
```

## 3. Games 엔진 - 결과 생성

```
logic_slot.ts → getGameResultPromise()
├─ 1) 게임 모듈 로드: game_manager.getGameModule(343)
├─ 2) RNG 생성: reelOutputList = [12, 45, 23, 67, 34, 89]
├─ 3) 당첨 계산: 심볼 그리드 → 페이테이블 → earnCredit
├─ 4) 보너스 결정: Scatter 3개 → bonusId 34300
└─ 5) 결과 반환
```

## 4. 응답 구조

```json
{
  "error": 0,
  "contents": {
    "result": {
      "reelOutputList": [12, 45, 23, 67, 34, 89],
      "earnCredit": 80000,
      "betCredit": 500
    },
    "customData": { "CustomData343": {...} },
    "bonusList": [
      { "bonusId": 34300, "initialSpinCount": 8 }
    ]
  },
  "user_sync_info": {
    "credit": 999500,
    "level": 42
  }
}
```

## 5. 클라이언트 - 응답 처리

```
MetaSystem 콜백
├─ 1) Blackboard에 응답 저장
│     ./spin/response/result/reelOutputList
│     ./spin/response/customData
│     ./spin/response/bonusList
│     /me/credit
├─ 2) FSM 계속 진행
└─ Action: UpdateDeck.cs
   ├─ reelOutputList로 Deck 구성
   └─ SlotMachine에 인덱스 전달
```

## 6. 클라이언트 - 결과 표시

```
FSM 흐름:
├─ SendSlotEvent("StopSlotMachine")
│  └─ 릴 정지 애니메이션
│     └─ SymbolBehaviour: OnPrepareStop() → OnStopEffect()
│
├─ CalcWayWin.cs
│  └─ 당첨 라인 계산 및 표시
│
├─ CheckBonus(34300)
│  └─ 프리게임 존재 → BeginSpinBonus(34300)
│     └─ FreeSpin_FSM 실행
│
├─ BigWin 처리
│  └─ earnCredit >= 5 × betCredit → 빅윈 팝업
│
└─ RequestEndTurn()
   └─ POST /v3/slot/end-turn
```

## 주요 API

| API | 용도 |
|-----|------|
| `POST /v3/slot/spin` | 스핀 요청 |
| `POST /v3/slot/claim` | 보너스 클레임 |
| `POST /v3/slot/end-turn` | 턴 종료 |
| `POST /v3/slot/boast` | 빅윈 자랑 |

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `bonus-flow` | 보너스 처리 흐름 |
| `blackboard-path` | 데이터 경로 |
| `fsm-edit` | FSM 상태 수정 |
