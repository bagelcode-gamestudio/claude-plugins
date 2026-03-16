---
description: "Games engine architecture. Core modules, interfaces, spin processing flow, programmed win/loss."
user-invocable: false
globs:
  - "**/GAMES/*.ts"
  - "**/game_manager.ts"
---

# Games Server - 게임 엔진 아키텍처

## 개요

Games Server는 Club Vegas Slots의 **게임 로직 엔진**입니다. 슬롯 게임의 핵심인 **RNG(난수 생성)**, **페이테이블 계산**, **보너스 결정**, **잭팟 시스템**을 담당합니다.

**핵심 역할**:
- 서버 권위적(Server-Authoritative) 게임 결과 생성
- 30개 이상의 슬롯 게임 지원
- RTP(Return to Player) 관리 및 시뮬레이션
- LOGIC 서버와 내부 통신

---

## 프로젝트 구조

```
games/
├── src/
│   ├── GAMES/                      # 게임 로직 코어
│   │   ├── server.ts               # Express 서버 진입점
│   │   ├── router.ts               # API 라우팅
│   │   ├── logic_slot.ts           # 슬롯 게임 핵심 로직 (32KB)
│   │   ├── logic_game_session.ts   # 게임 세션 관리
│   │   ├── logic_game_session_for_slot_game.ts  # 슬롯 전용 세션
│   │   ├── logic_bonus.ts          # 보너스 시스템
│   │   ├── logic_jackpot.ts        # 잭팟 시스템
│   │   ├── logic_keno.ts           # 키노 게임
│   │   ├── logic_video_poker.ts    # 비디오 포커
│   │   ├── logic_table_game.ts     # 테이블 게임
│   │   ├── logic_gamble.ts         # 갬블 기능
│   │   ├── logic_programmed_win.ts # 프로그램드 윈/로스
│   │   └── logic_contents_store.ts # 컨텐츠 스토어
│   │
│   ├── slot_game/                  # 슬롯 게임 모듈 (861개 파일)
│   │   ├── {game_name}.ts          # 게임 로직 (예: buffalo_burst.ts)
│   │   ├── {game_name}.value.ts    # 게임 설정값
│   │   └── {game_name}.bonus.ts    # 보너스 로직
│   │
│   ├── slot_simulation/            # RNG 시뮬레이션 (602개 파일)
│   │   ├── {game_name}_sim.ts      # 시뮬레이션 로직
│   │   └── {game_name}_rtp.ts      # RTP 계산
│   │
│   ├── keno/                       # 키노 게임 (23개 파일)
│   ├── table_game/                 # 테이블 게임
│   ├── common/                     # 공용 유틸리티
│   │   ├── common_util.ts          # 공통 함수
│   │   └── types.ts                # 상수/타입 정의
│   │
│   ├── types/                      # TypeScript 인터페이스
│   │   └── interface.ts            # 게임 결과 인터페이스
│   │
│   ├── game_manager.ts             # 게임 모듈 관리자
│   └── prompt-codegen/             # 코드 생성 도구
│
├── gen/                            # 컴파일된 JavaScript
├── package.json                    # 의존성 정의
└── tsconfig.json                   # TypeScript 설정
```

---

## 기술 스택

```
언어: TypeScript (Node.js)
프레임워크: Express.js 4.14.0
데이터베이스: Redis 2.7.1 (캐싱)

주요 의존성:
  - async 3.2.0
  - uuid 3.1.0
  - crc 3.2.1
  - winston 2.4.5 (로깅)

실행 명령어:
  npm start:debug     # 디버그 모드
  npm run dist-ts     # TypeScript 컴파일
  npm run tsc         # 타입 체크
  npm run prompt-codegen  # 코드 생성
```

---

## 핵심 모듈 설명

### 1. logic_slot.ts - 슬롯 게임 핵심 로직

스핀 요청의 메인 처리 함수들을 포함합니다.

```typescript
// 주요 함수
export function getGameResultPromise(
    gameId: number,
    contentsParam: any,
    betCredit: number,
    ...
): Promise<ISlotGameResult>

export function getSlotGameResultWithProgrammedLoss(
    gameModule: ISlotGame,
    ...
): Promise<ISlotGameResult>
```

**처리 흐름**:
1. 게임 모듈 로드 (`game_manager.getGameModule(gameId)`)
2. RNG 생성 및 릴 결과 결정
3. 페이테이블 기반 당첨 계산
4. 보너스 트리거 확인
5. 잭팟 기여금 처리
6. 결과 반환

### 2. logic_game_session.ts - 게임 세션 관리

게임 세션의 상태를 관리합니다.

```typescript
// 세션 관련 함수
export function createGameSession(userId: number, gameId: number): IGameSession
export function getGameSession(sessionId: string): IGameSession
export function updateGameSession(session: IGameSession, result: any): void
```

### 3. logic_bonus.ts - 보너스 시스템

다양한 보너스 타입을 처리합니다.

**보너스 타입**:
- Free Game (프리스핀)
- Pick Bonus (선택형 보너스)
- Lightning (번개 보너스)
- Wheel Bonus (휠 보너스)
- Jackpot Trigger (잭팟 트리거)

```typescript
// 보너스 추출 함수
export function extractBonus(
    gameResult: ISlotGameResult,
    bonusConfig: IBonusConfig[]
): IBonusResult[]
```

### 4. logic_jackpot.ts - 잭팟 시스템

프로그레시브 잭팟을 관리합니다.

```typescript
// 잭팟 함수
export function contributeToJackpot(betCredit: number, jackpotConfig: any): void
export function checkJackpotTrigger(result: ISlotGameResult): IJackpotWin | null
export function payJackpot(userId: number, jackpotId: number, amount: number): void
```

---

## 게임 결과 인터페이스

### ISlotGameResult

```typescript
interface ISlotGameResult {
    result: {
        reelOutputList: number[];      // 각 릴의 정지 인덱스
        betCredit: number;             // 베팅 금액
        earnCredit: number;            // 획득 금액
        betCountPerReel: number;       // 릴당 베팅 카운트
        jackpotInfo: IJackpotInfo[];   // 잭팟 정보
    };
    customData: {
        [key: string]: any;            // 게임별 커스텀 데이터
    };
    bonusList: IBonusResult[];         // 보너스 결과 리스트
}
```

### IBonusResult

```typescript
interface IBonusResult {
    bonusId: number;                   // 보너스 ID
    uid: string;                       // 고유 ID (클레임용)
    claim_type: number;                // 0: 자동, 1: 선택
    type: number;                      // 보너스 타입
    earnCredit: number;                // 보너스 획득액
    initialSpinCount?: number;         // 프리게임 스핀 수
    // ... 보너스별 추가 데이터
}
```

---

## 슬롯 게임 모듈 구조

### 게임 파일 패턴

각 슬롯 게임은 3개 파일로 구성됩니다:

```
slot_game/
├── buffalo_burst.ts           # 메인 게임 로직
├── buffalo_burst.value.ts     # 설정값 (릴스트립, 페이테이블)
└── buffalo_burst.bonus.ts     # 보너스 로직
```

### 게임 모듈 인터페이스

```typescript
interface ISlotGame {
    gameId: number;

    // 게임 결과 생성
    getGameResult(
        contentsParam: any,
        betCredit: number,
        options?: any
    ): ISlotGameResult;

    // 보너스 처리
    processBonus(
        bonusId: number,
        contentsParam: any
    ): IBonusResult;

    // 설정값 조회
    getConfig(): IGameConfig;
}
```

### 릴스트립 구조

```typescript
// {game_name}.value.ts
export const reelStrips = [
    // 릴 0 (총 100개 심볼)
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 1, 2, 3, 4, ...],
    // 릴 1
    [2, 3, 4, 5, 6, 7, 8, 9, 10, 1, 2, 3, 4, 5, ...],
    // ... 릴 5까지
];

export const payTable = {
    // symbolId: [2개, 3개, 4개, 5개, 6개 매칭 배당]
    5: [0, 5, 10, 50, 100, 500],    // 버팔로 심볼
    6: [0, 3, 8, 40, 80, 400],      // 이글 심볼
    // ...
};
```

---

## API 엔드포인트

### 내부 API (LOGIC 서버 전용)

```
POST /api/games/spin
  - 슬롯 스핀 결과 요청
  - Body: { gameId, contentsParam, betCredit, ... }
  - Response: ISlotGameResult

POST /api/games/bonus
  - 보너스 클레임 요청
  - Body: { gameId, bonusId, bonusUid, ... }
  - Response: IBonusResult

POST /api/games/validate
  - 결과 검증 (디버그용)
  - Body: { gameId, result, ... }
  - Response: { valid: boolean, errors: string[] }

GET /api/games/config/:gameId
  - 게임 설정 조회
  - Response: IGameConfig
```

---

## 스핀 처리 플로우

```
┌─────────────────────────────────────────────────────────────────┐
│                    LOGIC Server 요청                             │
│  spin({ gameId: 343, betCredit: 500, contentsParam: {...} })   │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 1. 게임 모듈 로드                                                │
│    gameModule = game_manager.getGameModule(343)                 │
│    → BuffaloBurst 클래스 로드                                   │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. RNG 생성                                                      │
│    각 릴에 대해 0 ~ reelStrip.length 범위의 난수 생성           │
│    reelOutputList = [12, 45, 23, 67, 34, 89]                   │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. 심볼 그리드 구성                                              │
│    6x6 Deck 생성 (reelStrip[i][index] → symbolId)              │
│    Wild, Scatter 등 특수 심볼 처리                              │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. 당첨 계산 (calculateWayWin)                                   │
│    PayTable 기반 Ways 계산                                      │
│    earnCredit = ways × payValue × (betCredit / baseWager)      │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. 보너스 트리거 확인                                            │
│    Scatter 3개 이상 → Free Game                                 │
│    특수 심볼 조합 → Lightning, Pick Bonus 등                   │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. 잭팟 기여금 처리                                              │
│    betCredit의 일정 비율을 잭팟 풀에 기여                       │
│    jackpotTrigger 확인 → 잭팟 당첨 처리                        │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. 결과 반환                                                     │
│    {                                                             │
│      result: { reelOutputList, earnCredit, ... },               │
│      customData: { CustomData343: {...} },                      │
│      bonusList: [{ bonusId: 34300, ... }]                       │
│    }                                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 프로그램드 윈/로스 시스템

RTP(Return to Player)를 관리하기 위한 시스템입니다.

```typescript
// logic_programmed_win.ts

// 프로그램드 로스: RTP가 높을 때 강제 패배
export function getSlotGameResultWithProgrammedLoss(
    gameModule: ISlotGame,
    currentRTP: number,
    targetRTP: number,
    maxRetries: number = 10
): ISlotGameResult {
    for (let i = 0; i < maxRetries; i++) {
        const result = gameModule.getGameResult(...);

        // RTP가 목표보다 높으면 낮은 당첨 결과 선택
        if (currentRTP > targetRTP && result.earnCredit < threshold) {
            return result;
        }

        // RTP가 목표보다 낮으면 높은 당첨 결과 선택
        if (currentRTP < targetRTP && result.earnCredit > threshold) {
            return result;
        }
    }

    return lastResult;
}
```

---

## LOGIC 서버와의 통신

Games 서버는 LOGIC 서버의 내부 모듈로 작동합니다.

```typescript
// LOGIC 서버에서 Games 함수 직접 호출
import { getGameResultPromise } from '../games/src/GAMES/logic_slot';

async function handleSpin(req, res) {
    const result = await getGameResultPromise(
        gameId,
        contentsParam,
        betCredit,
        ...
    );

    // 결과를 클라이언트에 반환
    res.json({
        error: 0,
        contents: result
    });
}
```
