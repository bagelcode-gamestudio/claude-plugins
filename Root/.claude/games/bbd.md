# BBD (Barbarian Destroyer) - Game Reference

## Quick Info

| 항목 | 값 |
|------|-----|
| Game ID | **234** |
| 코드명 | `bbd` |
| 표시명 | Barbarian Destroyer |
| 릴 구조 | 5x3 (25 paylines) |
| 심볼 수 | 14종 |
| 커밋 프리픽스 | `[BBD]` |
| 복잡도 | ★★★★☆ (Infinity Reels + SIS 3레벨) |
| base_bet | 2,500 |
| base_wager | 25 |

### 서버 아키텍처

```
Client (Unity) ──HTTP REST──→ LOGIC Server (Node.js, server/)
                                    │
                                    │ axios.post('http://127.0.0.1:38292/contents')
                                    ↓
                              Games Engine (독립 프로세스, games/)
```

- LOGIC 서버는 BBD를 **generic 슬롯**으로 처리 (게임별 특수 분기 없음)
- Games 엔진은 npm 패키지가 아닌 **독립 HTTP 서비스** (포트 38292)
- request_type: `C_SLOT_SPIN`, `C_SLOT_CLAIM_BONUS`

---

## File Locations

### LOGIC Server (server/) - BBD 전용 코드 없음

| 파일 | 용도 |
|------|------|
| `server/src/LOGIC/logic_games.ts` | Games 엔진 통신 (generic, 모든 슬롯 공통) |
| `server/src/games.ts` | 게임 메타데이터 (game_id:234, game_title:'bbd') |

### Games Engine (games/)

| 파일 | 용도 |
|------|------|
| `games/src/slot_game/barbarian_destroyer.ts` | 메인 게임 로직 (~2,500줄) |
| `games/src/slot_game/barbarian_destroyer.interface.ts` | 타입 정의 |
| `games/src/slot_game/barbarian_destroyer.value.ts` | 상수/릴셋/배당표 (~1,500줄) |
| `games/src/slot_game/__tests__/barbarian_destroyer.ts` | 테스트 |

### Client (Unity)

**Base:** `client/Assets/Contents/Contents Group 1/Barbarian Destroyer/`

| 폴더 | 내용 |
|------|------|
| `Scripts/` | 36개 C# 스크립트 |
| `Scripts/Behaviours/` | SymbolBehaviour 7종 |
| `Scripts/Symbol/` | 심볼 컨트롤러/데이터 |
| `Scripts/SlotMachine/` | Wild넛지, Scatter스킵 |
| `Scripts/InfinityFeature/` | Infinity Reel 관련 |
| `Scripts/SuperInfinityFeature/` | SIS + Golden Barbarian |
| `Scripts/Animation/` | Animator 상태 체커 |
| `FSM/GameLogic/` | 메인 FSM 3개 + BT 20개+ |
| `Data/Schema.json` | 클라-서버 스키마 |
| `Prefabs/` | 프리팹 (Slot Machine, Symbol, Popup 등) |

---

## Symbol Enum

```
0  = Wild        (3행 연결, 넛지)
1  = H1          (Barbarian, Spine)
2  = H2          (Spine)
3  = M1
4  = M2
5  = L1
6  = L2
7  = L3
8  = L4
9  = L5
10 = Mystery     (스핀 전 변환)
11 = Prize       (Cash 크레딧)
12 = Wheel       (Cash/Multiplier/Jackpot)
13 = Scatter     (3+ → FreeSpin)
```

### Scatter 배당 (xTotalBet)

| 개수 | 3개 | 4개 | 5개 |
|------|-----|-----|-----|
| 배수 | 2x | 6x | 20x |
| 프리스핀 | 10회 | 10회 | 10회 |

### Symbol 특수 동작

| 심볼 | 특수 | 구현 위치 |
|------|------|-----------|
| Wild(0) | 릴에 1개 → 전체 릴 와일드 (Nudge) | `BBDWildNudgeHelper`, 서버 `nudgeWild()` |
| Prize(11) | Golden 분할, Matrix(2x2/2x3) | `BBDCashSymbolBehavior` (1,127줄) |
| Wheel(12) | 3종 결과(Cash/Multiplier/Jackpot), Golden 듀얼 서브휠 | `BBDWheelSymbolBehavior` (1,512줄) |
| Scatter(13) | 3+ → FreeSpin 10회, 기대감 연출 | `BBDScatterSymbolBehavior` |

---

## Bonus ID Map

```
23400  FreeSpin              Scatter 3+ → 10스핀
23401  FreeSpin Retrigger    프리스핀 중 Scatter 3+
23402  Cash Prize            연속 Prize 2릴+
23403  Infinity Reel         프리스핀 중 5릴 모두 특수
23404  Jackpot               Wheel → Jackpot 결과
23405  Super Free Game       Super FSM 진입
23406  Super Infinity Slash  SIS 슬래시 피처

23450  Buy a Bonus           일반 프리게임 구매
23451  Super Bonus Lv1       Super FSG Level 1
23452  Super Bonus Lv2       Super FSG Level 2
23453  Super Bonus Lv3       Super FSG Level 3
```

---

## Game Flow

```
[Base Game]
├─ Line Pay (25라인)
├─ Cash Prize (23402) ← 연속 Prize/Wheel 2릴+
├─ Jackpot (23404) ← Wheel → JACKPOT
├─ Scatter 3+ ──→ [FreeSpin] (23400)
│   ├─ Sticky Barbarian + Line Pay
│   ├─ Retrigger +10 (23401)
│   └─ 5릴 트리거 → [Infinity Reel] (23403, 최대 +40 추가릴=45릴)
│
└─ Buy a Bonus ──→ [FreeSpin] (23450)
                └──→ [SuperFreeSpin] (23451~53)
                      ├─ Sticky + Golden Barbarian
                      ├─ Retrigger +10
                      └─ 5릴 트리거 → [Super Infinity Slash] (23406)
                            ├─ 생명 기반 슬래시 루프
                            ├─ Level 1/2/3
                            └─ Max Win 2,000x
```

---

## Key Classes

### FeatureController (핵심 3개)

| 클래스 | 베이스 | 시작 이벤트 | 종료 이벤트 | 보너스 ID |
|--------|--------|------------|------------|----------|
| `BBDInfinityReelController` | FeatureController (Coroutine) | `StartInfinityFeature` | `InfinityFeatureDone` | 23403 |
| `BBDSuperInfinitySlashController` | FeatureControllerAsync (UniTask) | `StartSuperInfinitySlash` | `SuperInfinitySlashComplete` | 23406 |
| `BBDGoldenBarbarianController` | FeatureController (Coroutine) | `GoldenBarbarianLanded` | `GoldenBarbarianLandedComplete` | - |

### FeatureModule

| 클래스 | 이벤트 핸들러 수 | 역할 |
|--------|----------------|------|
| `BBDStickyModule` | 9개 | Sticky 심볼 관리 허브, Overlay/Mask 처리 |

### SymbolBehaviour (6종)

| 클래스 | 심볼 | 코드량 |
|--------|------|--------|
| `BBDSymbolDefaultBehavior` | L1~L5, M1, M2 | 57줄 |
| `BBDHighSymbolBehavior` | H1, H2 | 55줄 |
| `BBDScatterSymbolBehavior` | Scatter(13) | 80줄 |
| `BBDWildSymbolBehavior` | Wild(0) | 119줄 |
| `BBDCashSymbolBehavior` | Prize(11)/Mystery(10) | 1,127줄 |
| `BBDWheelSymbolBehavior` | Wheel(12) | 1,512줄 |

### Helper/Utility

| 클래스 | 역할 |
|--------|------|
| `BBDStatusUpdater` | 슬롯 프레임 위치/너비 조정, SIS 초기 상태 저장/복원 |
| `BBDWildNudgeHelper` | Wild 1번 릴 넛지 (클라이언트 계산) |
| `BBDWildCheckHelper` | Wild 상단 버퍼 감지 |
| `BBDPrizeWheelNudgeHelper` | 서버 구동 넛지 (UniTask + Coroutine 이중) |
| `BBDScatterSkipHelper` | 모든 Scatter 애니메이션 스킵 |
| `BBDJackpotBoardComponent` | 4티어 잭팟 보드 (Mini/Minor/Major/Grand) |
| `BBDCashCollectBoardHelper` | 크레딧/멀티플라이어 수집 보드 |
| `BBDCashCountBoardHelper` | Cash 심볼 개수 카운트 |
| `BBDSymbolController` | 심볼 상태 중앙 관리, SymbolData 캐싱 |
| `BBDReelMovement` | **ReelMovement 직접 상속** (SlotMaker 네임스페이스), SIS 활성시 릴 정지 변위 커스텀 |

---

## ContentEvent Map (BBD 전용)

### FSM → C# (피처 시작)

```
StartInfinityFeature          → BBDInfinityReelController
StartSuperInfinitySlash       → BBDSuperInfinitySlashController
GoldenBarbarianLanded         → BBDGoldenBarbarianController
```

### C# → FSM (피처 완료)

```
InfinityFeatureDone           ← BBDInfinityReelController
SuperInfinitySlashComplete    ← BBDSuperInfinitySlashController
GoldenBarbarianLandedComplete ← BBDGoldenBarbarianController
OnWheelJackpotLand            ← BBDWheelSymbolBehavior
```

### StickyModule 이벤트 (9개)

```
BBD_CopyCustomDataToOverlay       원본→오버레이 customData 딥카피
BBD_ApplyOverlayToSymbols         오버레이→원본 심볼 복사
BBD_UpdateDeckMaskFromOverlay     오버레이에서 deck.mask 업데이트
BBD_FindScatter1PrizeSymbols      Prize(11) 찾기 → _prizeSymbolWin
BBD_FindScatter2PrizeSymbols      Wheel(12) 찾기 → _wheelSymbolWin
BBD_RestoreStickyWheelSymbols     Sticky Wheel 복원
BBD_ClearExpectationForStickyReels  Sticky열 기대감 초기화
BBD_UpdateDeckFromReelOutput      finalOutputList에서 덱 업데이트
BBD_UpdateBarbarianExpectation    Barbarian 카운트 기반 기대감
```

### 기타

```
OnScatterLanded               ← BBDScatterSymbolBehavior (정지 알림)
SpinSlotMachine               → BBDWildNudgeHelper (리셋)
UpdatedTotalBetCredit         → BBDJackpotBoardComponent
UpdateJackpotInfo             → BBDJackpotBoardComponent
```

---

## Blackboard Paths (BBD 전용)

```
./spin/response/result/reelOutputList         릴 결과
./spin/response/result/finalOutputList        최종 출력 (넛지 후)
./spin/response/customData/bbdOutput          BBD 전용 출력
./spin/response/bonusList                     보너스 목록

./bonus/response/level                        SIS 레벨 (1/2/3)
./bonus/response/livesRemaining               남은 생명
./bonus/response/reelSetIndexPerExtraReel     확장 릴 인덱스
./bonus/response/outputListForExtraReel       확장 릴 출력
./bonus/response/largeSymbolInfoList          대형 심볼 정보 (넛지)

./game/jackpotInfo/eligibleMinBetPerJackpot   잭팟 자격 베팅
/me/credit                                    유저 잔액
```

---

## FSM Structure

### 메인 FSM 3개

| FSM | 역할 | 상태 수 |
|-----|------|---------|
| `GameLogic_FSM` | 베이스 게임 | 26개 |
| `GameLogic_FreeSpin_FSM` | 일반 프리스핀 | 23개 |
| `GameLogic_SuperFreeSpin_FSM` | Super 프리스핀 + SIS | 27개 |

### 주요 BT

| BT | 역할 |
|----|------|
| `GameLogic_GameSpin_BT` | 스핀 실행 |
| `GameLogic_Win_BT` | 당첨 처리 |
| `GameLogic_Check FreeSpin_BT` | 프리스핀 진입 체크 |
| `GameLogic_Check Cash Win_BT` | Cash Prize 체크 |
| `GameLogic_Check Cash Win_Multiplier_BT` | Cash + Multiplier |
| `GameLogic_Check Big Win_BT` | 빅윈 체크 |
| `GameLogic_Sticky_Barbarian_BT` | Sticky 바바리안 |
| `GameLogic_SuperInfinitySlash_BT` | SIS 진입 |
| `GameLogic_SuperFreeSpin_ApplyGolden_BT` | Golden 적용 |
| `GameLogic_SuperFreeSpin_CheckInfinity_BT` | Infinity 체크 |
| `GameLogic_Spin Wheel Symbol_BT` | Wheel 심볼 |
| `GameLogic_InstantBonus_BT` | 인스턴트 보너스 |
| `FreeSpin_Infinite Reel_BT` | Infinity Reel |

---

## Super Infinity Slash (SIS) - 핵심 피처

### 3개 레벨

| | Level 1 | Level 2 | Level 3 |
|---|---------|---------|---------|
| 릴 구조 | 3x1 단일 | 3x1 단일 | **3x2 듀얼 스트립** |
| ReelSet | 8 | 9 | 10 |
| Golden | 기본 | Cash 분할(1→2) | Cash/Wheel 분할 + 넛지 |
| Matrix | 없음 | 없음 | 2x2, 2x3 대형 심볼 |

### 생명 시스템

- 시작: `livesRemaining = 0` → `targetLives`까지 순차 충전
- 빈 슬래시: 생명 -1, 릴 확장 없이 리스핀
- Life 심볼: 생명 +1 (최대 3)
- 모든 생명 소진 시 종료

### Static IsActive 패턴

```csharp
public static bool IsActive { get; private set; }  // 프로퍼티 (필드 아님!)
// 참조: BBDReelMovement, BBDSymbolData
```

### MAX_LIVES (서버 vs 클라이언트)

| 위치 | 값 | 의미 |
|------|-----|------|
| 서버 `SUPER_BONUS_MAX_LIVES` | **3** | SIS 세션 내 생명 상한 (게임 밸런스) |
| 클라이언트 `MAX_LIVES` | **20** | UI 절대 상한 (안전장치, 도달 불가) |
| 클라이언트 `initialLives` | **3** | SerializeField, 실제 시작 생명 (서버와 동기화) |

---

## Server Key Functions

### 퍼블릭 API (Game class export)

| 함수 | 역할 | 호출 경로 |
|------|------|-----------|
| `getGameResult()` | 스핀 결과 반환 | LOGIC → `C_SLOT_SPIN` |
| `claimBonus()` | 보너스 클레임 | LOGIC → `C_SLOT_CLAIM_BONUS` |
| `claimRewardBonus()` | 리워드 보너스 클레임 | LOGIC → 리워드 |
| `useTicket()` | Buy a Bonus 처리 | LOGIC → 티켓 사용 |

### 내부 핵심 함수 (barbarian_destroyer.ts)

| 함수 | 역할 |
|------|------|
| `spin()` | 메인 스핀 처리 |
| `processFreeSpin()` | 프리스핀 스핀 |
| `processSuperFreeSpin()` | Super FSG 스핀 |
| `processInfinityReel()` | Infinity Reel 확장 |
| `processSuperInfinitySlash()` | SIS 슬래시 |
| `nudgeWild()` | Wild 넛지 (전체 릴 와일드) |
| `processCashPrize()` | Cash Prize 계산 |
| `processWheelResult()` | Wheel 결과 결정 |
| `processGoldenSplit()` | Golden 분할 |
| `calculateMultiplier()` | 멀티플라이어 계산 |

### barbarian_destroyer.value.ts 주요 상수

| 상수 | 값 | 설명 |
|------|-----|------|
| `REEL_SET` | 11개 | 기본(0~5) + 보너스(6) + **Fake(7)** + SIS(8~10) |
| `PAYLINE_COUNT` | 25 | 페이라인 수 |
| `MAX_WIN_MULTIPLIER` | 2000 | 최대 배당 배수 |
| `JACKPOT_MULTIPLIERS` | {Grand:1500, Major:150, Minor:60, Mini:18} | 잭팟 고정 배수 |
| `FREE_SPIN_COUNT` | 10 | 프리스핀 기본 횟수 |
| `SIS_INITIAL_LIVES` | {Lv1:3, Lv2:3, Lv3:3} | SIS 초기 생명 |
| `SIS_MAX_LIVES` | 3 | SIS 최대 생명 |
| `INFINITY_MAX_REELS` | 40 | **추가** 릴 최대 수 (기본5 + 추가40 = 최대45릴) |
| `MULTIPLIER_CAP` | 80000 | 멀티플라이어 상한 |

---

## Jackpot (Progressive RICH_HITS + Fixed Seed 하이브리드)

| 잭팟 | 고정배수 (xBaseBet) | 기여율 | 기여범위 |
|-------|---------------------|--------|----------|
| Grand | 1,500x | 0.15% | GAME (전체) |
| Major | 150x | 0.20% | ROOM |
| Minor | 60x | 0.30% | ROOM |
| Mini | 18x | 0.35% | ROOM |

- **타입**: `RICH_HITS` Progressive + Fixed Seed 값 하이브리드
- **자격 조건**: `eligible_min_bet_scale` 이상 베팅 시 잭팟 자격 획득
- **트리거**: Wheel 심볼 → JACKPOT 결과 시 해당 티어 잭팟 수여

---

## Patterns & Gotchas

### 알아야 할 것

1. **Coroutine vs UniTask 혼용**: InfinityReel(Coroutine) vs SIS(UniTask), `BBDPrizeWheelNudgeHelper`가 양쪽 다 구현
2. **Static IsActive**: `BBDSuperInfinitySlashController.IsActive`로 SIS 활성 여부 전역 참조 (BBDReelMovement, BBDSymbolData)
3. **SkipAfter() 체인 패턴**: 모든 BBD SymbolBehaviour가 `BBDSkipTimerSymbolBehaviour`를 상속, `SkipAfter(seconds)` 체이닝
4. **Overlay/Mask 시스템**: Sticky 심볼은 Overlay|Reject 마스크로 CalcLineWin 제외
5. **column >= 5 분기**: 확장 릴(5번+) 심볼은 spriteIndex 1 사용 (다른 외형)
6. **H1 대체 버그 수정**: Wild가 H1 위치일 때 `SetSprite(1)`로 H1 스프라이트 표시
7. **BBDCashSymbolBehavior 17개 CachedObjects**: 인덱스 0~21 중 15~19 미사용 갭, Prize/Wheel/Life/Mystery/Golden/Matrix 조합별 프리팹 전환
8. **Deep Copy 패턴**: StickyModule에서 SymbolInfo 딥카피 (customData, link, subSymbol 포함)
9. **서버 earn_credit=0**: Super Free Game은 스핀 자체에서 라인페이 0, 모든 수익은 보너스에서

### 수정 시 주의

- SIS 관련 수정 시 `IsActive` 플래그 영향 범위 확인 (BBDReelMovement, BBDSymbolData 참조)
- Sticky 로직 수정 시 Overlay/Mask 동기화 확인
- Prize/Wheel Behaviour 수정 시 Golden/Matrix 분기 주의
- 서버 릴셋 수정 시 클라이언트 ReelSet 인덱스(8/9/10) 매칭 확인
- Level 3 듀얼 스트립: Strip A = `i*2`, Strip B = `i*2+1` 인덱스 규칙
- **Fake Reel(인덱스 7)**: 릴셋에 존재하지만 게임 로직에서 직접 사용되지 않음 - 수정 시 주의
- **BBDReelMovement는 ReelMovement 직접 상속** (VerticalReelMovement 아님)
- **LOGIC 서버는 BBD를 generic 처리**: 게임별 서버 코드 수정은 games/ 폴더만 해당
- **Ticketed Bonus (23451~53)**: 모두 `EQUIV_BONUS_ID: 23405` (Super Free Game)로 매핑
- **SIS Jackpot 별도 정산**: Jackpot 크레딧은 멀티플라이어에 미포함, 별도 JackpotBonus(23404)로 지급
- **클라이언트 폴백 랜덤**: customData null이면 Wheel/Prize가 랜덤 가짜 데이터 생성 (릴 스핀 중 시각적 폴백)
- **BBDSymbolEventHandler**: Win/InfinityWin 애니메이션을 오버레이(Sticky) 심볼로 라우팅

---

## Paytable (xBaseWager=25 기준)

| 심볼 | ID | 3매칭 | 4매칭 | 5매칭 |
|------|-----|-------|-------|-------|
| Wild | 0 | 0 | 0 | 0 | (대체만)
| H1 | 1 | 150 | 450 | 1000 |
| H2 | 2 | 100 | 250 | 750 |
| M1 | 3 | 75 | 200 | 500 |
| M2 | 4 | 50 | 100 | 400 |
| L1 | 5 | 40 | 80 | 125 |
| L2 | 6 | 40 | 80 | 125 |
| L3 | 7 | 30 | 75 | 100 |
| L4 | 8 | 30 | 75 | 100 |
| L5 | 9 | 25 | 50 | 100 |

### Cash Prize Values (xBaseBet)

```
인덱스 0~3: Jackpot (별도 처리)
인덱스 4~12: 12, 10, 8, 6, 5, 4, 3, 2, 1
```

### Multiplier Values + Weights

| 인덱스 | 배수 | Base/Free Weight | Super/SIS Weight |
|--------|------|------------------|------------------|
| 0 | 10x | 10 (0.8%) | 5 (0.4%) |
| 1 | 8x | 20 | 10 |
| 2 | 6x | 45 | 25 |
| 3 | 5x | 100 | 75 |
| 4 | 4x | 150 | 200 |
| 5 | 3x | 250 | 300 |
| 6 | 2x | 650 (53.1%) | 650 (51.4%) |

---

## Wheel Probability (릴별 Cash:Multiplier:Jackpot)

### Base/Free Game

| 릴 | Cash | Mult | Jackpot | 비고 |
|----|------|------|---------|------|
| 3 | 14 | 9 | 7 | Jackpot 23.3% |
| 4 | 14 | 6 | 10 | Jackpot 33.3% |
| 5 | 14 | 3 | 13 | Jackpot **43.3%** |

### Super Infinity Slash (3릴 동일)

| Cash | Mult | Jackpot | 비고 |
|------|------|---------|------|
| 60 | 10 | 1 | Jackpot **1.4%** (극히 낮음) |

### Jackpot 티어별 가중치 (잭팟 당첨 시)

| | GRAND | MAJOR | MINOR | MINI |
|---|-------|-------|-------|------|
| Base 릴3 | 1 | 8 | 45 | 150 |
| Base 릴5 | 1 | 10 | 55 | 300 |
| SIS | 1 | 10 | 40 | 100 |

### 잭팟 자격 (eligible_min_bet_scale)

| 잭팟 | 최소 베팅 | 스케일 |
|------|----------|--------|
| Mini | 2x baseBet | 2 |
| Minor | 4x baseBet | 4 |
| Major | 10x baseBet | 10 |
| Grand | 20x baseBet | 20 |

---

## Golden 분할 규칙 (Level별)

### Golden Type Weights

| Level | Cash% | Wheel% |
|-------|-------|--------|
| 1 | 100% | **0%** (Golden 불가) |
| 2 | 80% | 20% |
| 3 | 70% | 30% |

### SIS 내 Golden 심볼 확률

| Level | Prize Golden% | Wheel Golden% |
|-------|--------------|---------------|
| 1 | **0%** | **0%** |
| 2 | 40% | 60% |
| 3 | 50% | 70% |

### Golden Wheel 멀티플라이어 계산

- Cash: `cashValue = value1 + value2` (합산)
- Jackpot: `jackpotValue = jp1 + jp2` (합산)
- Multiplier: `multiplierFactor = 1 * mult1 * mult2` (**곱**)

---

## Schema.json (CustomData 구조)

### CustomData234 (스핀 응답)

| 필드 | 타입 | 용도 |
|------|------|------|
| `bbdOutput` | BBDSymbolInfo[][] | 릴별 심볼 상세 (subtype, isGolden 등) |
| `nowMysterySymbols` | map | 현재 스핀 미스터리 변환 맵 |
| `nextMysterySymbols` | map | 다음 스핀 미스터리 변환 맵 |
| `finalOutputList` | int[][] | 넛지 후 최종 릴 출력 |
| `barbarianPositions` | array | H1 Sticky 위치/값 |
| `goldenPositions` | array | Golden Barbarian 위치 |
| `goldenSplitInfo` | array | Golden 분할 결과 |
| `isSuperInfinityTriggered` | bool | SIS 트리거 여부 |

### BBDSymbolInfo (심볼 단위)

| 필드 | 타입 | 용도 |
|------|------|------|
| `type` | string | "normal"/"cash prize"/"jackpot prize"/"multiplier"/"life" |
| `index` | int | 심볼/값 인덱스 |
| `subtype` | string | Wheel 서브타입 |
| `isGolden` | bool | Golden 여부 |
| `subIndex` | int | Golden 분할 서브 인덱스 |
| `goldenSubType` | string | Golden 분할 서브타입 |

### 주요 BonusResult 스키마

| 보너스 | 핵심 필드 |
|--------|----------|
| 23400 FreeSpin | `addedSpinCount` |
| 23403 InfinityReel | `reelSetIndexPerExtraReel`, `outputListForExtraReel`, `bbdOutput` |
| 23404 Jackpot | `jackpotIndex`, `isJackpot` |
| 23405 SuperFreeGame | `level`, `initialSpinCount`, `goldenPositions` |
| 23406 SIS | `level`, `output`, `reelSetIndices`, `finalMultiplier`, `livesRemaining`, `largeSymbolInfoList` + 기타 |

---

## BBDSymbolData 계층 (클라이언트)

```
BBDSymbolData (abstract)
├── BBDWheelSymbolData    → Wheel(12), WheelResult 3종
├── BBDPrizeSymbolData    → Prize(11), credit*multiplier
└── BBDDefaultSymbolData  → 나머지 (BBDSymbol enum)

BBDWheelResult (abstract)
├── BBDWheelResultCash       → credit * multiplier
├── BBDWheelResultMultiplier → X2~X10
└── BBDWheelResultJackpot    → Mini/Minor/Major/Grand

enum BBDJackpot { Mini=0, Minor=1, Major=2, Grand=3 }
```

### customData 키 상수

| 키 | 용도 |
|-----|------|
| `"SymbolSubtype"` | "cash prize" / "jackpot prize" / "multiplier" |
| `"ValueIndex"` | cashPrizes/multipliers 인덱스 |
| `"IsGolden"` | Golden 여부 |
| `"GoldenSubType"` | Golden 분할 서브타입 |
| `"SubIndex"` | Golden 분할 서브 인덱스 |

### 클라이언트 폴백 랜덤 상수

| 상수 | 값 | 위치 |
|------|-----|------|
| `SUPER_BONUS_GOLDEN_SYMBOL_CHANCE` | **0.30** (30%) | BBDWheelSymbolData |
| `SUPER_BONUS_LIFE_SYMBOL_CHANCE` | 0.05 (5%, 미사용) | BBDPrizeSymbolData |

---

## Animation Bridge 패턴 (3쌍)

| MonoBehaviour | StateMachineBehaviour | 연결 필드 |
|--------------|----------------------|-----------|
| BBDAnimationChecker | BBDAnimationStateChecker | `progress` (float 0~1) |
| BBDCashWinController | BBDCashWinAnimatorBehaviour | `animationProgress` |
| BBDWheelSymbolController | BBDWheelSpinBehaviour | `spinStatus` (SpinStatus) |

---

## Server Bonus Processing

### claimBonus() 분기

| bonus_id | 처리 |
|----------|------|
| 23405/23451~53 (SuperFreeGame) | earnCredit=0, 아무것도 안 함 |
| 23402 (CashPrize) | earnCredit += earn_credit |
| 23404 (Jackpot) | earnCredit += earn_credit, **jackpot_index > 1만 kudo** |
| 23406 (SIS) | earnCredit += earn_credit |

### useTicket() 분기

| ticket_id | 처리 |
|-----------|------|
| 23450 | UseInsTicket() → FreeSpin 10회 |
| 23451~53 | UseSuperBonusTicket(level) → SuperFreeGame 10회 + enterSuperFreeSpin() |

### SIS 정산 로직

```
finalCredit = accumulatedBaseCredit × currentMultiplier
```

- **Jackpot 크레딧은 멀티플라이어 미적용** → 별도 JackpotBonus(23404)로 지급
- Max Win 체크: `(finalCredit + jackpotCredit) / totalBet > 2000` → 재시도
- 잭팟 인덱스 변환: 서버→클라: `3 - jackpotIndex` (GRAND 0→3, MINI 3→0)

### Game Class

```typescript
export default class Game implements
  ISlotGamePromise,
  ISlotGameWithMysterySymbols,
  ISlotGameWithTicketedBonusPromise
```

### Bonus Type/Claim 매핑

| 보너스 | type | claim_type |
|--------|------|------------|
| CashPrize (23402) | NORMAL(1) | ALWAYS(1) |
| Jackpot (23404) | NORMAL(1) | ALWAYS(1) |
| FreeGame (23400) | FREE_SPIN(2) | NONE(3) |
| Retrigger (23401) | FREE_SPIN(2) | NONE(3) |
| InfinityReel (23403) | NORMAL(1) | NONE(3) |
| SuperFreeGame (23405) | FREE_SPIN(2) | ALWAYS(1) |
| SIS (23406) | NORMAL(1) | ALWAYS(1) |

---

## 누락 클래스 (추가 발견)

| 클래스 | 역할 |
|--------|------|
| `BBDSymbolEventHandler` | DefaultSymbolEventHandler 확장, Win→오버레이 라우팅 |
| `BBDCashSymbolController` | Cash UI 표시 (텍스트/잭팟 스프라이트 전환) |
| `BBDWheelSymbolController` | Wheel 스핀/결과 표시, WheelSector, SortingGroup 레이어 |
| `BBDFlyingObjectsHelper` | ObjectPool 기반 플라잉 텍스트 |
| `BBDFlyingTextController` | DirectionalWeightPositionController 경로 설정 |
| `BBDCashCreditUpHelper` | Cash 크레딧 변경 애니메이션 |
| `BBDCashWinController` | animationProgress 필드 (StateMachineBehaviour 연결) |
| `BBDScatterStopHelper` | Scatter 정지 애니메이션 대기 |
| `BBDJackpotUpdater` | 개별 잭팟 크레딧 업데이트 + Lock 표시 |
| `BBDAnimationChecker` | Animator normalizedTime 추적 (WaitUntil/WaitUntilFinish) |
