# Games Server - 게임 엔진 아키텍처 문서

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

## 게임 ID 및 파일 매핑 (전체 목록)

### 파일 통계

| 항목 | 수량 |
|------|------|
| 메인 게임 파일 (.ts) | 356개 |
| 인터페이스 파일 (.interface.ts) | 148개 |
| 값 정의 파일 (.value.ts) | 357개 |
| 테스트 파일 | 314개 |

### 전체 게임 ID-파일명 매핑 (384개 게임)

#### 슬롯 게임 ID 1-50

| ID | 게임명 | 파일명 | Base Bet | 보너스 ID |
|----|--------|--------|----------|-----------|
| 1 | Jungle Quest | jungle_quest.ts | 6000 | 100-103 |
| 2 | Wild City | wild_city.ts | - | 200-202 |
| 3 | Billionaire Beauty | billionaire_beauty.ts | - | 300-302 |
| 4 | Red White Blue Stacked | red_white_blue_stacked.ts | - | 400-401 |
| 5 | Double Wheel Diamond | double_wheel_diamond.ts | - | 500-502 |
| 6 | Super Re-Spin | - | - | 600+ |
| 7 | Gold Times Pay | gold_times_pay.ts | - | 700+ |
| 8 | Dynamite Wild | dynamite_wild.ts | - | 800+ |
| 9 | Dragons Roar | dragons_roar.ts | - | 900+ |
| 10 | Majestic Fortune | majestic_fortune.ts | - | 1000+ |
| 11 | Aztec Charms | aztec_charms.ts | - | 1100+ |
| 12 | Great Catsby | great_catsby.ts | - | 1200-1201 |
| 13 | Tropical Dreams | - | - | 1300+ |
| 14 | Colossal Classic | colossal_classic.ts | - | 1400+ |
| 15 | Lucky Pet Salon | lucky_pet_salon.ts | - | 1500+ |
| 16 | Big Top Dogs | big_top_dogs.ts | - | 1600+ |
| 17 | Mayan Dynasty | mayan_dynasty.ts | - | 1700+ |
| 18 | Matsuri | matsuri.ts | - | 1800+ |
| 19 | Little Glass Slipper | little_glass_slipper.ts | - | 1900+ |
| 20 | Reign of Gnomes | reign_of_gnomes.ts | - | 2000+ |
| 21 | Rich Hits Red | rich_hits_red.ts | - | 2100+ |
| 30 | God of the sea | god_of_the_sea.ts | - | 3000+ |
| 33 | Kingdom of Zeus | kingdom_of_zeus.ts | - | 3300+ |
| 34 | Mystic Moon | mystic_moon.ts | - | 3400+ |
| 40 | Wild Saloon | - | 5000 | 4000-4003 |
| 48 | Mega Money Inferno | mega_money_inferno.ts | - | 4800+ |
| 49 | Grand Jazzy Sevens | grand_jazzy_sevens.ts | - | 4900+ |
| 50 | Great Ocean | great_ocean.ts | - | 5000+ |

#### 슬롯 게임 ID 51-100

| ID | 게임명 | 파일명 | 보너스 ID |
|----|--------|--------|-----------|
| 55 | God of the sky | god_of_the_sky.ts | 5500-5505 |
| 59 | Mega Money Lightning | mega_money_lightning.ts | 5900-5902 |
| 60 | Spooky Sweets | spooky_sweets.ts | 6000-6004 |
| 61 | Rich Hits Gold | rich_hits_gold.ts | 6100+ |
| 68 | Rich Hits Platinum | rich_hits_platinum.ts | 6800+ |
| 72 | Midas Gold | midas_gold.ts | 7200+ |
| 82 | Five Phoenixes | five_phoenixes.ts | 8200+ |
| 83 | Four Gods | four_gods.ts | 8300+ |
| 84 | Book of Ramses | book_of_ramses.ts | 8400+ |
| 85 | Eureka Rush | eureka_rush.ts | 8501-8505 |
| 87 | Taurus Link | buffalo_link.ts | 8700+ |
| 88 | Fireball Frenzy | fireball_frenzy.ts | 8800+ |
| 91 | Hot Hot Chili | hot_hot_chili.ts | 9100+ |
| 92 | Golden Piggy | golden_piggy.ts | 9200+ |
| 96 | Genie | genie.ts | 9600+ |
| 100 | Bingo Lane | bingo_lane.ts | 10000+ |

#### 슬롯 게임 ID 101-200

| ID | 게임명 | 파일명 | 보너스 ID |
|----|--------|--------|-----------|
| 102 | Ice Queen | ice_queen.ts | 10200+ |
| 112 | Roar King | roar_king.ts | 11200+ |
| 121 | Inferno Balls | inferno_balls.ts | 12100+ |
| 127 | Grand Mammoth | grand_mammoth.ts | 12700+ |
| 133 | Rage of Rhino | rage_of_rhino.ts | 13300+ |
| 134 | Cash Train | cash_train.ts | 13400+ |
| 136 | Howling Wilds | howling_wilds.ts | 13600+ |
| 151 | Mr Gooses Bank | mr_gooses_bank.ts | 15100+ |
| 158 | Incredible Robin | incredible_robin.ts | 15800+ |
| 159 | Eagle-s Rise | eagle-s_rise.ts | 15900+ |
| 163 | Incredible Rhino | incredible_rhino.ts | 16300+ |
| 164 | Grand Stampede | grand_stampede.ts | 16400+ |
| 165 | Sovereign Dragon | sovereign_dragon.ts | 16500+ |
| 166 | Treasure Goblin | treasure_goblin.ts | 16600-16609 |
| 174 | Cashing In | cashing_in.ts | 17400+ |
| 176 | Cash Billionaire Bingo | cash_billionaire_bingo.ts | 17600+ |
| 177 | Mighty Wheels | charming_wheel.ts | 17700+ |
| 182 | Bingo Inferno | bingo_inferno.ts | 18200+ |
| 184 | Cashing In Gold | cashing_in_gold.ts | 18400+ |
| 191 | Buffalo Triple Riches | buffalo_triple_riches.ts | 19100+ |
| 192 | Rhino Triple Riches | rhino_triple_riches.ts | 19200+ |

#### 슬롯 게임 ID 201-300

| ID | 게임명 | 파일명 | 보너스 ID |
|----|--------|--------|-----------|
| 209 | Flamin Hot Link | flamin_hot_link.ts | 20900+ |
| 216 | Nudge Strike | nudge_strike.ts | 21600+ |
| 222 | Medusa Unlimited | medusa_unlimited.ts | 22200+ |
| 228 | Rich Hits Strike | rich_hits_strike.ts | 22800+ |
| 233 | Bingo Inferno Deluxe | bingo_inferno_deluxe.ts | 23300+ |
| 234 | Barbarian Destroyer | barbarian_destroyer.ts | 23400-23404 |
| 237 | Coin Bite | coin_bite.ts | 23700+ |
| 243 | Alpha Bravo Cash | alpha_bravo_cash.ts | 24300-24302 |
| 245 | Pearlinko | pearlinko.ts | 24500+ |
| 254 | Djinn the Win | djinn_the_win.ts | 25400-25402 |
| 259 | AZ Stack Rise | az_stack_rise.ts | 25900+ |
| 260 | Bacon Trio | bacon_trio.ts | 26000+ |
| 262 | Mighty Pot Hero | mighty_pot_hero.ts | 26200+ |
| 263 | Grand Hippo | grand_hippo.ts | 26300+ |
| 264 | Barbarian Golden Axe | barbarian_golden_axe.ts | 26400+ |
| 267 | Beans Grow Wild | beans_grow_wild.ts | 26700+ |
| 274 | Rich for the Star | rich_for_the_star.ts | 27400+ |
| 279 | Infinite Fireball Link | infinite_fireball_link.ts | 27900+ |
| 282 | Taurus Burst (Buffalo Burst) | buffalo_burst.ts | 28200-28203 |
| 284 | Lion Rumble | lion_rumble.ts | 28400+ |
| 285 | Cash Hits Blitz | cash_hits_blitz.ts | 28500+ |
| 287 | Mighty Dragons | mighty_dragons.ts | 28700+ |
| 296 | Mighty Dragons Yellow | mighty_dragons_yellow.ts | 29600+ |
| 297 | Rich Hits Triple Pop | rich_hits_triple_pop.ts | 29700+ |
| 298 | Dragons Gate | dragons_gate.ts | 29800+ |

#### 슬롯 게임 ID 301-362

| ID | 게임명 | 파일명 | 보너스 ID |
|----|--------|--------|-----------|
| 301 | Rich for the Star Winter | rich_for_the_star_winter.ts | 30100+ |
| 304 | Big Bang Bounty | big_bang_bounty.ts | 30401-30402 |
| 307 | Lucky Seven Locomotive | lucky_seven_locomotive.ts | 30700+ |
| 308 | Bingo Ahoy | bingo_ahoy.ts | 30800-30805 |
| 309 | Dino Duo Wild | dino_duo_wild.ts | 30900+ |
| 313 | Fire Pawtrol | fire_pawtrol.ts | 31300+ |
| 315 | Golden Mango Kingdom | golden_mango_kingdom.ts | 31500+ |
| 316 | Cowsmic Invaders | cowsmic_invaders.ts | 31600+ |
| 319 | Kung-Food Panda | kung_food_panda.ts | 31900+ |
| 322 | Captain Mango Bingo | captain_mango_bingo.ts | 32200+ |
| 331 | Radiant Cleopatra | radiant_cleopatra.ts | 33100+ |
| 333 | Fu Dai Bao-Nanza | fu_dai_bao_nanza.ts | 33300+ |
| 336 | Panda Unleashed | panda_unleashed.ts | 33600+ |
| 337 | Crystal Nudge | crystal_nudge.ts | 33700+ |
| 338 | Phoenix Bounty | phoenix_bounty.ts | 33800+ |
| 343 | Supercharge X-treme | supercharge_x_treme.ts | 34300-34305 |
| 345 | Boombastic Bingo | boombastic_bingo.ts | 34500+ |
| 346 | Mammoth Mega Rush | mammoth_mega_rush.ts | 34600+ |
| 350 | Rhino Mega Splits | rhino_mega_splits.ts | 35000+ |
| 351 | Fire Bat 888 | fire_bat_888.ts | 35100+ |
| 354 | El Oro De Zorro | el_oro_de_zorro.ts | 35400+ |
| 356 | Grand Gator | grand_gator.ts | 35600+ |
| 358 | Big Footunate Wheel | big_footunate_wheel.ts | 35800+ |

#### 비디오 포커 (ID 100001-400017)

| ID | 게임명 | Base Bet |
|----|--------|----------|
| 100001 | Jacks or Better Classic | 5000 |
| 100002 | Jacks or Better Mega X | 5000 |
| 100003 | Jacks or Better Jackpot | 5000 |
| 400001 | Jacks or Better Classic (GEM) | 5000 |
| 400004 | Jacks Wild (GEM) | 5000 |
| 400006 | Deuces Wild Gem | 5000 |
| 400012 | Bonus Poker Gem | 5000 |
| 400013 | Double Bonus Poker | 5000 |
| 400015 | Double Double Bonus Poker | 5000 |

#### 케노 게임 (ID 200001-500005)

| ID | 게임명 |
|----|--------|
| 200001 | Keno Classic |
| 200002 | Keno Mega Wheel |
| 200003 | Keno Mega Money |
| 200004 | Keno Golden Mine |
| 200005 | Keno Dollar |
| 500001-500005 | Keno GEM 버전 |

#### 테이블 게임 (ID 600001-600004)

| ID | 게임명 |
|----|--------|
| 600001 | Blackjack Classic |
| 600002 | Wheel of Blackjack |
| 600003 | Blackjack Lucky Bet |
| 600004 | Blackjack 21 Plus 3 |

### 주요 게임 상세 (샘플)

| 게임명 | 게임ID | 서버 파일명 | 보너스 ID 범위 | Base Bet | 릴 레이아웃 |
|--------|--------|-----------|------------|---------|-----------|
| **Supercharge X-treme** | 343 | supercharge_x_treme.ts | 34300-34305 | 1000 | 1x6 |
| **Buffalo Burst (Taurus Burst)** | 282 | buffalo_burst.ts | 28200-28203 | 5000 | 4x5 |
| **Bingo Ahoy** | 308 | bingo_ahoy.ts | 30800-30805 | 5000 | 5x5 |
| **Great Catsby** | 12 | great_catsby.ts | 1200-1201 | - | 5x4 |
| **Barbarian Destroyer** | 234 | barbarian_destroyer.ts | 23400-23404 | - | 6x4 |
| **Eureka Rush** | 85 | eureka_rush.ts | 8501-8505 | - | 5x3 |
| **God of the Sky** | 55 | god_of_the_sky.ts | 5500-5505 | - | 5x4 |
| **Treasure Goblin** | 166 | treasure_goblin.ts | 16600-16609 | - | 6x4 |

### 보너스 ID 네이밍 컨벤션

```
기본 규칙: game_id × 100 + sequence

예시:
  - 게임 ID 343 → 보너스 ID 34300, 34301, 34302...
  - 게임 ID 282 → 보너스 ID 28200, 28201, 28202...

티켓형 보너스: game_id × 100 + 50 + sequence
  - 게임 ID 343 → 티켓 보너스 ID 34350, 34351...
  - EQUIV_BONUS_ID로 일반 보너스와 연결
```

---

## 보너스 시스템 상세

### 보너스 ID 체계 (상세)

```
게임 ID: 343 (Supercharge X-treme)
├── 34300: FREE_SPIN (프리스핀)
├── 34301: RETRIGGER (리트리거)
├── 34302: LIGHTNING (번개 보너스)
├── 34303: MULTIPLIER_LIGHTNING (배수 번개)
├── 34304: SUPER_BOOM (슈퍼붐)
├── 34305: INITIALIZE_VALUE (값 초기화)
└── 34350: INSTANT_BONUS (티켓형) → EQUIV: 34300

게임 ID: 282 (Buffalo Burst / Taurus Burst)
├── 28200: FREE_SPIN (프리스핀)
├── 28201: FREE_SPIN_RETRIGGER (리트리거)
├── 28202: WILD_MULTIPLIER_BONUS (와일드 배수)
├── 28203: MIGHTY_LINK (마이티 링크)
└── 28250: FREE_SPIN (티켓형) → EQUIV: 28200

게임 ID: 308 (Bingo Ahoy)
├── 30800: RESPIN_BONUS (리스핀)
├── 30801: BINGO_SYMBOLS_BONUS (빙고 심볼)
├── 30802: BINGO_BONUS (빙고 보너스)
├── 30803: JACKPOT (잭팟)
├── 30804: BONUS_SYMBOL (보너스 심볼)
├── 30805: BONUS_GAME (보너스 게임)
└── 30850: RESPIN_BONUS (티켓형) → EQUIV: 30800

게임 ID: 12 (Great Catsby)
├── 1200: FREE_SPIN
└── 1201: JACKPOT

게임 ID: 234 (Barbarian Destroyer)
├── 23400: FREE_SPIN
├── 23401: FREE_SPIN_RETRIGGER
├── 23402: CASH_PRIZE
├── 23403: INFINITY_REEL
└── 23404: JACKPOT

게임 ID: 55 (God of the Sky)
├── 5500: FREE_SPIN
├── 5501: LIT_BONUS
├── 5502: WILD_SPIN
├── 5503: WHEEL_BONUS
├── 5504: JACKPOT
└── 5505: CREDIT_BONUS

게임 ID: 166 (Treasure Goblin)
├── 16600: GOBLIN_TRIGGERED
├── 16601: LEVEL_CHANGED
├── 16602: PAYOUT
├── 16603: FREE_SPIN
├── 16604: FREE_SPIN_FULL_STACK_WILD
├── 16605: WILD_REEL
├── 16606: DROP_SUB_SYMBOL
├── 16607: SUB_REEL_INFO
├── 16608: CHEST_BONUS
└── 16609: JACKPOT
```

### BonusResultType 분류

```typescript
export enum BonusResultType {
  NORMAL = 1,                           // 일반 보너스 (즉시 지급)
  FREE_SPIN = 2,                        // 프리스핀
  SELECTABLE_FREE_SPIN = 3,             // 선택형 프리스핀
  RESPIN_REGARDED_AS_FREE_SPIN = 4,     // 리스핀 (프리스핀으로 처리)
  PARTY_GAME = 5                        // 커뮤니티/파티 게임
}
```

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

---

## 게임별 특수 기능 비교

### 기능 매트릭스

| 기능 | SCX | Buffalo | Bingo | Great Catsby | Barbarian |
|------|-----|---------|-------|--------------|-----------|
| **당첨 방식** | Value Item | Ways | Paylines | Paylines | Ways |
| **Free Spin** | O | O | O | O | O |
| **Lightning** | O | X | X | X | X |
| **Link Bonus** | X | O | X | X | X |
| **Bingo 진행도** | X | X | O | X | X |
| **Permanent State** | X | X | O | X | X |
| **Infinity Reel** | X | X | X | X | O |
| **Wheel Bonus** | X | X | X | X | X |
| **Multiplier** | O | O | X | X | X |
| **커뮤니티 게임** | X | X | O | X | X |

### 심볼 정의 패턴

```typescript
// 공통 심볼 ID (게임별로 다를 수 있음)
0: Wild
11-13: 특수 심볼 (Scatter, Bonus, Link 등)
1-10: 일반 심볼

// SCX (1x6)
0: Wild, 11: Lightning, 12: Scatter

// Buffalo Burst (4x5)
0: Wild, 12: Bonus (Link), 13: Scatter

// Bingo Ahoy (5x5)
0: Wild, 11: Bingo, 12: Blank, 13: Bonus, 14: Multiplier
```

### 보너스 처리 흐름

```typescript
// 1. 스핀 결과에서 보너스 추출
const bonusList = extractBonus(gameResult, bonusConfig);

// 2. 각 보너스에 대해 처리
for (const bonus of bonusList) {
    switch (bonus.type) {
        case BonusType.FREE_GAME:
            // 프리게임 설정
            bonus.initialSpinCount = 8;
            bonus.freeGameMultiplier = 2;
            break;

        case BonusType.LIGHTNING:
            // 번개 보너스 설정
            bonus.hitValueIndicies = [0, 3, 5];
            bonus.hitValues = [500, 1000, 750];
            break;

        case BonusType.PICK:
            // 선택형 보너스 설정
            bonus.pickOptions = generatePickOptions();
            break;
    }
}

// 3. 결과에 포함
return {
    result: { ... },
    bonusList: bonusList
};
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

## 시뮬레이션 시스템

### 목적
- 게임 밸런스 검증
- RTP 계산 및 검증
- 페이테이블 조정

### 실행 방법

```bash
# ts-node로 직접 실행 (games/ 디렉토리에서)
npx ts-node ./src/slot_simulation/{game_name}.ts {spin_count} -r {rtp} -t {threads}

# 예시: Barbarian Destroyer 시뮬레이션 (10000 스핀, HIGH RTP, 10 스레드)
npx ts-node ./src/slot_simulation/bbd.ts 10000 -r HIGH -t 10

# Super Bonus 전용 시뮬레이션 (InstantBonusSimulator 사용)
npx ts-node ./src/slot_simulation/bbd_sb.ts 1000 --rtp=HIGH --level=1 -t 1
```

### 시뮬레이션 파라미터

- 첫 번째 숫자: 총 스핀 횟수 (positional argument)
- `-r` / `--rtp`: RTP 타입 (LOW, MIDDLE, HIGH)
- `-t`: 스레드 수 (병렬 처리)
- `--level`: 보너스 레벨 (커스텀 시뮬레이션용)

### 시뮬레이션 결과 예시

```
Game: Supercharge X-treme (343)
Total Spins: 1,000,000
Total Bet: 500,000,000
Total Win: 480,000,000
RTP: 96.0%

Symbol Distribution:
  - Wild (1): 2.5%
  - Scatter (2): 1.8%
  - Buffalo (5): 8.2%
  ...

Bonus Trigger Rate:
  - Free Game: 0.8%
  - Lightning: 2.1%
  - Jackpot: 0.001%
```

### 시뮬레이션 작성 시 주의사항

#### ⚠️ output 배열 구조 확인 필수

보너스 결과의 `output` 배열 구조는 **게임/레벨마다 다를 수 있습니다**. 시뮬레이션 코드 작성 전에 반드시 게임 로직에서 output이 어떻게 구성되는지 확인하세요.

**잘못된 가정의 예 (Barbarian Destroyer Super Infinity):**

```typescript
// ❌ 잘못된 코드 - output 구조를 확인하지 않고 가정
const spinCount = sbLevel === 3 ? outputLength / 2 : outputLength;

// 실제 구조:
// - Level 1, 2, 3 모두: 1 슬래시당 ROW_COUNT(3)개 배열이 allOutput에 push됨
// - Level 1, 2: [[sym], [sym], [sym]] (각 1심볼)
// - Level 3: [[sym, sym], [sym, sym], [sym, sym]] (각 2심볼)
// → 모든 레벨에서 outputLength / 3이 실제 슬래시 수!
```

**체크리스트:**

1. [ ] 게임 로직에서 `output` 배열이 어떻게 생성되는지 확인
2. [ ] `allOutput.push()` 호출 방식 확인 (spread vs 단일 push)
3. [ ] 레벨/모드별로 output 구조가 다른지 확인
4. [ ] spinCount, slashCount 등의 의미가 "횟수"인지 "릴 수"인지 명확히

**질문 먼저:**

output 구조가 불확실하면 **코드를 작성하기 전에 질문**하세요:

- "Level별 output 구조가 어떻게 다른가요?"
- "spinCount는 슬래시 횟수를 의미하나요, 릴 개수를 의미하나요?"

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

---

## 새 게임 추가 가이드

### 1. 게임 파일 생성

```bash
# 템플릿 복사
cp slot_game/template.ts slot_game/new_game.ts
cp slot_game/template.value.ts slot_game/new_game.value.ts
cp slot_game/template.bonus.ts slot_game/new_game.bonus.ts
```

### 2. 게임 ID 할당

```typescript
// common/types.ts
export const GAME_IDS = {
    // ... 기존 게임
    NEW_GAME: 999,
};
```

### 3. 릴스트립 및 페이테이블 정의

```typescript
// new_game.value.ts
export const reelStrips = [
    [1, 2, 3, ...], // 릴 0
    [2, 3, 4, ...], // 릴 1
    // ...
];

export const payTable = {
    1: [0, 0, 5, 10, 50, 100],   // 심볼 1
    2: [0, 0, 3, 8, 40, 80],     // 심볼 2
    // ...
};
```

### 4. 게임 로직 구현

```typescript
// new_game.ts
import { ISlotGame, ISlotGameResult } from '../types/interface';
import { reelStrips, payTable } from './new_game.value';

export class NewGame implements ISlotGame {
    gameId = 999;

    getGameResult(contentsParam: any, betCredit: number): ISlotGameResult {
        // 1. RNG 생성
        const reelOutputList = this.generateRNG();

        // 2. 당첨 계산
        const earnCredit = this.calculateWin(reelOutputList, betCredit);

        // 3. 보너스 확인
        const bonusList = this.checkBonus(reelOutputList);

        return {
            result: { reelOutputList, betCredit, earnCredit, ... },
            customData: { ... },
            bonusList
        };
    }
}
```

### 5. 게임 매니저에 등록

```typescript
// game_manager.ts
import { NewGame } from './slot_game/new_game';

gameModules[999] = new NewGame();
```

---

## 디버깅 및 로깅

### 로그 레벨

```typescript
// Winston 로거 설정
const logger = winston.createLogger({
    level: 'info',  // debug, info, warn, error
    format: winston.format.json(),
    transports: [
        new winston.transports.Console(),
        new winston.transports.File({ filename: 'games.log' })
    ]
});

// 사용
logger.info('Spin result', { gameId, earnCredit, bonusCount: bonusList.length });
logger.debug('RNG values', { reelOutputList });
```

### 디버그 모드

```bash
# 상세 로그 출력
DEBUG=* npm start:debug

# 특정 모듈만 디버그
DEBUG=games:slot npm start:debug
```

---

## 주의사항

1. **RNG 보안**: 난수 생성은 반드시 서버에서만 수행
2. **RTP 모니터링**: 실시간 RTP가 목표 범위를 벗어나면 알림
3. **잭팟 무결성**: 잭팟 풀 변경은 트랜잭션으로 처리
4. **성능 최적화**: 무거운 계산은 캐싱 활용
5. **버전 관리**: 게임 설정 변경 시 버전 기록

---

## 게임 로직 개발 필수 규칙

### 데이터 참조 규칙

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

### Mystery Symbol 처리 주의사항

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

### INSTANT_BONUS 디버그 타입 구현 (티켓 보너스 시뮬레이션)

티켓으로 진입하는 보너스 (Super Bonus 등)를 시뮬레이션할 때 `SLOT_DEBUG.INSTANT_BONUS` (400)를 처리해야 합니다.

#### Step 1: modifyDebugParam에서 플래그 설정

```typescript
case SLOT_DEBUG.INSTANT_BONUS:
  const simLevel = (Game as any).simulationSuperBonusLevel || 1;
  newDebugParam = {
    force_instant_bonus: true,
    super_bonus_level: simLevel,
  };
  break;
```

#### Step 2: getGameResult에서 free_spin 세션 설정 (bacon_trio 패턴)

```typescript
if (modifiedDebugParam?.force_instant_bonus && !isFreeSpin) {
  // superBonusInfo는 RTP로 키 지정됨 (level이 아님!)
  const superBonusWeights = superBonusInfo[rtp];
  const spinCount = superBonusWeights.super_free_spin_count;

  // free_spin 세션 설정
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

  // 빈 결과 반환 - 시뮬레이터가 free_spin 세션을 감지함
  return {
    result: { earn_credit: 0, reel_output_list: [], ... },
    bonusResultList: [],  // 빈 배열!
    symbolCountInfo: {},
    extraData: { ... },
  };
}
```

#### Step 3: 시뮬레이션 파일에서 레벨 전달

```typescript
const game = new Game();
(Game as any).simulationSuperBonusLevel = level;
const simulator = new Simulator(VALUE.game_id, baseBet, game);
// InstantBonusSimulator가 자동으로 INSTANT_BONUS (400) 설정
```

---

## 보너스 생성 패턴

### _GenerateBonusResult 함수

각 게임에서 보너스를 생성할 때 사용하는 핵심 함수입니다.

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
  NEW_BONUS: { BONUS_ID: gameId * 100 + sequence },  // 추가
} as const;
```

#### 2. 응답 인터페이스 정의 (game.interface.ts)

```typescript
export interface INewBonusResponse {
  custom_field: number;
  // 필요한 필드 추가
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
  - `getNormalGameResult()`: 베이스 게임
  - `getFreeGameResult()`: 프리게임
- **연출 전용 보너스**: `earnCredit: 0`, `claimType: NONE`으로 설정
- **보너스 ID 규칙**: `gameId × 100 + sequence` (예: MSL 338 → 33800, 33801, ...)

---

## ContentsStore / GameProgress / CustomData 시스템

343개 게임 분석에서 발견된 핵심 데이터 저장 패턴입니다.

### 1. ContentsStore - 세션 간 영속 데이터

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

#### ContentsStore 사용 게임

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

#### 사용 예시 (Party Bonus 게임)

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

### 2. GameProgress - 영구 게임 진행 상태

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

#### GameProgress 사용 게임 (Bingo 계열)

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

#### 베팅별 독립 진행도 패턴

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

### 3. CustomData - 게임별 커스텀 데이터

**정의**: 각 게임의 고유한 상태를 저장하는 구조입니다. `gameSession.custom_data`에 저장됩니다.

```typescript
// 게임 세션 내 CustomData 위치
export interface IGameSessionSlotGame {
  // ...
  custom_data: any;   // 게임별 CustomData가 여기 저장됨
  // ...
}
```

#### CustomData 초기화 패턴

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

#### CustomData 유형별 구조

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

### 4. 세 가지 시스템 비교

| 특성 | ContentsStore | GameProgress | CustomData |
|------|--------------|--------------|------------|
| **저장 위치** | 별도 DB 테이블 | gameSession 내부 | gameSession.custom_data |
| **영속성** | 세션 간 유지 | 세션 간 유지 | 세션 종료 시 소멸 |
| **용도** | Party/Community | Bingo/Quest | 일반 게임 상태 |
| **변경 추적** | O (change_list) | X | X |
| **게임 설정** | is_contents_store_used | has_permanent_state | - |
| **인터페이스** | ISlotGameWithContentsStore | ISlotGameWithGameProgress | ISlotGame |

---

### 5. 데이터 흐름

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

### 6. 구현 체크리스트

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

## 피처별 서버 구현 패턴 (343개 게임 분석 결과)

343개 슬롯 게임 분석에서 발견된 서버 측 핵심 패턴들입니다.

### 1. Hold & Spin / Link 메카닉

**대표 게임**: Infinite Fireball Link (279), Buffalo Link (146), Flamin Hot Link (253)

#### 서버 로직 구조

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

#### CustomData 구조

```typescript
// CustomData279 (Infinite Fireball Link)
interface IIFLCustomData {
  link_bonus_state: ILinkBonusState | null;
  progressive_multipliers: number[];   // [1, 2, 3, 5, 10]
  spot_split_data: ISpotSplitData[];   // SpotSplit 릴 정보
}
```

#### 보너스 ID 패턴

```
27900: FREE_SPIN
27901: LINK_BONUS (Hold & Spin 진입)
27902: LINK_RESPIN (링크 리스핀)
27903: JACKPOT_WIN
27904: ROW_UNLOCK
27950: INSTANT_LINK (티켓형)
```

---

### 2. Cascade/Tumble 메카닉

**대표 게임**: Oink Overflow (325), Buzz Pop (187), Fruity Luck (142)

#### 서버 로직 구조

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

#### Reel Multiplier 시스템

```typescript
// 릴별 배수 (2x ~ 100x)
const REEL_MULTIPLIERS = [2, 3, 5, 10, 25, 50, 100];

function applyReelMultiplier(baseWin: number, multiplierIndex: number): number {
  return baseWin * REEL_MULTIPLIERS[multiplierIndex];
}
```

#### 보너스 ID 패턴

```
32500: FREE_SPIN
32501: CASCADE_BONUS (캐스케이드 보너스)
32502: REEL_MULTIPLIER (릴 배수 적용)
32503: MEGA_WIN
32504: GRAND_WIN
32550: INSTANT_CASCADE (티켓형)
```

---

### 3. Nudge 메카닉

**대표 게임**: Crystal Nudge (337), Nudge Strike (216)

#### 서버 로직 구조

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

#### CustomData 구조

```typescript
// CustomData337 (Crystal Nudge)
interface ICNCustomData {
  nudge_results: INudgeResult[];     // 넛지 결과 배열
  wild_positions: number[][];        // Wild 위치
  total_nudge_count: number;         // 총 넛지 횟수
}
```

---

### 4. Mystery Symbol 시스템

**대표 게임**: Grand Jazzy Sevens (49), Midas Gold (72)

#### 서버 로직 구조

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

### 5. Wheel 보너스 시스템

**대표 게임**: Golden Piggy (92), Charming Wheels (177), Big Footunate Wheel (358)

#### 서버 로직 구조

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

#### 복잡한 보너스 체인 구조

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

### 6. WILDx2 / 계층형 Wild 시스템

**대표 게임**: Dazzling Jewel Streak (212), Rich Hits (시리즈)

#### 서버 로직 구조

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

### 7. Pre-Event / Fake Reel 시스템

**대표 게임**: Oink Overflow (325), 대부분의 Hold & Spin 게임

#### 서버 로직 구조

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

### 8. Infinity Reel / 동적 릴 확장

**대표 게임**: Barbarian Destroyer (234), Medusa Unlimited (222)

#### 서버 로직 구조

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

### 9. Multi-tier Progressive Jackpot

**대표 게임**: 60개+ 게임 (Mega Money 시리즈, Link 시리즈)

#### 서버 로직 구조

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

### 10. Bingo 진행도 시스템

**대표 게임**: Bingo Ahoy (308), Bingo Inferno (182), Cash Billionaire Bingo (176)

#### 서버 로직 구조 (Permanent State)

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

### 11. SpotSplit Reel 시스템

**대표 게임**: Infinite Fireball Link (279), 일부 Link 게임

#### 서버 로직 구조

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

## 보너스 ID 체계 종합

### 기본 규칙

```
기본 보너스: gameId × 100 + 0~9
├─ +00: FREE_SPIN (주요 보너스)
├─ +01: RETRIGGER / 보조 보너스
├─ +02: LINK / LIGHTNING
├─ +03: WHEEL / PICK
├─ +04: JACKPOT
├─ +05: MULTIPLIER
├─ +06: MEGA_SYMBOL
├─ +07~09: 게임별 추가

티켓형 보너스: gameId × 100 + 50~59
├─ +50: INSTANT_FREE_SPIN → equiv: +00
├─ +51: INSTANT_LINK → equiv: +02
└─ +52~59: 추가 티켓
```

### 게임별 보너스 ID 예시

| 게임 | ID | 보너스 범위 | 특징 |
|------|-----|------------|------|
| Great Catsby | 12 | 1200-1201 | 단순 (2개) |
| Grand Jazzy Sevens | 49 | 4900-4903 | Mystery Symbol |
| Golden Piggy | 92 | 9200-9206 | 복합 (7개) |
| Buffalo Burst | 282 | 28200-28203 | Link 포함 |
| Infinite Fireball Link | 279 | 27900-27904 | Hold & Spin |
| Bingo Ahoy | 308 | 30800-30805 | Bingo + Permanent |
| Oink Overflow | 325 | 32500-32504 | Cascade |
| Crystal Nudge | 337 | 33700-33705 | Nudge |
| Supercharge X-treme | 343 | 34300-34305 | Lightning |

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

---

## 스키마/인터페이스 작성 규칙

### 배열 타입 문법

> ⚠️ **중요**: 클라이언트에서 스키마 파싱 시 `Array<T>` 문법이 인식되지 않습니다.

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

> 🚨 **필수**: 코드 작성 시 아래 ESLint 규칙을 반드시 준수하세요.

### 1. Import 규칙 (no-duplicate-imports)

같은 모듈에서 여러 번 import하지 마세요. 한 줄로 통합하세요.

| 사용 금지 | 사용 권장 |
|-----------|-----------|
| `import { VALUE } from './game';`<br>`import Game from './game';` | `import Game, { VALUE } from './game';` |

**이유**: 코드 중복 방지, 가독성 향상, 번들 최적화

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
| `{ "count": 0, "sum": 0 }` | `{ count: 0, sum: 0 }` |

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

### 코드 작성 전 체크리스트

```markdown
[ ] Import 문이 중복되지 않았는가?
[ ] 파일 끝에 빈 줄이 있는가?
[ ] 객체 키에 불필요한 따옴표가 없는가?
[ ] Switch문에 default case가 있는가?
[ ] 들여쓰기가 2 spaces인가?
```

---

## 관련 문서

- [클라이언트 아키텍처](../client/.claude/CLAUDE.md)
- [클라이언트 피처 카탈로그](../client/Assets/Contents/.claude/CLAUDE.md)
- [LOGIC 서버 아키텍처](../server/.claude/CLAUDE.md)
- [전체 시스템 아키텍처](../.claude/CLAUDE.md)
