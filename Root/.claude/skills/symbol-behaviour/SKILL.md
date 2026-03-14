---
name: symbol-behaviour
description: SymbolBehaviour 구현 가이드. 심볼 동작, Scatter, Wild 스크립트 작성 시 자동으로 사용됨.
trigger: 심볼 동작, SymbolBehaviour, Scatter, Wild, 심볼 라이프사이클 관련 작업 시
priority: medium
---

# SymbolBehaviour 구현 가이드

심볼별 동작(Behaviour)을 구현할 때 사용합니다.

## 언제 사용하나요?

- "심볼 동작 만들어줘", "SymbolBehaviour 작성" 요청 시
- "Scatter 효과", "Wild 애니메이션" 요청 시
- 심볼 라이프사이클 관련 작업 시

## SymbolBehaviour 템플릿

```csharp
public class MySymbolBehaviour : SymbolBehaviour
{
    [SerializeField] private Animator animator;
    [SerializeField] private ParticleSystem landEffect;

    public override void OnEntry()
    {
        // 심볼이 화면에 나타날 때
        animator.Play("Idle");
    }

    public override void OnPrepareStop()
    {
        // 릴 정지 직전
        landEffect.Play();
    }

    public override void OnStopEffect()
    {
        // 릴 정지 시 (착지 효과)
        animator.Play("Land");
        SendContentEvent("OnSymbolLanded", reelIndex);
    }

    public override void OnWin()
    {
        // 당첨 조합에 포함될 때
        animator.Play("Win");
    }

    public override void OnExit()
    {
        // 심볼이 화면에서 사라질 때
        landEffect.Stop();
    }
}
```

## 심볼 라이프사이클

```
OnEntry() - 심볼 생성/화면 진입
    ↓
[스핀 중...]
    ↓
OnPrepareStop() - 릴 정지 직전
    ↓
OnStopEffect() - 릴 완전 정지 (착지)
    ↓
[당첨 계산]
    ↓
OnWin() - 당첨 심볼인 경우
    ↓
[다음 스핀]
    ↓
OnExit() - 화면에서 제거
```

## 이벤트 발송 패턴

```csharp
public override void OnStopEffect()
{
    // 특정 심볼 착지 시 이벤트 발송
    if (IsSpecialSymbol())
    {
        SendContentEvent("OnSpecialLanded", new {
            reel = reelIndex,
            row = rowIndex,
            value = symbolValue
        });
    }
}
```

## 게임별 예시

### Scatter 심볼
```csharp
public class SCXScatterBehaviour : SymbolBehaviour
{
    public override void OnStopEffect()
    {
        // Scatter 착지 시 사운드 + 이벤트
        AudioManager.Play("scatter_land");
        SendContentEvent("OnScatterLanded", reelIndex);
    }

    public override void OnWin()
    {
        // 프리스핀 트리거 시 특수 연출
        animator.Play("Trigger");
    }
}
```

### Wild 심볼
```csharp
public class WildBehaviour : SymbolBehaviour
{
    public override void OnWin()
    {
        // Wild가 당첨에 기여할 때
        animator.Play("WildWin");
        ShowMultiplierEffect(multiplier);
    }
}
```

### Lightning/Value 심볼
```csharp
public class LightningBehaviour : SymbolBehaviour
{
    [SerializeField] private TextMeshProUGUI valueText;

    public override void OnEntry()
    {
        // 값 표시
        var value = GetSymbolCustomData<long>("value");
        valueText.text = value.ToString("N0");
    }

    public override void OnStopEffect()
    {
        SendContentEvent("OnLightningLanded", new {
            position = symbolPosition,
            value = GetSymbolCustomData<long>("value")
        });
    }
}
```

## 심볼 데이터 접근

```csharp
// 심볼 커스텀 데이터 읽기
var customData = GetSymbolCustomData<T>("key");

// 현재 위치 정보
int reel = reelIndex;
int row = rowIndex;
Vector3 worldPos = symbol.transform.position;

// 심볼 정보
SymbolInfo info = symbol.symbolInfo;
bool isWild = info.IsWild();
bool isScatter = info.IsScatter();
```

## 파일 위치

```
Contents Group 1/{GameName}/Scripts/Behaviours/
├── {Game}ScatterBehaviour.cs
├── {Game}WildBehaviour.cs
├── {Game}CashSymbolBehavior.cs
└── {Game}LifeSymbolBehavior.cs
```

## 프리팹 생성 워크플로우

### 1단계: 스크립트 작성

```
Assets/Contents/Contents Group 1/{GameName}/Scripts/Behaviours/{Game}XXXSymbolBehavior.cs
```

### 2단계: 빈 프리팹 생성

```
Assets/Contents/Contents Group 1/{GameName}/Prefabs/Symbol Behaviors/
```

위 폴더에서:

1. **우클릭 → Create → Prefab** (빈 프리팹 생성)
2. 프리팹 이름을 스크립트와 동일하게 설정 (예: `BBDLifeSymbolBehavior`)

### 3단계: 컴포넌트 추가

1. 생성한 빈 프리팹 선택
2. **Inspector → Add Component**
3. 작성한 스크립트 검색 및 추가 (예: `BBDLifeSymbolBehavior`)

### 4단계: CachedObject 설정 (필요시)

프리팹에 자식 오브젝트 추가:

```
{Game}XXXSymbolBehavior (Prefab Root)
├── StopObject (index 0) - GetCachedObject(0)
├── WinObject (index 1) - GetCachedObject(1)
└── ... 추가 오브젝트들
```

> **참고**: `GetCachedObject(index)` 순서는 Hierarchy에서 자식 오브젝트 순서와 동일합니다.

## 프리팹 구조 예시

```
Prefabs/Symbol Behaviors/BBDCashSymbolBehavior.prefab
├── BBDCashSymbolBehavior (스크립트)
├── CashStopObject (index 0)
├── CashWinObject (index 1)
├── InfinityWinObject (index 2)
└── CreditUpObject (index 3)
```

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `feature-controller` | 피처 연출 |
| `spin-flow` | 스핀 흐름 |
