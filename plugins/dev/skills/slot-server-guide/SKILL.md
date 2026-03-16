---
description: "Slot server implementation guide. Game logic patterns, coding rules, bonus creation, ESLint conventions."
user-invocable: false
globs:
  - "**/slot_game/*.ts"
  - "**/slot_game/*.value.ts"
  - "**/slot_game/*.interface.ts"
---

# Slot Server Guide

When implementing or modifying slot game server logic, follow these patterns and rules.

## File Structure (3 files per game)

```
games/src/slot_game/
├── {game_code}.ts           # Main game logic
├── {game_code}.value.ts     # Reel strips, pay tables, probability constants
└── {game_code}.interface.ts # TypeScript types (CustomData, bonus responses)
```

## Game Class Structure

```typescript
export class GameXXX extends SlotGame {
  async spin(request): Promise<SpinResponse> { ... }
  async claim(request): Promise<ClaimResponse> { ... }
  calcWin(grid, betCredit): WinResult { ... }
  checkBonusTrigger(grid): BonusList { ... }
}
```

## File Locations

| 타입 | 경로 |
|------|------|
| 서버 게임 로직 | `games/src/slot_game/{game_name}.ts` |
| 서버 인터페이스 | `games/src/slot_game/{game_name}.interface.ts` |
| 서버 값 정의 | `games/src/slot_game/{game_name}.value.ts` |
| 클라이언트 Schema | `client/Assets/Contents/Contents Group 1/{GameName}/Data/Schema.json` |

## Core Principles

- **Server-Authoritative RNG**: All random numbers generated on server
- **Programmed Win/Loss**: RTP managed via `getSlotGameResultWithProgrammedLoss()`

### Bonus ID Convention

```
기본 보너스: gameId × 100 + 0~9
├─ +00: Free Spin (주요 보너스)
├─ +01: Jackpot / 보조 보너스
├─ +02: Retrigger / 라이트닝
├─ +03: 멀티플라이어 / 특수
├─ +04: Pre-Event / Fake
└─ +05~09: 게임별 추가 피처

티켓형 보너스: gameId × 100 + 50~99
├─ +50: Buy a Bonus (기본)
├─ +51: Buy a Bonus Lv2
└─ +52: Buy a Bonus Lv3
```

**예시** (게임 343):
- 34300: Free Game, 34301: Pick Bonus, 34302: Lightning
- 34350: Buy A Bonus

## API Response Structure

```typescript
{
  result: {
    reelOutputList: number[][],
    earnCredit: number,
  },
  bonusList: [{
    bonusId: number,
    bonusData: any,
  }],
  customData: {
    CustomDataXXX: { ... }
  },
  user_sync_info: {
    credit: number,
  }
}
```

---

## Data Reference Rules

1. **reel_sequence_list 사용**: 릴 스트립을 직접 생성하지 말고, 반드시 `.value.ts` 파일에 정의된 `reel_sequence_list`를 사용할 것

```typescript
// ❌ 잘못된 방법 - 직접 릴 스트립 생성
const reelStrip = [1, 2, 3, 4, 5, ...];

// ✅ 올바른 방법 - reel_sequence_list 활용
const reelStrip = VALUE.bonus_info[rtpID].reel_sequence_list[reelIndex];
```

2. **PayTable 사용**: 당첨 계산 시 하드코딩하지 말고, 반드시 `PayTable`을 참조할 것

```typescript
// ❌ 잘못된 방법 - 하드코딩된 배당
const payout = symbol === 5 ? 100 : 50;

// ✅ 올바른 방법 - PayTable 참조
const payout = VALUE.bonus_info[rtpID].PayTable[symbol][matchCount];
```

3. **배열 기반 Weight Total 정의**: 가중치 총합은 단일 값이 아닌 배열로 정의할 것 (RTP별, 릴셋별 등)

```typescript
// ❌ 잘못된 방법 - 단일 값
const superBonusJackpotWeightsTotal = 1000;

// ✅ 올바른 방법 - 배열로 정의
const superBonusJackpotWeightsTotal = [1000, 1000, 1000, 1000, 1000]; // RTP별
const mysterySymbolsWeightsTotals = [0, 50, 100, 150, 200]; // 릴셋별
```

---

## Mystery Symbol 처리 주의사항

- `getRandomIndexWithWeight`가 total이 0일 때 마지막 인덱스를 반환함
- 유효하지 않은 심볼 (SCATTER, GOLDEN_BARBARIAN 등)이 선택될 수 있음
- 반드시 fallback 로직 추가 필요:

```typescript
let symbol = getRandomIndexWithWeight(weights, totals);
// 유효하지 않은 심볼일 경우 fallback
if (symbol === SYMBOLS.SCATTER || symbol === SYMBOLS.GOLDEN_BARBARIAN) {
  symbol = SYMBOLS.H1;
}
```

---

## INSTANT_BONUS 디버그 타입 구현 (티켓 보너스 시뮬레이션)

티켓으로 진입하는 보너스 (Super Bonus 등)를 시뮬레이션할 때 `SLOT_DEBUG.INSTANT_BONUS` (400)를 처리해야 합니다.

### Step 1: modifyDebugParam에서 플래그 설정

```typescript
case SLOT_DEBUG.INSTANT_BONUS:
  const simLevel = (Game as any).simulationSuperBonusLevel || 1;
  newDebugParam = {
    force_instant_bonus: true,
    super_bonus_level: simLevel,
  };
  break;
```

### Step 2: getGameResult에서 free_spin 세션 설정 (bacon_trio 패턴)

```typescript
if (modifiedDebugParam?.force_instant_bonus && !isFreeSpin) {
  const superBonusWeights = superBonusInfo[rtp];
  const spinCount = superBonusWeights.super_free_spin_count;

  gameSession.free_spin = {
    type: SLOT.FREE_SPIN.NORMAL,
    bet: baseBet + extraBet,
    count: spinCount,
    total_count: spinCount,
    initial_count: spinCount,
    spun_count: 0,
    multiplier: 1,
    sticky_wild: [],
    extra_data: { /* 보너스별 커스텀 데이터 */ },
  };

  return {
    result: { earn_credit: 0, reel_output_list: [], ... },
    bonusResultList: [],
    symbolCountInfo: {},
    extraData: { ... },
  };
}
```

### Step 3: 시뮬레이션 파일에서 레벨 전달

```typescript
const game = new Game();
(Game as any).simulationSuperBonusLevel = level;
const simulator = new Simulator(VALUE.game_id, baseBet, game);
```

---

## Bonus Creation Pattern

### _GenerateBonusResult 함수

```typescript
function _GenerateBonusResult(
  bonusId: number,      // 보너스 ID (gameId × 100 + sequence)
  bonusType: number,    // BonusResultType 열거형
  earnCredit: number,   // 즉시 지급 크레딧 (0이면 별도 처리)
  claimType: number,    // types.CLAIM_TYPE (ALWAYS, OPTIONAL, NONE)
  response: any         // 보너스별 응답 데이터
): TBonusResult
```

### BonusResultType 열거형

```typescript
export enum BonusResultType {
  NORMAL = 1,                    // 일반 보너스
  FREE_SPIN = 2,                 // 프리스핀
  SELECTABLE_FREE_SPIN = 3,      // 선택형 프리스핀
  RESPIN_REGARDED_AS_FREE_SPIN = 4,  // 리스핀 (프리스핀 취급)
  PARTY_GAME = 5,                // 파티 게임
}
```

### CLAIM_TYPE 상수

```typescript
export const CLAIM_TYPE = {
  ALWAYS: 1,    // 자동 클레임 (즉시 적용)
  OPTIONAL: 2,  // 선택적 클레임 (유저 선택)
  NONE: 3,      // 클레임 불필요 (정보/연출만)
};
```

### 새 보너스 추가 패턴

#### 1. bonusType 상수 정의 (game.ts)

```typescript
export const bonusType = {
  EXISTING_BONUS: { BONUS_ID: gameId * 100 + 0 },
  NEW_BONUS: { BONUS_ID: gameId * 100 + sequence },
} as const;
```

#### 2. 응답 인터페이스 정의 (game.interface.ts)

```typescript
export interface INewBonusResponse {
  custom_field: number;
}
```

#### 3. 보너스 생성 로직 (game.ts의 게임 결과 함수 내)

```typescript
// 조건 판정 후 보너스 생성
if (triggerCondition) {
  bonusResultList.push(_GenerateBonusResult(
    bonusType.NEW_BONUS.BONUS_ID,
    BonusResultType.NORMAL,
    0,                        // earnCredit (0이면 별도 처리)
    types.CLAIM_TYPE.NONE,    // 연출만 필요한 경우
    { custom_field: value }   // 보너스별 응답 데이터
  ));
}
```

#### 4. games.ts 업데이트 (선택적)

```typescript
// server/src/games.ts에서 해당 게임의 bonus_list에 추가
bonus_list: [
  { bonus_name: 'EXISTING_BONUS', bonus_id: xxxxx },
  { bonus_name: 'NEW_BONUS', bonus_id: xxxxx },  // 추가
],
```

### 주의사항

- **베이스 게임 vs 프리게임**: 보너스 트리거 조건이 게임 모드에 따라 다를 수 있음
- **연출 전용 보너스**: `earnCredit: 0`, `claimType: NONE`으로 설정
- **보너스 ID 규칙**: `gameId × 100 + sequence`

---

## 스키마/인터페이스 작성 규칙

### 배열 타입 문법

> **중요**: 클라이언트에서 스키마 파싱 시 `Array<T>` 문법이 인식되지 않습니다.

| 사용 금지 | 사용 권장 |
|-----------|-----------|
| `Array<{ reel: number }>` | `{ reel: number }[]` |
| `Array<string>` | `string[]` |
| `Array<T>` | `T[]` |

**예시:**
```typescript
// ❌ 잘못된 예
barbarian_positions?: Array<{ reel: number; row: number; symbol: number }>;

// ✅ 올바른 예
barbarian_positions?: { reel: number; row: number; symbol: number }[];
```

**적용 파일**: 모든 `.interface.ts` 파일의 타입 정의

---

## TypeScript/ESLint 코딩 규약

### 1. Import 규칙 (no-duplicate-imports)

같은 모듈에서 여러 번 import하지 마세요. 한 줄로 통합하세요.

| 사용 금지 | 사용 권장 |
|-----------|-----------|
| `import { VALUE } from './game';`<br>`import Game from './game';` | `import Game, { VALUE } from './game';` |

### 2. 파일 끝 줄바꿈 (eol-last)

모든 파일은 빈 줄로 끝나야 합니다.

```typescript
// ❌ 잘못된 예 (파일 끝에 줄바꿈 없음)
simulator.runParallel();⏎(EOF)

// ✅ 올바른 예 (파일 끝에 빈 줄)
simulator.runParallel();
⏎(EOF)
```

**이유**: POSIX 표준 준수, git diff 시 불필요한 변경 방지

### 3. 객체 키 따옴표 (quote-props)

숫자 키나 유효한 식별자에는 따옴표를 사용하지 마세요.

| 사용 금지 | 사용 권장 |
|-----------|-----------|
| `{ "1": value, "2": value }` | `{ 1: value, 2: value }` |

**예외**: 특수 문자가 포함된 키는 따옴표 필요 (예: `{ "4+": 0 }`)

### 4. Switch문 Default Case (default-case)

모든 switch문에는 default case를 포함하세요.

```typescript
// ❌ 잘못된 예
switch (type) {
  case "cash": return cashValue;
  case "jackpot": return jackpotValue;
}

// ✅ 올바른 예
switch (type) {
  case "cash": return cashValue;
  case "jackpot": return jackpotValue;
  default: break;  // 또는 throw new Error(`Unknown type: ${type}`)
}
```

**이유**: 예상치 못한 값에 대한 방어적 코딩

### 5. 들여쓰기 (indent)

2 spaces를 사용하세요. 탭이나 4 spaces 사용 금지.

```typescript
// ❌ 잘못된 예 (4 spaces 또는 탭)
if (condition) {
    doSomething();
        nestedCall();
}

// ✅ 올바른 예 (2 spaces)
if (condition) {
  doSomething();
  nestedCall();
}
```

---

## 코드 작성 전 체크리스트

```markdown
[ ] reel_sequence_list 사용 (하드코딩 X)
[ ] PayTable 참조 (하드코딩 X)
[ ] Weight 총합 배열로 정의 (RTP별)
[ ] Array<T> 대신 T[] 사용
[ ] Import 문 중복 없음
[ ] 파일 끝 빈 줄
[ ] Switch문 default case
[ ] 2-space indent
[ ] Mystery Symbol fallback 로직
```

---

## 주의사항

1. **RNG 보안**: 난수 생성은 반드시 서버에서만 수행
2. **RTP 모니터링**: 실시간 RTP가 목표 범위를 벗어나면 알림
3. **잭팟 무결성**: 잭팟 풀 변경은 트랜잭션으로 처리
4. **성능 최적화**: 무거운 계산은 캐싱 활용
5. **버전 관리**: 게임 설정 변경 시 버전 기록

---

## 디버깅 및 로깅

```typescript
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.json(),
    transports: [
        new winston.transports.Console(),
        new winston.transports.File({ filename: 'games.log' })
    ]
});

logger.info('Spin result', { gameId, earnCredit, bonusCount: bonusList.length });
logger.debug('RNG values', { reelOutputList });
```

```bash
DEBUG=* npm start:debug          # 전체 디버그
DEBUG=games:slot npm start:debug # 슬롯만 디버그
```

## Schema Sync Reminder

When modifying `.interface.ts`, remind the user about client Schema.json sync.
