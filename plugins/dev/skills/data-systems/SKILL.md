---
description: "Game data persistence systems. ContentsStore, GameProgress, CustomData patterns and lifecycle."
user-invocable: false
globs:
  - "**/slot_game/*.interface.ts"
---

# ContentsStore / GameProgress / CustomData 시스템

343개 게임 분석에서 발견된 핵심 데이터 저장 패턴입니다.

## 1. ContentsStore - 세션 간 영속 데이터

**정의**: 게임 세션 간에 유지되는 키-값 저장소입니다. Party Bonus, 커뮤니티 게임 등에서 사용됩니다.

```typescript
// games/src/types/interface.ts

// Raw 데이터 구조 (DB 저장용)
export interface IContentsStoreRaw {
  [key: string]: number | { [key: string]: any };
}

// ContentsStore 인터페이스
export interface IContentsStore {
  setNumberValue(key: string, value: number): void;
  increaseNumberValue(key: string, value: number): void;
  getNumberValue(key: string): number;
  setObjectValue(key: string, value: { [key: string]: any }): void;
  getObjectValue(key: string): { [key: string]: any };
  getChangeResult(): IContentsStoreChangeResult;
}

// 변경 추적용 인터페이스
export interface IContentsStoreChange {
  key: string;
  data_type: TContentsStoreDataType;       // NUMBER | OBJECT
  operation: TContentsStoreOprationType;   // SET | INCREASE
  value: number | { [key: string]: any };
}
```

### ContentsStore 사용 게임

```typescript
// ISlotGameWithContentsStore 인터페이스 구현 필요
export interface ISlotGameWithContentsStore {
  generateInitialContentsStore(gameSession: IGameSessionSlotGame): IContentsStoreRaw;
}

// 게임 정보에서 확인
interface IGameInfo {
  is_contents_store_used?: boolean;   // true면 ContentsStore 사용
}
```

### 사용 예시 (Party Bonus 게임)

```typescript
// 파티 보너스 진행도 저장
contentsStore.setNumberValue('party_progress', 50);
contentsStore.increaseNumberValue('party_tickets', 1);

// 복잡한 데이터 저장
contentsStore.setObjectValue('party_state', {
  current_round: 3,
  participants: ['user1', 'user2'],
  rewards: [1000, 2000]
});
```

---

## 2. GameProgress - 영구 게임 진행 상태

**정의**: 게임 세션과 독립적으로 유지되는 영구 진행 상태입니다. 빙고 보드, 레벨 진행도 등에 사용됩니다.

```typescript
// 기본 인터페이스 (게임별로 확장)
export interface IGameProgress {
  [key: string]: any;
}

// 게임 정보에서 확인
interface IGameInfo {
  has_permanent_state: boolean;   // true면 GameProgress 사용
}

// GameProgress 사용 게임 인터페이스
export interface ISlotGameWithGameProgress {
  generateInitialGameProgress(): IGameProgress;
  updateGameProgress(gameProgress: IGameProgress, customData: any);
}
```

### GameProgress 사용 게임 (Bingo 계열)

```typescript
// Bingo Ahoy (308) - GameProgress 구조
interface IBingoAhoyProgress extends IGameProgress {
  bingo_per_bet: {
    [totalBet: string]: {
      board: number[][];           // 5x5 빙고판
      marked_cells: number[];      // 마킹된 셀
      completed_lines: number[];   // 완성된 라인
    };
  };
  bonus_progress_data: {
    [key: string]: any;
  };
}

// Bingo Inferno Deluxe (233) - GameProgress 구조
interface IBIDProgress extends IGameProgress {
  bingo_progress_info_per_total_bet: {
    [totalBet: number]: {
      bingo_info: IBIDBingoSpot[];
      is_diamond_spin: boolean;
      diamond_spot_index: number;
      left_diamond_spin_count: number;
    };
  };
  default_bingo_info: IBIDBingoSpot[];
}
```

### 베팅별 독립 진행도 패턴

```typescript
// 베팅 금액별로 별도 진행도 관리
function getBingoProgress(gameSession: IGameSession, totalBet: number) {
  const progressKey = totalBet.toString();

  if (!(progressKey in gameSession.custom_data.bingo_per_bet)) {
    // 새 베팅 금액에 대한 초기 진행도 생성
    gameSession.custom_data.bingo_per_bet[progressKey] = {
      board: generateNewBingoBoard(),
      marked_cells: [],
      completed_lines: []
    };
  }

  return gameSession.custom_data.bingo_per_bet[progressKey];
}
```

---

## 3. CustomData - 게임별 커스텀 데이터

**정의**: 각 게임의 고유한 상태를 저장하는 구조입니다. `gameSession.custom_data`에 저장됩니다.

```typescript
// 게임 세션 내 CustomData 위치
export interface IGameSessionSlotGame {
  // ...
  custom_data: any;   // 게임별 CustomData가 여기 저장됨
  // ...
}
```

### CustomData 초기화 패턴

```typescript
// 게임 모듈에서 구현 필수
export interface ISlotGame {
  generateInitialCustomData(rtpId: string, isFreeSpin: boolean): ICustomData;
  getInitialCustomDataForClient(gameSession: IGameSession, extraData: any): any;
}

// Bingo Ahoy 초기화 예시
generateInitialCustomData(rtpId: string, isFreeSpin: boolean): IBingoAhoyCustomData {
  return {
    bingo_per_bet: {},
    bonus_progress_data: {},
  };
}
```

### CustomData 유형별 구조

**A. 단순 상태 저장 (대부분의 게임)**

```typescript
// Buffalo Burst (282)
interface IBuffaloBurstCustomData {
  extra_bet_index: number;
}

// Supercharge X-treme (343)
interface ISCXCustomData {
  value_item_states_per_bet: {
    [betCredit: number]: ISCXBetData;
  };
  lightning_hit_count: number[];
}
```

**B. 영구 진행도 저장 (Bingo, 퀘스트 게임)**

```typescript
// Bingo Ahoy (308) - Permanent State
interface IBingoAhoyCustomData {
  bingo_per_bet: {                           // 베팅별 빙고판
    [totalBet: string]: IBingoBoard;
  };
  bonus_progress_data: {                     // 보너스 진행도
    [key: string]: IBonusProgressionInfo;
  };
}

// 세션 간 유지됨 (has_permanent_state: true)
```

**C. Link 보너스 상태 (Hold & Spin 게임)**

```typescript
// Infinite Fireball Link (279)
interface IIFLCustomData {
  link_bonus_state: {
    collected_symbols: ILinkSymbol[];
    remaining_spins: number;
    current_grid: number[][];
    unlocked_rows: number;
  } | null;
  spot_split_data: ISpotSplitData[];
}
```

---

## 4. 세 가지 시스템 비교

| 특성 | ContentsStore | GameProgress | CustomData |
|------|--------------|--------------|------------|
| **저장 위치** | 별도 DB 테이블 | gameSession 내부 | gameSession.custom_data |
| **영속성** | 세션 간 유지 | 세션 간 유지 | 세션 종료 시 소멸 |
| **용도** | Party/Community | Bingo/Quest | 일반 게임 상태 |
| **변경 추적** | O (change_list) | X | X |
| **게임 설정** | is_contents_store_used | has_permanent_state | - |
| **인터페이스** | ISlotGameWithContentsStore | ISlotGameWithGameProgress | ISlotGame |

---

## 5. 데이터 흐름

```
┌─────────────────────────────────────────────────────────────────┐
│                      클라이언트 요청                             │
│  POST /v3/slot/spin                                             │
│  { contents: { CustomData308: {...} } }                        │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ LOGIC 서버                                                       │
│                                                                  │
│  1. gameSession 로드 (DB에서)                                    │
│     └─ custom_data, game_progress 포함                          │
│                                                                  │
│  2. ContentsStore 로드 (필요시)                                  │
│     └─ initial_store에서 IContentsStore 생성                    │
│                                                                  │
│  3. Games 엔진 호출                                              │
│     └─ getGameResult(..., contentsStore)                        │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ Games 엔진                                                       │
│                                                                  │
│  1. CustomData 처리                                              │
│     └─ gameSession.custom_data 읽기/수정                        │
│                                                                  │
│  2. GameProgress 업데이트 (해당 게임만)                          │
│     └─ updateGameProgress(game_progress, customData)            │
│                                                                  │
│  3. ContentsStore 변경 (해당 게임만)                             │
│     └─ contentsStore.setNumberValue('key', value)               │
│                                                                  │
│  4. 결과 반환                                                    │
│     └─ { result, customData, contents_store_change_list }       │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ LOGIC 서버 - 응답 처리                                           │
│                                                                  │
│  1. gameSession 저장 (custom_data, game_progress 포함)          │
│  2. ContentsStore 변경사항 적용                                  │
│  3. 클라이언트에 응답 반환                                       │
│     └─ { customData: CustomData308, ... }                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. 구현 체크리스트

**새 게임에 Permanent State 추가 시:**

1. `IGameInfo.has_permanent_state = true` 설정
2. `ISlotGameWithGameProgress` 인터페이스 구현
3. `generateInitialGameProgress()` 구현
4. `updateGameProgress()` 구현
5. CustomData 인터페이스 정의 (`{game_name}.interface.ts`)

**ContentsStore 사용 시:**

1. `IGameInfo.is_contents_store_used = true` 설정
2. `ISlotGameWithContentsStore` 인터페이스 구현
3. `generateInitialContentsStore()` 구현
4. `getGameResult()`에서 contentsStore 파라미터 활용

---

## 게임별 CustomData 구조

### Supercharge X-treme (343)

```typescript
interface ISCXCustomData {
  value_item_states_per_bet: {
    [betCredit: number]: ISCXBetData;
  };
  lightning_hit_count: number[];
  // Value Item 상태 관리 (최대 14개)
}

// 서버 응답 예시
{
  "CustomData343": {
    "valueList": [
      { "value": 100, "jackpotIndex": 0 },
      { "value": 200, "jackpotIndex": 1 },
      { "value": 500, "jackpotIndex": -1 }
    ],
    "lightningHitCount": [0, 2, 0, 1, 0, 0, 0, 0]
  }
}
```

### Buffalo Burst (282)

```typescript
interface IBuffaloBurstCustomData {
  extra_bet_index: number;  // 추가베팅 인덱스
}

// 서버 응답 예시
{
  "CustomData282": {
    "extra_bet_index": 0
  }
}
```

### Bingo Ahoy (308) - Permanent State

```typescript
interface IBAHCustomData {
  bingo_per_bet: {
    [totalBet: string]: IBingoInfo;
  };
  bonus_progress_data: {
    [key: string]: IBonusProgressionInfo;
  };
  default_bingo_info: IBingoInfo;
  bingo_progress_info_per_total_bet: {
    [totalBet: number]: IBingoProgressionInfo;
  };
  // Bingo 진행도는 세션 간 유지됨
}

// 서버 응답 예시
{
  "CustomData308": {
    "bingoPerBet": {
      "5000": {
        "completedLines": [0, 4],
        "markedCells": [1, 5, 12, 18, 24]
      }
    },
    "bonusProgressData": {...}
  }
}
```

---

## 클라이언트-서버 데이터 매핑

### 요청/응답 매핑

```
클라이언트 요청                    서버 처리                      Blackboard 저장
────────────────────────────────────────────────────────────────────────────
game_id: 343              →  getGameModule(343)           →  (내부 처리)
bet_credit: 1000          →  차감 후 게임 로직 실행        →  ./betCredit
contents.CustomData343    →  contentsParam으로 전달        →  ./spin/request/customData

서버 응답                                                   Blackboard 저장
────────────────────────────────────────────────────────────────────────────
contents.result.reelOutputList                             ./spin/response/result/reelOutputList
contents.result.earnCredit                                 ./spin/response/result/earnCredit
contents.customData.CustomData343                          ./spin/response/customData/CustomData343
contents.bonusList[0].bonus_id                             ./spin/response/bonusList/0/bonusId
contents.bonusList[0].result.added_spin_count              ./bonus/response/initialSpinCount
user_sync_info.credit                                      /me/credit
user_sync_info.level                                       /me/level
```

### 보너스 응답 구조

```typescript
// 서버 보너스 응답
{
  bonus_id: 34300,           // 보너스 ID
  earn_credit: 0,            // 즉시 획득액 (FREE_SPIN은 0)
  type: 2,                   // BonusResultType.FREE_SPIN
  result: {
    is_jackpot: false,
    added_spin_count: 8      // 프리스핀 횟수
  },
  claim_type: 0,             // 0: 자동, 1: 선택
  uid: "freespin_343_001"    // 고유 ID (클레임용)
}

// 클라이언트 Blackboard 매핑
./spin/response/bonusList/0/bonusId = 34300
./spin/response/bonusList/0/type = 2
./bonus/bonusId = 34300
./bonus/response/initialSpinCount = 8
./bonus/spinCount = 8  // 클라이언트에서 복사
```
