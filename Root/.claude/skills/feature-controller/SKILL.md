---
name: feature-controller
description: FeatureController 구현 가이드. 피처, Controller 스크립트 작성 시 자동으로 사용됨.
trigger: 피처, FeatureController, FeatureModule 관련 작업 시
priority: high
---

# FeatureController 구현 가이드

슬롯 게임의 Feature (피처) 스크립트를 작성할 때 사용합니다.

## 언제 사용하나요?

- "피처 만들어줘", "Feature 스크립트 작성" 요청 시
- "라이트닝 구현", "프리스핀 로직" 요청 시
- FeatureController/FeatureModule 작업 시

## FeatureController vs FeatureModule

| 클래스 | 용도 | 선택 기준 |
|--------|------|----------|
| `FeatureController` | 단순 피처 | 동기적, 단순 연출 |
| `FeatureControllerAsync` | 비동기 피처 | UniTask 기반, 복잡 연출 |
| `FeatureModule` | 복잡 로직 | 여러 이벤트 조합 |
| `FeatureModuleAsync` | 복잡 비동기 | 코디네이터 역할 |

**우선 순위**: FeatureController > FeatureModule (단순한 것 먼저)

## FeatureControllerAsync 템플릿

```csharp
public class MyFeatureController : FeatureControllerAsync
{
    // 이벤트 이름 정의
    protected override string ON_FEATURE_BEGIN_EVENT => "MyFeature_Begin";
    protected override string ON_FEATURE_END_EVENT => "MyFeature_End";

    [SerializeField] private GameObject effect;

    protected override async UniTask OnPlay()
    {
        // 1. Blackboard에서 데이터 읽기
        var bb = ContentBlackboard.Get();
        var bonusData = bb.GetValue<BonusResult>("./bonus/response");

        // 2. 연출 실행
        effect.SetActive(true);
        await WaitForSeconds(1.5f);

        // 3. 추가 로직
        foreach (var item in bonusData.items)
        {
            await PlayItemEffect(item);
        }

        // 4. 연출 종료
        effect.SetActive(false);
        // OnPlay 완료 시 자동으로 END_EVENT 발송됨
    }
}
```

## FeatureModuleAsync 템플릿

```csharp
public class MyPotAdmin : FeatureModuleAsync
{
    protected override void OnEnable()
    {
        // 이벤트 리스너 등록
        RegisterEvent("OnLightningLanded", OnLightningHandler);
        RegisterEvent("BeginLightning", OnBeginLightning);
        RegisterEvent("EndLightning", OnEndLightning);
    }

    private void OnBeginLightning(object data)
    {
        var bb = ContentBlackboard.Get();
        var bonus = bb.GetValue<BonusResult34302>("./bonus/response");

        foreach (var index in bonus.hitValueIndicies)
        {
            pots[index].ShowLightningEffect();
        }

        SendContentEvent("EndLightning");
    }
}
```

## BT vs Script 역할 분리 (중요!)

**BT에서 처리**:
- 보너스 체크 (CheckBonus)
- 조건 분기, 흐름 제어
- 이벤트 발송/대기

**Script에서 처리**:
- 타이밍/연출만 담당
- 이펙트 활성화/비활성화
- 사운드 재생
- **보너스 검사 X** (BT에서 처리!)

```csharp
// 잘못된 예 - Script에서 보너스 체크 (하지마세요!)
public async UniTask PlayFeature()
{
    if (!GameStudioUtils.CheckBonus(BONUS_ID)) return; // X
    // ...
}

// 올바른 예 - BT에서 CheckBonus 후 호출되므로 바로 연출
public async UniTask PlayFeature()
{
    effect.SetActive(true);
    await WaitForSeconds(duration);
    effect.SetActive(false);
    ContentEvent.SendEvent("EndFeature");
}
```

## 이벤트 흐름

```
BT: SendContentEvent("MyFeature_Begin")
    → FeatureControllerAsync.OnFeaturePlay()
    → OnPlay() 실행
    → ContentEvent.SendEvent("MyFeature_End")
    → BT: WaitUntil 완료 → FSM 계속
```

## 슬롯 이벤트 구독

```csharp
protected override void OnEnable()
{
    base.OnEnable();
    MessageDispatcher.Register(ON_SLOT_EVENT, OnSlotEvent);
    MessageDispatcher.Register(ON_SLOT_DETAIL_EVENT, OnSlotDetailEvent);
}

protected virtual void OnSlotDetailEvent(EventData receivedEvent)
{
    if (receivedEvent.name.Equals("PrepareStoppedReel"))
    {
        int currentStopReel = (int)receivedEvent.value;
        // 릴 정지 시 처리
    }
}
```

## 파일 위치

```
Contents Group 1/{GameName}/Scripts/
├── {Game}PotAdmin.cs         # 메인 Feature Module
├── {Game}Pot.cs              # 개별 팟 로직
├── {Game}Lightning.cs        # 라이트닝 피처
└── Features/
    └── {Feature}Controller.cs
```

## 🚨 연출 구현 시 필수 확인

> **중요**: 연출(Animation/Effect) 관련 작업 전에 반드시 사용자에게 질문하세요!

### 확인해야 할 사항

1. **현재 구현 상태**
   - 애니메이션/이펙트가 어디까지 구현되어 있는지?
   - Animator, Particle, Spine 등 어떤 리소스가 준비되어 있는지?

2. **연출 처리 방법**
   - 실제 구현: 리소스가 준비된 경우
   - Placeholder: 임시 대체 (WaitForSeconds, Debug.Log)
   - TODO 주석: TA 작업 필요 부분 표시

### 질문 예시

```
연출 관련 확인이 필요합니다:
1. Golden Barbarian 랜딩 이펙트가 준비되어 있나요?
2. Split 애니메이션 Animator가 있나요?
3. 없다면 placeholder로 처리할까요, TODO로 남겨둘까요?
```

### TA 작업 분리 표시

```csharp
// TODO: [TA] Split 애니메이션 구현 필요
// - Animator: cashSplitTrigger, wheelSplitTrigger
// - 예상 duration: 1.0s
yield return new WaitForSeconds(splitAnimationDuration);
```

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `bt-edit` | Feature BT 연결 |
| `bonus-flow` | 보너스 처리 흐름 |
| `symbol-behaviour` | 심볼 동작 |
| `contents-event` | BT/FSM ↔ Script 이벤트 통신 |
