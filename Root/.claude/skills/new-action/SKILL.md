---
name: new-action
description: Custom Action/Condition 작성 가이드. 새 액션, 커스텀 액션 추가 시 자동으로 사용됨.
trigger: 새 액션, 커스텀 액션, Action/Condition 작성 관련 작업 시
priority: medium
---

# Custom Action 작성 가이드

SlotMaker용 새 Action/Condition을 작성할 때 사용합니다.

## 언제 사용하나요?

- "새 액션 만들어줘", "커스텀 액션 추가" 요청 시
- FSM/BT에서 새 기능 필요 시

## Action 템플릿

```csharp
using NodeCanvas.Framework;
using ParadoxNotion.Design;
using SlotMaker.Core;

namespace SlotMaker.Tasks.Actions
{
    [Category("SlotMaker/Custom")]
    [Description("액션 설명")]
    public class MyCustomAction : SlotActionTask
    {
        [RequiredField]
        public BBParameter<string> inputParam;

        public BBParameter<int> optionalParam = new BBParameter<int> { value = 0 };

        [BlackboardOnly]
        public BBParameter<long> outputResult;

        protected override string info => $"MyCustomAction({inputParam})";

        protected override void OnExecute()
        {
            // Blackboard 값 읽기
            var value = inputParam.value;

            // 로직 처리
            var result = ProcessLogic(value);

            // 결과 저장
            outputResult.value = result;

            EndAction(true);
        }

        private long ProcessLogic(string input)
        {
            // 구현
            return 0;
        }
    }
}
```

## Condition 템플릿

```csharp
using NodeCanvas.Framework;
using ParadoxNotion.Design;
using SlotMaker.Core;

namespace SlotMaker.Tasks.Conditions
{
    [Category("SlotMaker/Custom")]
    [Description("조건 설명")]
    public class MyCustomCondition : SlotConditionTask
    {
        [RequiredField]
        public BBParameter<int> checkValue;

        public BBParameter<int> threshold = new BBParameter<int> { value = 10 };

        protected override string info => $"Is {checkValue} > {threshold}?";

        protected override bool OnCheck()
        {
            return checkValue.value > threshold.value;
        }
    }
}
```

## 파일 위치

```
client/Assets/SlotMaker/NodeCanvas/Tasks/Actions/Custom/
client/Assets/SlotMaker/NodeCanvas/Tasks/Conditions/Custom/
```

## 주요 베이스 클래스

| 클래스 | 용도 |
|--------|------|
| `SlotActionTask` | Action 기본 클래스 |
| `SlotConditionTask` | Condition 기본 클래스 |
| `BBParameter<T>` | Blackboard 파라미터 |

## Blackboard 경로 규칙

| 경로 | 용도 |
|------|------|
| `./spin/` | 현재 스핀 |
| `./bonus/` | 보너스 |
| `/me/` | 유저 정보 |
| `./customData/` | 커스텀 데이터 |

## 기존 Action 참고 (635개)

| Action | 용도 |
|--------|------|
| `BeginSpin.cs` | 스핀 시작 |
| `UpdateDeck.cs` | 덱 업데이트 |
| `CalcWayWin.cs` | 당첨 계산 |
| `CheckBonus.cs` | 보너스 확인 |
| `SendSlotEvent.cs` | 슬롯 이벤트 전송 |

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `fsm-edit` | FSM 편집 |
| `bt-edit` | BT 편집 |
| `blackboard-path` | 데이터 경로 |
