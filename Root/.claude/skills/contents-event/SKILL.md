---
name: contents-event
description: Feature Module(Controller) 간 Contents Event 통신 규격 가이드. BT/FSM과 스크립트 간 이벤트 통신 시 자동으로 사용됨.
trigger: 이벤트 통신, SendContentEvent, FeatureController 이벤트, 피처 간 통신 관련 작업 시
priority: high
---

# Contents Event 통신 가이드

Feature Module/Controller 간의 이벤트 통신 규격을 안내합니다.

## 언제 사용하나요?

- BT/FSM에서 피처 스크립트 호출 시
- 피처 간 통신 구현 시
- 커스텀 이벤트 정의 시
- "이벤트 보내줘", "피처 호출" 요청 시

## 이벤트 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                    MessageDispatcher                         │
│              (전역 Pub/Sub 이벤트 버스)                       │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ "OnContentEvent" 채널
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      ContentEvent                            │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ 라이프사이클 이벤트                                    │   │
│   │ - BeginGame/EndGame                                  │   │
│   │ - BeginTurn/EndTurn                                  │   │
│   │ - BeginSpin/EndSpin                                  │   │
│   │ - BeginBonus/EndBonus                                │   │
│   └─────────────────────────────────────────────────────┘   │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ 커스텀 이벤트                                         │   │
│   │ - SendEvent(eventName)                               │   │
│   │ - SendEvent<T>(eventName, data)                      │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
            ┌───────────────┼───────────────┐
            ▼               ▼               ▼
    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
    │FeatureModule │ │FeatureModule │ │FeatureModule │
    │ (Listener A) │ │ (Listener B) │ │ (Listener C) │
    └──────────────┘ └──────────────┘ └──────────────┘
```

## 이벤트 발송 방법

### 1. FSM/BT에서 발송 (Action)

```
SendContentEvent
├── eventName: "MyFeature_Begin"    # 이벤트 이름
├── delay: 0                        # 지연 시간 (초)
└── sendGlobal: true                # 전역 발송 (항상 true)
```

**제네릭 버전** (데이터 포함):
```
SendContentEvent<Cell>
├── eventName: "SymbolLanded"
├── eventValue: ./currentCell       # Blackboard 경로
├── delay: 0
└── sendGlobal: true
```

### 2. Script에서 발송

```csharp
// 단순 이벤트
ContentEvent.SendEvent("MyFeature_Begin");

// 데이터 포함
ContentEvent.SendEvent<Cell>("SymbolLanded", cell);
ContentEvent.SendEvent<int>("LevelUp", newLevel);
ContentEvent.SendEvent<Blackboard>("BonusData", bonusBlackboard);
```

## 이벤트 수신 방법

### FeatureController (권장)

가장 간단한 패턴. Begin/End 이벤트 쌍으로 자동 관리됩니다.

```csharp
public class MyFeatureController : FeatureControllerAsync
{
    // 이벤트 이름 정의
    protected override string ON_FEATURE_BEGIN_EVENT => "MyFeature_Begin";
    protected override string ON_FEATURE_END_EVENT => "MyFeature_End";

    protected override async UniTask OnPlay()
    {
        // 연출 실행
        await PlayAnimation();
        // OnPlay 완료 시 자동으로 END_EVENT 발송
    }
}
```

**자동 흐름**:
```
BT: SendContentEvent("MyFeature_Begin")
  → FeatureControllerAsync.OnFeaturePlay() 트리거
  → OnStart() → OnPlay() → OnFinish()
  → ContentEvent.SendEvent("MyFeature_End") 자동 발송
  → BT: WaitUntilEvent 완료
```

### FeatureModule (복잡한 경우)

여러 이벤트를 독립적으로 처리해야 할 때 사용합니다.

```csharp
public class MyPotAdmin : FeatureModuleAsync
{
    protected override void OnEnable()
    {
        base.OnEnable();
        // 여러 이벤트 핸들러 등록
        RegisterEvent("LightningSymbolLanded", OnLightningLanded);
        RegisterEvent("CollectAllPots", OnCollectAllPots);
        RegisterEvent("ResetPots", OnResetPots);
    }

    protected override void OnDisable()
    {
        base.OnDisable();
        UnRegisterEvent("LightningSymbolLanded");
        UnRegisterEvent("CollectAllPots");
        UnRegisterEvent("ResetPots");
    }

    private void OnLightningLanded(EventData eventData)
    {
        // eventData.value로 데이터 접근
        var cell = eventData.value as Cell;
        ShowPotEffect(cell);
    }

    private void OnCollectAllPots(EventData eventData)
    {
        CollectAnimation();
        // 완료 후 응답 이벤트 발송
        ContentEvent.SendEvent("CollectAllPotsComplete");
    }
}
```

## 이벤트 네이밍 컨벤션

### Begin/End 쌍 (피처 라이프사이클)

```
{FeatureName}_Begin  →  {FeatureName}_End
{FeatureName}Begin   →  {FeatureName}End

예시:
- FreeSpin_Begin → FreeSpin_End
- LightningFeature_Begin → LightningFeature_End
- GoldenBarbarianLanded → GoldenBarbarianLandedComplete
```

### 단일 이벤트 (상태 변경/알림)

```
On{Action}              # 동작 발생
{Subject}{Action}       # 주체+동작
{Action}Complete        # 완료 알림

예시:
- OnSymbolLanded
- PotCollected
- WheelSpinComplete
- MultiplierApplied
```

### 제어 이벤트 (명령)

```
Do{Action}              # 동작 요청
Request{Action}         # 요청
Trigger{Feature}        # 피처 트리거

예시:
- DoCollectWin
- RequestRetrigger
- TriggerJackpotAnimation
```

## 데이터 전달 타입

| 타입 | 용도 | 예시 |
|------|------|------|
| `Cell` | 셀 위치 | 심볼 랜딩 위치 |
| `int` | 숫자값 | 레벨, 인덱스, 카운트 |
| `long` | 금액 | 당첨금, 잭팟 금액 |
| `string` | 식별자 | 심볼 이름, 상태 |
| `Blackboard` | 복합 데이터 | 보너스 결과 전체 |
| `List<T>` | 배열 | 당첨 셀 목록 |
| `GameObject` | UI 참조 | 애니메이션 타겟 |

## BT/FSM 이벤트 흐름 예시

### 프리스핀 보너스 흐름

```
┌─────────────────────────────────────────────────────────┐
│ FreeSpin FSM/BT                                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  [CheckBonus(34300)]                                    │
│         │ true                                           │
│         ▼                                                │
│  [BeginSpinBonus(34300)]                                │
│         │                                                │
│         ▼                                                │
│  [SendContentEvent: "FreeSpin_Begin"] ──────────────────┼──→ FreeSpinController.OnPlay()
│         │                                                │
│         ▼                                                │
│  [WaitUntilEvent: "FreeSpin_End"] ←─────────────────────┼─── ContentEvent.SendEvent("FreeSpin_End")
│         │                                                │
│         ▼                                                │
│  [Nested FSM: FreeSpin Loop]                            │
│         │                                                │
│         ▼                                                │
│  [EndBonus]                                              │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 심볼 랜딩 이벤트 흐름

```
┌─────────────────────────────────────────────────────────┐
│ Reel Stop BT                                             │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  [ForEach: stoppedCells]                                │
│         │                                                │
│         ▼                                                │
│  [CheckSymbol: "GoldenBarbarian"]                       │
│         │ true                                           │
│         ▼                                                │
│  [SendContentEvent<Cell>: "GoldenBarbarianLanded"] ─────┼──→ BBDGoldenBarbarianController
│         │                                                │           .OnPlay(cell)
│         ▼                                                │
│  [WaitUntilEvent: "GoldenBarbarianLandedComplete"] ←────┼─── OnPlay 완료
│         │                                                │
│         ▼                                                │
│  [Continue Loop...]                                      │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## 이벤트 데이터 접근

### FSM/BT에서 이벤트 값 읽기

```
SetBlackboardValue
├── path: ./eventValue
└── value: ${receivedEvent.value}  # 이벤트로 받은 값
```

### Script에서 이벤트 값 읽기

```csharp
private void OnMyEvent(EventData eventData)
{
    // 타입 캐스팅
    var cell = eventData.value as Cell;
    var amount = (long)eventData.value;

    // 또는 BlackboardUtils 사용
    var value = BlackboardUtils.FindValue<Cell>(null, "eventValue");
}
```

## 이벤트 채널 종류

| 채널 | 상수 | 용도 |
|------|------|------|
| `OnContentEvent` | 기본 | Feature 간 통신 |
| `OnSlotEvent` | 슬롯 | 릴/심볼 이벤트 |
| `OnSlotDetailEvent` | 슬롯 상세 | 릴 정지 등 |
| `OnContentUIEvent` | UI | UI 상태 변경 |
| `OnCreditEvent` | 크레딧 | 잔액 변경 |
| `OnSoundEvent` | 사운드 | BGM/효과음 |
| `OnWinEvent` | 당첨 | 당첨금 표시 |

### 다른 채널 구독 예시

```csharp
protected override void OnEnable()
{
    base.OnEnable();
    MessageDispatcher.Register("OnSlotDetailEvent", OnSlotDetail);
}

private void OnSlotDetail(EventData eventData)
{
    if (eventData.name == "PrepareStoppedReel")
    {
        int reelIndex = (int)eventData.value;
        // 릴 정지 직전 처리
    }
}
```

## 주의사항

1. **Begin 이벤트 후 반드시 End 이벤트 발송**
   - WaitUntilEvent로 대기 중인 FSM/BT가 무한 대기할 수 있음

2. **sendGlobal = true 필수**
   - 로컬 이벤트는 같은 GameObject 내에서만 동작

3. **이벤트 이름 중복 주의**
   - 게임별 프리픽스 사용 권장 (예: `BBD_LightningLanded`)

4. **이벤트 핸들러 해제**
   - OnDisable에서 반드시 UnRegisterEvent 호출

5. **🚨 연출 구현 시 반드시 사용자에게 질문**
   - 연출(Animation/Effect) 관련 작업 전에 반드시 확인:
     - 현재 어디까지 구현되어 있는지?
     - 연출을 어떻게 처리할지? (placeholder, TODO, 실제 구현)
   - TA 작업과 분리해야 하는 부분은 TODO 주석으로 표시

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `feature-controller` | FeatureController 구현 |
| `bt-edit` | BT에서 이벤트 사용 |
| `fsm-edit` | FSM에서 이벤트 사용 |
| `bonus-flow` | 보너스 이벤트 흐름 |
