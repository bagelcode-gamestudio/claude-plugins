---
name: bt-edit
description: BT (Behavior Tree) 편집/생성 가이드. BT 작업이나 피처 로직 수정 시 자동으로 사용됨.
trigger: BT, Behavior Tree, 피처 BT 관련 작업 시
priority: high
---

# BT (Behavior Tree) 편집 가이드

BT (Behavior Tree) 에셋을 편집하거나 생성할 때 사용합니다.

## 언제 사용하나요?

- "BT 만들어줘", "BT 수정해줘" 요청 시
- "Behavior Tree 추가", "피처 BT 생성" 요청 시
- GameLogic_*_BT.asset 파일 작업 시

## BT 에셋 위치

```
Contents Group 1/{GameName}/FSM/Game Logic/
├── GameLogic_Idle_BT.asset          # Idle 상태 BT
├── GameLogic_Free Game_BT.asset     # 프리게임 BT
├── GameLogic_Check*.asset           # Feature 체크 BT
│   ├── Check Buy a Bonus_BT.asset
│   ├── Check Lightning_BT.asset
│   ├── Check Jackpot_BT.asset
│   └── Check Big Win_BT.asset
└── GameLogic_Win*.asset             # 당첨 계산 BT
    ├── Win Way_BT.asset
    └── Win Classic_BT.asset
```

## 핵심 BT 노드 타입

| 노드 타입 | 용도 | 자식 필요 |
|-----------|------|----------|
| `Sequencer` | 순차 실행 | O |
| `Selector` | 첫 성공까지 실행 | O |
| `BinarySelector` | 조건 분기 (True/False) | O |
| `ActionNode` | Action 실행 | X |
| `WaitUntil` | 조건 대기 | **O (필수!)** |
| `Parallel` | 병렬 실행 | O |

## WaitUntil 필수 규칙

**중요**: WaitUntil 노드는 반드시 자식 노드가 있어야 합니다!

```json
// 잘못된 예 - WaitUntil만 있고 자식 없음 (오류!)
{"_condition":{"eventName":{"_value":"EndFeature"}},"$type":"NodeCanvas.BehaviourTrees.WaitUntil","$id":"5"}

// 올바른 예 - WaitUntil 아래 RaiseStatus/Success 추가
{"_condition":{"eventName":{"_value":"EndFeature"}},"$type":"NodeCanvas.BehaviourTrees.WaitUntil","$id":"5"},
{"_action":{"status":"Success","$type":"NodeCanvas.Tasks.Actions.RaiseStatus"},"_position":{"x":2000.0,"y":2360.0},"$type":"NodeCanvas.BehaviourTrees.ActionNode","$id":"8"}

// connections에 반드시 추가:
{"_sourceNode":{"$ref":"5"},"_targetNode":{"$ref":"8"},"$type":"NodeCanvas.BehaviourTrees.BTConnection"}
```

## Feature BT 표준 패턴

**BinarySelector + CheckBonus → Sequencer → SendContentEvent → WaitUntil**:

```json
{
  "nodes": [
    // 1. BinarySelector - 보너스 체크 조건
    {"_condition":{"bonusId":{"_value":33803},"$type":"BagelCode.Tasks.Conditions.Contents.CheckBonus"},"$type":"NodeCanvas.BehaviourTrees.BinarySelector","$id":"2"},

    // 2. Sequencer (True 분기)
    {"$type":"NodeCanvas.BehaviourTrees.Sequencer","$id":"3"},

    // 3. SendContentEvent - Module에 이벤트 발송
    {"_action":{"eventName":{"_value":"FeatureName"},"sendGlobal":true,"$type":"BagelCode.Tasks.Actions.SendContentEvent"},"$type":"NodeCanvas.BehaviourTrees.ActionNode","$id":"4"},

    // 4. WaitUntil - Module 완료 대기
    {"_condition":{"eventName":{"_value":"EndFeatureName"},"$type":"BagelCode.Task.Condition.CheckContentEvent"},"$type":"NodeCanvas.BehaviourTrees.WaitUntil","$id":"5"},

    // 5. WaitUntil 자식 (필수!)
    {"_action":{"status":"Success","$type":"NodeCanvas.Tasks.Actions.RaiseStatus"},"$type":"NodeCanvas.BehaviourTrees.ActionNode","$id":"6"}
  ],
  "connections": [
    {"_sourceNode":{"$ref":"2"},"_targetNode":{"$ref":"3"}},  // CheckBonus True → Sequencer
    {"_sourceNode":{"$ref":"3"},"_targetNode":{"$ref":"4"}},  // Sequencer → Send
    {"_sourceNode":{"$ref":"3"},"_targetNode":{"$ref":"5"}},  // Sequencer → Wait
    {"_sourceNode":{"$ref":"5"},"_targetNode":{"$ref":"6"}}   // WaitUntil → Success
  ]
}
```

## 피처 실행 타이밍

- **릴 정지 전 연출**: Add Wild → **Feature BT** → Update Deck (StopSlotMachine)
- **릴 정지 후 연출**: Update Deck → StoppedSlotMachine → **Feature BT** → Win

## 새 BT 노드 추가 순서

1. nodes 배열에 노드 추가
2. 고유 $id 부여
3. _position 설정
4. _action 또는 _condition 설정
5. connections에 부모-자식 연결 추가

## 주요 Condition 타입

| Condition | 용도 |
|-----------|------|
| `CheckBonus` | 보너스 존재 확인 |
| `CheckContentEvent` | 이벤트 발생 확인 |
| `CheckBigWin` | 빅윈 조건 확인 |
| `CheckBoolean` | Blackboard 값 체크 |

## 주요 Action 타입

| Action | 용도 |
|--------|------|
| `SendContentEvent` | 이벤트 발송 |
| `BeginSpinBonus` | 보너스 시작 |
| `EndBonus` | 보너스 종료 |
| `RaiseStatus` | Success/Failure 반환 |

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `fsm-edit` | FSM에서 BT 참조 설정 |
| `new-action` | 커스텀 Action 작성 |
| `feature-controller` | Feature 스크립트 |
