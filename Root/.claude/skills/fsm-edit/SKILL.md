---
name: fsm-edit
description: FSM (Finite State Machine) 편집/생성 가이드. FSM 작업이나 상태 머신 수정 시 자동으로 사용됨.
trigger: FSM, 상태 머신, State Machine 관련 작업 시
priority: high
---

# FSM 편집 가이드

FSM (Finite State Machine) 에셋을 편집하거나 생성할 때 사용합니다.

## 언제 사용하나요?

- "FSM 만들어줘", "FSM 수정해줘" 요청 시
- "상태 머신 추가", "State 편집" 요청 시
- GameLogic_FSM.asset 파일 작업 시

## FSM 에셋 위치

```
Contents Group 1/{GameName}/FSM/Game Logic/
├── GameLogic_FSM.asset              # 메인 State Machine
├── GameLogic_FreeSpin_FSM.asset     # 서브 FSM (프리스핀)
└── GameLogic_*.asset                # 기타 FSM
```

## 메인 FSM 상태 흐름

```
Begin Game → Intro → Idle → GameSpin → TrySpin
                              ↓
                         BeginTurn → BeginSpin
                              ↓
                         Initialization → UpdateDeck
                              ↓
                         Lightning → Multiplier Lightning
                              ↓
                         BigWin → Win → EndSpin → EndTurn
                              ↓
                         [루프: Idle로 복귀]
```

## FSM에서 BT 참조 (NestedBTState)

```json
{
  "_nestedBT": {"_value": 11},
  "executionMode": "Once",
  "_position": {"x": 7869.0, "y": 5156.0},
  "_name": "Feature Name",
  "$type": "NodeCanvas.StateMachines.NestedBTState",
  "$id": "300"
}
```

**_objectReferences에 BT 추가**:
```yaml
_objectReferences:
  - {fileID: 11400000, guid: <BT파일GUID>, type: 2}  # index 11
```

## FSM State 타입

| State 타입 | 용도 |
|------------|------|
| `ActionState` | 단일 Action 실행 |
| `NestedBTState` | BT 실행 후 자동 전환 |
| `NestedFSMState` | 서브 FSM 실행 |
| `ConcurrentState` | 병렬 실행 |

## FSM 전환 조건

```json
{
  "_sourceNode": {"$ref": "10"},
  "_targetNode": {"$ref": "20"},
  "_condition": {
    "$type": "BagelCode.Tasks.Conditions.Contents.CheckBonus",
    "bonusId": {"_value": 34300}
  },
  "$type": "NodeCanvas.StateMachines.FSMConnection"
}
```

## 새 State 추가 순서

1. nodes 배열에 State 추가
2. 고유 $id 부여
3. _position 설정 (x, y 좌표)
4. connections에 전환 조건 추가
5. _objectReferences에 참조 에셋 추가 (필요시)

## 템플릿 위치

```
_Common/Template/Game Logic/Slot/FSM/
├── Game Logic/
│   ├── GameLogic_FSM.asset
│   └── GameLogic_Idle_BT.asset
└── Popup/
```

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `bt-edit` | BT 에셋 편집 |
| `new-action` | 커스텀 Action 작성 |
| `bonus-flow` | 보너스 상태 추가 시 |
