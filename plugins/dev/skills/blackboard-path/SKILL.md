---
description: "Blackboard data path guide. Server response to client Blackboard mapping."
user-invocable: false
globs:
  - "**/*Controller*.cs"
  - "**/*Module*.cs"
  - "**/FSM/**/*.asset"
---

# Blackboard 경로 가이드

Blackboard 데이터 경로와 접근 방법을 안내합니다.

## 경로 체계

```
./                              # ContentBlackboard 루트
├── baseSlotMachine            # SlotMachine GameObject
├── betCredit                  # 현재 베팅액
├── totalBetCredit             # 총 베팅액
├── earnCredit                 # 획득 크레딧
├── autoSpin                   # 오토스핀 플래그
│
├── customData/                # 게임별 서버 데이터
│   ├── slotDataList/0/column  # 릴 수
│   ├── slotDataList/0/row     # 보이는 심볼 수
│   ├── betList                # 가능한 베팅 목록
│   └── valueItemsPerBet       # 베팅별 팟 값
│
├── spin/                      # 현재 스핀 데이터
│   ├── uid                    # 스핀 고유 ID
│   ├── earnCredit             # 스핀 획득금
│   ├── response/
│   │   ├── result/reelOutputList  # [12, 45, 23, 67, 34, 89]
│   │   ├── customData         # 게임별 응답
│   │   ├── bonusList          # [BonusResult34300, ...]
│   │   └── earnCredit         # 서버 당첨 크레딧
│   ├── deck                   # Deck 객체
│   └── winList                # 당첨 라인 목록
│
├── bonus/                     # 보너스/피처 데이터
│   ├── bonusId                # 34300=FreeGame, 34302=Lightning
│   ├── spinCount              # 남은 스핀 수
│   ├── totalSpinCount         # 총 스핀 수
│   ├── earnCredit             # 보너스 획득금
│   └── response/
│       ├── uid                # 보너스 고유 ID
│       ├── initialSpinCount   # 부여된 프리스핀
│       └── freeGameValueItems # 프리스핀 팟 값
│
└── game/                      # 정적 게임 설정
    ├── baseWager              # 최소 베팅
    ├── paytables              # 페이라인 배당
    └── jackpotInfo/infoList   # 잭팟 정의

/me/                           # MainBlackboard (유저 정보)
├── credit                     # 잔액
├── level                      # 레벨
└── profile                    # 프로필
```

## 코드에서 접근

### 읽기
```csharp
// ContentBlackboard에서 읽기
var bb = ContentBlackboard.Get();
var earnCredit = bb.GetValue<long>("./spin/earnCredit");
var reelOutputList = bb.GetValue<List<int>>("./spin/response/result/reelOutputList");

// 중첩 객체 읽기
var bonus = bb.GetValue<BonusResult34300>("./bonus/response");
var spinCount = bonus.initialSpinCount;

// MainBlackboard에서 읽기
var credit = MainBlackboard.Get().GetValue<long>("/me/credit");
```

### 쓰기
```csharp
// ContentBlackboard에 쓰기
var bb = ContentBlackboard.Get();
bb.SetValue("./spin/earnCredit", 50000L);
bb.SetValue("./customData/myValue", myData);
```

### FSM/BT Action에서
```csharp
// BBParameter 사용
public BBParameter<long> earnCredit;
public BBParameter<string> path = new BBParameter<string> { value = "./spin/earnCredit" };

protected override void OnExecute()
{
    var value = earnCredit.value;  // 직접 접근
    // 또는
    var bb = ContentBlackboard.Get();
    var value = bb.GetValue<long>(path.value);
}
```

## 자주 사용하는 경로

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

## BlackboardUtils 헬퍼

```csharp
// 변수 가져오기/생성
var variable = BlackboardUtils.GetOrCreateVariable<GameObject>("./baseSlotMachine");
var slotMachine = variable.value.GetComponent<SlotMachine>();

// 값 찾기
var value = BlackboardUtils.FindValue<long>("./spin/earnCredit");

// 리스트 접근
var list = BlackboardUtils.FindValue<List<int>>("./customData/betList");
```
