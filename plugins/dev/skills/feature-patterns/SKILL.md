---
description: "Server-side feature implementation patterns. Hold&Spin, Cascade, Nudge, Mystery, Wheel, and more."
user-invocable: false
globs:
  - "**/slot_game/*.ts"
  - "**/slot_game/*.bonus.ts"
---

# 피처별 서버 구현 패턴 (343개 게임 분석 결과)

343개 슬롯 게임 분석에서 발견된 서버 측 핵심 패턴들입니다.

## 1. Hold & Spin / Link 메카닉

**대표 게임**: Infinite Fireball Link (279), Buffalo Link (146), Flamin Hot Link (253)

### 서버 로직 구조

```typescript
// infinite_fireball_link.ts 패턴
interface ILinkBonusState {
  collectedSymbols: ILinkSymbol[];     // 수집된 심볼
  remainingSpins: number;              // 남은 스핀 (기본 3)
  currentGrid: number[][];             // 현재 그리드 상태
  unlockedRows: number;                // 해금된 행 수
  totalValue: number;                  // 누적 가치
}

// Row Unlock 계산
function calculateUnlockedRows(collectedCount: number): number {
  if (collectedCount < 10) return 0;      // 기본 4행
  if (collectedCount < 15) return 1;      // 5행
  if (collectedCount < 20) return 2;      // 6행
  if (collectedCount < 25) return 3;      // 7행
  return 4;                               // 8행 (최대)
}

// 스핀 리셋 로직
function processLinkSpin(state: ILinkBonusState, newSymbols: ILinkSymbol[]): void {
  if (newSymbols.length > 0) {
    state.remainingSpins = 3;  // 심볼 수집 시 3스핀으로 리셋
    state.collectedSymbols.push(...newSymbols);
  } else {
    state.remainingSpins--;
  }
}
```

### CustomData 구조

```typescript
// CustomData279 (Infinite Fireball Link)
interface IIFLCustomData {
  link_bonus_state: ILinkBonusState | null;
  progressive_multipliers: number[];   // [1, 2, 3, 5, 10]
  spot_split_data: ISpotSplitData[];   // SpotSplit 릴 정보
}
```

### 보너스 ID 패턴

```
27900: FREE_SPIN
27901: LINK_BONUS (Hold & Spin 진입)
27902: LINK_RESPIN (링크 리스핀)
27903: JACKPOT_WIN
27904: ROW_UNLOCK
27950: INSTANT_LINK (티켓형)
```

---

## 2. Cascade/Tumble 메카닉

**대표 게임**: Oink Overflow (325), Buzz Pop (187), Fruity Luck (142)

### 서버 로직 구조

```typescript
// oink_overflow.ts 패턴
interface ICascadeResult {
  initialGrid: number[][];           // 초기 심볼 배치
  cascadeSteps: ICascadeStep[];      // 각 캐스케이드 단계
  totalEarnCredit: number;           // 총 획득액
  finalMultiplier: number;           // 최종 배수
}

interface ICascadeStep {
  matchedSymbols: IMatchedSymbol[];  // 매칭된 심볼
  earnCredit: number;                // 이 단계 획득액
  droppedSymbols: number[][];        // 드롭된 새 심볼
  reelMultiplier: number;            // 릴 배수
}

// Grade 기반 당첨 계산
function calculateGradeWin(matchedCount: number, payTable: number[]): number {
  if (matchedCount < 8) return 0;              // Grade 0: 무당첨
  const gradeIndex = Math.min(matchedCount - 7, payTable.length - 1);
  return payTable[gradeIndex];
}

// 등급 체계
// Grade 0: 0-7개 → 배당 없음
// Grade 1: 8-9개 → payTable[1]
// Grade 2: 10-11개 → payTable[2]
// Grade 3: 12-13개 → payTable[3]
// Grade 4: 14-15개 → payTable[4]
// Grade 5: 16개+ → payTable[5]
```

### Reel Multiplier 시스템

```typescript
// 릴별 배수 (2x ~ 100x)
const REEL_MULTIPLIERS = [2, 3, 5, 10, 25, 50, 100];

function applyReelMultiplier(baseWin: number, multiplierIndex: number): number {
  return baseWin * REEL_MULTIPLIERS[multiplierIndex];
}
```

### 보너스 ID 패턴

```
32500: FREE_SPIN
32501: CASCADE_BONUS (캐스케이드 보너스)
32502: REEL_MULTIPLIER (릴 배수 적용)
32503: MEGA_WIN
32504: GRAND_WIN
32550: INSTANT_CASCADE (티켓형)
```

---

## 3. Nudge 메카닉

**대표 게임**: Crystal Nudge (337), Nudge Strike (216)

### 서버 로직 구조

```typescript
// crystal_nudge.ts 패턴
interface INudgeResult {
  originalGrid: number[][];          // 원본 그리드
  nudgeOperations: INudgeOp[];       // 넛지 연산 목록
  finalGrid: number[][];             // 최종 그리드
}

interface INudgeOp {
  reelIndex: number;                 // 릴 인덱스
  direction: NudgeDirection;         // 방향 (UP/DOWN)
  symbolId: number;                  // 이동할 심볼
  startRow: number;                  // 시작 행
  endRow: number;                    // 종료 행
}

enum NudgeDirection {
  WILD_DOWN = 0,   // Wild가 아래로 (-1)
  WILD_UP = 1      // Wild가 위로 (+1)
}

// Nudge 적용 로직
function applyNudge(grid: number[][], nudgeOps: INudgeOp[]): number[][] {
  const result = JSON.parse(JSON.stringify(grid));

  for (const op of nudgeOps) {
    const direction = op.direction === NudgeDirection.WILD_DOWN ? -1 : 1;
    // Wild 심볼을 direction 방향으로 이동
    shiftSymbol(result, op.reelIndex, op.startRow, direction);
  }

  return result;
}
```

### CustomData 구조

```typescript
// CustomData337 (Crystal Nudge)
interface ICNCustomData {
  nudge_results: INudgeResult[];     // 넛지 결과 배열
  wild_positions: number[][];        // Wild 위치
  total_nudge_count: number;         // 총 넛지 횟수
}
```

---

## 4. Mystery Symbol 시스템

**대표 게임**: Grand Jazzy Sevens (49), Midas Gold (72)

### 서버 로직 구조

```typescript
// grand_jazzy_sevens.ts 패턴
interface IMysterySymbolResult {
  nowMysterySymbols: number[];       // 현재 스핀 미스터리 심볼
  nextMysterySymbols: number[];      // 다음 스핀 미스터리 심볼
  transformedPositions: number[][];  // 변환된 위치
  targetSymbol: number;              // 변환 대상 심볼
}

// Mystery Symbol 변환 로직
function transformMysterySymbols(
  grid: number[][],
  mysterySymbolId: number,
  targetSymbol: number
): number[][] {
  const result = JSON.parse(JSON.stringify(grid));

  for (let reel = 0; reel < result.length; reel++) {
    for (let row = 0; row < result[reel].length; row++) {
      if (result[reel][row] === mysterySymbolId) {
        result[reel][row] = targetSymbol;
      }
    }
  }

  return result;
}

// ANY SEVEN 페이 로직
function calculateAnySevenPay(symbols: number[], payTable: any): number {
  const sevenSymbols = [7, 77, 777];  // 단일, 더블, 트리플 세븐
  const sevenCount = symbols.filter(s => sevenSymbols.includes(s)).length;

  if (sevenCount >= 3) {
    return payTable.ANY_SEVEN[sevenCount];
  }
  return 0;
}
```

---

## 5. Wheel 보너스 시스템

**대표 게임**: Golden Piggy (92), Charming Wheels (177), Big Footunate Wheel (358)

### 서버 로직 구조

```typescript
// golden_piggy.ts 패턴 (7개 보너스 타입 - 가장 복잡한 게임)
interface IWheelResult {
  wheelType: WheelType;              // 휠 종류
  spinResult: number;                // 휠 정지 인덱스
  prizeTier: string;                 // 당첨 티어
  earnCredit: number;                // 획득 크레딧
  nextBonus?: number;                // 체인 보너스 ID
}

enum WheelType {
  PIGGY_WHEEL = 0,        // 메인 피기 휠
  BONUS_WHEEL = 1,        // 보너스 휠
  JACKPOT_WHEEL = 2,      // 잭팟 휠
}

// 3-way 분기 로직
function processWheelResult(wheelResult: number): IWheelOutcome {
  const wheelConfig = getWheelConfig();
  const segment = wheelConfig.segments[wheelResult];

  switch (segment.type) {
    case 'COIN_PICK':
      return { bonusId: 9201, chainBonus: true };
    case 'FREE_SPIN':
      return { bonusId: 9200, chainBonus: true };
    case 'COIN_PRIZE':
      return { earnCredit: segment.value, chainBonus: false };
  }
}
```

### 복잡한 보너스 체인 구조

```typescript
// Golden Piggy (92) - 7개 보너스 타입
const bonusType = {
  FREE_SPIN: { BONUS_ID: 9200 },
  COIN_PICK: { BONUS_ID: 9201 },
  COIN_PRIZE: { BONUS_ID: 9202 },
  PIGGY_WHEEL: { BONUS_ID: 9203 },
  PROGRESSIVE_JACKPOT: { BONUS_ID: 9204 },
  MYSTERY_BONUS: { BONUS_ID: 9205 },
  MEGA_SYMBOL: { BONUS_ID: 9206 },
} as const;
```

---

## 6. WILDx2 / 계층형 Wild 시스템

**대표 게임**: Dazzling Jewel Streak (212), Rich Hits (시리즈)

### 서버 로직 구조

```typescript
// dazzling_jewel_streak.ts 패턴
interface IHierarchicalWildResult {
  wildPositions: IWildPosition[];    // Wild 위치 및 배수
  totalMultiplier: number;           // 총 배수
  jackpotTier?: string;              // 잭팟 등급 (WILDx2 개수 기반)
}

interface IWildPosition {
  reel: number;
  row: number;
  multiplier: 1 | 2;                 // WILDx1 또는 WILDx2
}

// 잭팟 등급 결정 (WILDx2 개수 기반)
function determineJackpotTier(wildx2Count: number): string | null {
  if (wildx2Count >= 5) return 'GRAND';
  if (wildx2Count >= 4) return 'MAJOR';
  if (wildx2Count >= 3) return 'MINOR';
  if (wildx2Count >= 2) return 'MINI';
  return null;
}

// 당첨 계산 시 배수 적용
function calculateWinWithWildMultiplier(
  baseWin: number,
  matchedWilds: IWildPosition[]
): number {
  let totalMultiplier = 1;
  for (const wild of matchedWilds) {
    totalMultiplier *= wild.multiplier;
  }
  return baseWin * totalMultiplier;
}
```

---

## 7. Pre-Event / Fake Reel 시스템

**대표 게임**: Oink Overflow (325), 대부분의 Hold & Spin 게임

### 서버 로직 구조

```typescript
// Pre-Event 시스템: 실제 결과 전 가짜 결과 표시
interface IPreEventResult {
  fakeReelOutput: number[];          // 가짜 릴 결과 (클라이언트 표시용)
  realReelOutput: number[];          // 실제 릴 결과
  preEventType: PreEventType;        // Pre-Event 종류
  triggerDelay: number;              // 연출 딜레이 (ms)
}

enum PreEventType {
  NEAR_MISS = 0,          // 아깝게 빗나감
  FAKE_BONUS = 1,         // 가짜 보너스 트리거
  ANTICIPATION = 2,       // 기대감 연출
}

// Pre-Event 생성 조건
function shouldGeneratePreEvent(
  realResult: IGameResult,
  rngValue: number
): boolean {
  // 실제 당첨이 아닐 때만 Pre-Event 가능
  if (realResult.earnCredit > 0) return false;

  // 확률적으로 Pre-Event 생성
  return rngValue < PRE_EVENT_PROBABILITY;
}
```

---

## 8. Infinity Reel / 동적 릴 확장

**대표 게임**: Barbarian Destroyer (234), Medusa Unlimited (222)

### 서버 로직 구조

```typescript
// barbarian_destroyer.ts 패턴
interface IInfinityReelResult {
  initialReelCount: number;          // 초기 릴 수
  expansions: IReelExpansion[];      // 확장 기록
  finalReelCount: number;            // 최종 릴 수
  expandedGrid: number[][];          // 확장된 그리드
}

interface IReelExpansion {
  triggerSymbol: number;             // 트리거 심볼
  newReelIndex: number;              // 추가된 릴 인덱스
  newReelStrip: number[];            // 새 릴스트립
}

// 릴 확장 조건
function checkReelExpansion(
  currentGrid: number[][],
  lastReelIndex: number
): boolean {
  // 마지막 릴에 Wild가 있으면 확장
  const lastReel = currentGrid[lastReelIndex];
  return lastReel.some(symbol => symbol === WILD_SYMBOL);
}

// 새 릴 생성
function generateNewReel(reelIndex: number): number[] {
  const reelStrip = getInfinityReelStrip(reelIndex);
  const randomIndex = Math.floor(Math.random() * reelStrip.length);
  return extractReelWindow(reelStrip, randomIndex);
}
```

---

## 9. Multi-tier Progressive Jackpot

**대표 게임**: 60개+ 게임 (Mega Money 시리즈, Link 시리즈)

### 서버 로직 구조

```typescript
// 5-tier Progressive Jackpot 시스템
interface IJackpotPool {
  MINI: number;
  MINOR: number;
  MAJOR: number;
  GRAND: number;
  MEGA: number;
}

interface IJackpotContribution {
  poolId: string;
  contributionRate: number;          // 베팅액 대비 기여율
  seedAmount: number;                // 시드 금액
  capAmount: number;                 // 최대 금액
}

// 잭팟 기여금 계산
function contributeToJackpot(
  betCredit: number,
  jackpotConfig: IJackpotContribution[]
): void {
  for (const config of jackpotConfig) {
    const contribution = betCredit * config.contributionRate;
    addToJackpotPool(config.poolId, contribution, config.capAmount);
  }
}

// 잭팟 당첨 처리
function processJackpotWin(
  tier: string,
  userId: number
): number {
  const poolAmount = getJackpotPool(tier);
  resetJackpotPool(tier);  // 시드로 리셋
  awardJackpot(userId, poolAmount);
  return poolAmount;
}
```

---

## 10. Bingo 진행도 시스템

**대표 게임**: Bingo Ahoy (308), Bingo Inferno (182), Cash Billionaire Bingo (176)

### 서버 로직 구조 (Permanent State)

```typescript
// bingo_ahoy.ts 패턴
interface IBingoProgress {
  bingoPerBet: {
    [totalBet: string]: IBingoBoard;
  };
  bonusProgressData: {
    [key: string]: IBonusProgressionInfo;
  };
}

interface IBingoBoard {
  board: number[][];                 // 5x5 빙고판
  markedCells: number[];             // 마킹된 셀 인덱스
  completedLines: number[];          // 완성된 라인 인덱스
  currentProgress: number;           // 현재 진행도 (0-25)
}

// 베팅 금액별 별도 진행도 관리
function getBingoBoardForBet(progress: IBingoProgress, totalBet: number): IBingoBoard {
  const betKey = totalBet.toString();
  if (!progress.bingoPerBet[betKey]) {
    progress.bingoPerBet[betKey] = createNewBingoBoard();
  }
  return progress.bingoPerBet[betKey];
}

// 빙고 라인 체크
function checkBingoLines(board: IBingoBoard): number[] {
  const newCompletedLines: number[] = [];

  // 가로 5줄
  for (let row = 0; row < 5; row++) {
    if (isLineComplete(board, 'horizontal', row)) {
      newCompletedLines.push(row);
    }
  }

  // 세로 5줄
  for (let col = 0; col < 5; col++) {
    if (isLineComplete(board, 'vertical', col)) {
      newCompletedLines.push(5 + col);
    }
  }

  // 대각선 2줄
  if (isLineComplete(board, 'diagonal', 0)) newCompletedLines.push(10);
  if (isLineComplete(board, 'diagonal', 1)) newCompletedLines.push(11);

  return newCompletedLines;
}
```

---

## 11. SpotSplit Reel 시스템

**대표 게임**: Infinite Fireball Link (279), 일부 Link 게임

### 서버 로직 구조

```typescript
// SpotSplit: 한 릴에 여러 심볼 영역
interface ISpotSplitReel {
  reelIndex: number;
  spots: ISpot[];                    // 분할된 영역들
  totalHeight: number;               // 총 높이
}

interface ISpot {
  startRow: number;
  endRow: number;
  symbolId: number;
  isActive: boolean;
}

// SpotSplit 결과 생성
function generateSpotSplitResult(
  reelStrip: number[],
  splitConfig: ISpotSplitConfig
): ISpotSplitReel {
  const spots: ISpot[] = [];
  let currentRow = 0;

  for (const split of splitConfig.splits) {
    spots.push({
      startRow: currentRow,
      endRow: currentRow + split.height - 1,
      symbolId: getRandomSymbol(reelStrip),
      isActive: Math.random() < split.activeProbability
    });
    currentRow += split.height;
  }

  return { reelIndex: splitConfig.reelIndex, spots, totalHeight: currentRow };
}
```

---

## 게임 복잡도별 서버 구현 패턴

### 단순 게임 (보너스 1-2개)

```typescript
// 예: Great Catsby (12)
// 파일 구조: great_catsby.ts, great_catsby.value.ts
// 보너스: 12000 (Free Spin), 12001 (Jackpot)

function getGameResult(contentsParam: any, betCredit: number): IGameResult {
  const reelOutput = generateRNG();
  const earnCredit = calculateLineWin(reelOutput);
  const bonusList: TBonusResult[] = [];

  if (checkScatterTrigger(reelOutput)) {
    bonusList.push(_GenerateBonusResult(
      1200, BonusResultType.FREE_SPIN, 0, CLAIM_TYPE.ALWAYS,
      { added_spin_count: 10 }
    ));
  }

  return { result: { reelOutput, earnCredit }, bonusList };
}
```

### 중급 게임 (보너스 3-5개)

```typescript
// 예: Buffalo Burst (282)
// 파일 구조: buffalo_burst.ts, buffalo_burst.value.ts, buffalo_burst.bonus.ts
// 보너스: 28200-28203 + 28250 (티켓)

function getGameResult(contentsParam: any, betCredit: number): IGameResult {
  // 1. 기본 스핀 결과
  const baseResult = generateBaseResult();

  // 2. 보너스 체크 (순서 중요)
  if (checkLinkTrigger(baseResult)) {
    return processLinkBonus(baseResult);
  }

  if (checkFreeSpinTrigger(baseResult)) {
    return processFreeSpinTrigger(baseResult);
  }

  // 3. 일반 결과 반환
  return baseResult;
}
```

### 복합 게임 (보너스 6개+)

```typescript
// 예: Golden Piggy (92) - 7개 보너스
// 파일 구조: golden_piggy.ts, golden_piggy.value.ts, golden_piggy.bonus.ts,
//           golden_piggy.interface.ts, golden_piggy.wheel.ts
// 보너스: 9200-9206 + 9250 (티켓)

function getGameResult(contentsParam: any, betCredit: number): IGameResult {
  const state = getGameState(contentsParam);

  // 복합 보너스 체인 처리
  if (state.activeBonus) {
    return processBonusChain(state);
  }

  // 1. 기본 스핀
  const baseResult = generateBaseResult();

  // 2. 휠 보너스 트리거
  if (checkWheelTrigger(baseResult)) {
    const wheelResult = spinWheel();

    // 3-way 분기
    switch (wheelResult.outcome) {
      case 'COIN_PICK':
        return chainToBonus(9201, wheelResult);
      case 'FREE_SPIN':
        return chainToBonus(9200, wheelResult);
      case 'JACKPOT':
        return processJackpot(wheelResult);
      default:
        return processDirectPrize(wheelResult);
    }
  }

  return baseResult;
}
```

---

## 통계 (343개 게임 분석 기준)

| 항목 | 수치 |
|------|------|
| **총 슬롯 게임 수** | 356개 |
| **Hold & Spin 게임** | 45개+ |
| **Cascade 게임** | 30개+ |
| **Nudge 게임** | 15개+ |
| **Mystery Symbol 게임** | 25개+ |
| **Wheel 보너스 게임** | 50개+ |
| **Infinity Reel 게임** | 10개+ |
| **Bingo 게임** | 8개+ |
| **평균 보너스 타입 수** | 3.2개 |
| **최대 보너스 타입 수** | 10개 (Treasure Goblin) |
