# Roar and Flame (RNF) - Game Reference

## 기본 정보

| 항목 | 값 |
|------|-----|
| **Game ID** | 321 |
| **game_title** | `rnf` |
| **Display Name** | Roar And Flame |
| **Base Bet** | 100 |
| **릴 구성** | 3x5 (베이스), 8x5 (보너스 확장) |
| **Ways** | 243 Ways |
| **모드** | 세로 모드 |
| **RTP IDs** | LOW, MIDDLE, HIGH |
| **Deck** | 사용 (is_deck_used: true) |
| **Permanent State** | 있음 (pot 적립 추적) |

## Bonus ID Map

| Bonus | ID | 설명 |
|-------|----|------|
| CASH_WIN | 32100 | 캐시 심볼 히트 보상 |
| ROAR_BONUS | 32101 | 타이거 보너스 (3x5, 리스핀) |
| FLAME_BONUS | 32102 | 피닉스 보너스 (8x5, 1회 스핀) |
| ROAR_AND_FLAME_BONUS | 32103 | 타이거&피닉스 보너스 (8x5+그랜드, 리스핀) |
| JACKPOT | 32104 | 잭팟 보상 |
| CASH_BONUS | 32105 | 랜덤 캐시 보너스 |
| INSTANT_BONUS (티켓) | 32150 | 인스턴트 보너스 (→ 32103 동일) |
| SUPER_ROAR_BONUS | 32106 | 슈퍼 타이거 보너스 (L1, Concatenation+배수) |
| SUPER_FLAME_BONUS | 32107 | 슈퍼 드래곤 보너스 (L2, Concatenation) |
| SUPER_ROAR_AND_FLAME_BONUS | 32108 | 슈퍼 합체 보너스 (L3, Concatenation+배수) |
| SUPER_BONUS_TICKET_L1 (티켓) | 32151 | → 32106 |
| SUPER_BONUS_TICKET_L2 (티켓) | 32152 | → 32107 |
| SUPER_BONUS_TICKET_L3 (티켓) | 32153 | → 32108 |

## 심볼 Enum

```typescript
// roar_and_flame.interface.ts
export const enum SYMBOLS {
  BONUS_1 = 0,  // Tiger
  BONUS_2 = 1,  // Phoenix
  CASH = 2,
  H1 = 3,       // Turtle
  M1 = 4,       // Lantern
  M2 = 5,       // Lotus Flower
  M3 = 6,       // Invitation
  M4 = 7,       // Coin
  L1 = 8,       // A
  L2 = 9,       // K
  L3 = 10,      // Q
  L4 = 11,      // J
}

export const enum CASH_SYMBOL_TYPE {
  CASH = 0,      // 숫자 프라이즈
  UPGRADE = 1,   // 업그레이드
  JACKPOT = 2,   // 잭팟 (Mega/Major/Minor/Mini)
}

export const enum BONUS_SPOT_TYPE {
  BLANK = 0,
  LOW = 1,
  HIGH = 2,
  LOCKED = 3,    // 캐시 심볼 고정됨
}
```

## 파일 위치

### 서버 (games 레포)
| 파일 | 경로 |
|------|------|
| 메인 로직 | `games/src/slot_game/roar_and_flame.ts` |
| 인터페이스 | `games/src/slot_game/roar_and_flame.interface.ts` |
| 밸류(릴/보너스 설정) | `games/src/slot_game/roar_and_flame.value.ts` |

### 클라이언트 (client 레포)

| 파일 | 경로 |
|------|------|
| 루트 | `client/Assets/Contents/Contents Group 1/Roar And Flame/` |
| gameInfo | `Roar And Flame/gameInfo.json` |
| FSM 메인 | `Roar And Flame/FSM/Game Logic/GameLogic_FSM.asset` |

#### 피처 컨트롤러 스크립트
| 파일 | 용도 |
|------|------|
| `RNFRoarBonusController.cs` | 타이거(Roar) 보너스 컨트롤러 |
| `RNFFlameBonusController.cs` | 피닉스(Flame) 보너스 컨트롤러 |
| `RNFRoarAndFlameController.cs` | 타이거&피닉스 합체 보너스 컨트롤러 |
| `RNFCashWinController.cs` | 캐시 심볼 히트 연출 |
| `RNFCashBonusController.cs` | 랜덤 캐시 보너스 연출 |
| `RNFBonusCollectController.cs` | 보너스 적립 연출 |
| `RNFBonusPotAdmin.cs` | 보너스 Pot UI 관리 |
| `RNFCashPotAdmin.cs` | 캐시 Pot UI 관리 |
| `RNFCashHighlightController.cs` | 캐시 하이라이트 연출 |
| `RNFMultiplierBoardAdmin.cs` | 배수 보드 UI |
| `RNFPreEventController.cs` | 프리 이벤트 연출 |
| `RNFSuperBonusBaseController.cs` | 슈퍼보너스 공통 베이스 (리스핀, 캐시히트, 잭팟) |
| `RNFSuperBonusRoarController.cs` | 슈퍼 타이거 보너스 L1 (배수) |
| `RNFSuperBonusFlameController.cs` | 슈퍼 드래곤 보너스 L2 (캐시밀기) |
| `RNFSuperBonusCombinedController.cs` | 슈퍼 합체 보너스 L3 (밀기+배수) |

#### FSM/BT 구조
| BT | 용도 |
|----|------|
| `GameLogic_Roar Bonus BT` | 타이거 보너스 흐름 |
| `GameLogic_Flame Bonus BT` | 피닉스 보너스 흐름 |
| `GameLogic_Roar And Flame Bonus BT` | 합체 보너스 흐름 |
| `GameLogic_Cash Win BT` | 캐시 히트 흐름 |
| `GameLogic_Cash Bonus BT` | 랜덤 캐시 보너스 흐름 |
| `GameLogic_Collect Bonus BT` | 적립 보너스 흐름 |
| `GameLogic_Win Way BT` | 웨이 윈 정산 |
| `GameLogic_Check Big Win_BT` | 빅윈 체크 |
| `GameLogic_Check Pre Event BT` | 프리 이벤트 체크 |
| `GameLogic_Super Tiger Bonus BT` | 슈퍼 타이거 보너스 진입 (Buy A Bonus BT에서 호출) |
| `GameLogic_Super Dragon Bonus BT` | 슈퍼 드래곤 보너스 진입 |
| `GameLogic_Super Combined Bonus BT` | 슈퍼 합체 보너스 진입 |
| `GameLogic_Check Buy A Bonus_BT` | 바이어보너스 체크 (IB+SB Optional->SubTree) |

## 핵심 로직 구조

### 보너스 트리거 조건
```
- Tiger 3개 이상 → ROAR_BONUS (32101)
- Phoenix 3개 이상 → FLAME_BONUS (32102)
- Tiger + Phoenix = 5개 → ROAR_AND_FLAME_BONUS (32103)
- 보너스 심볼 1개 이상 + 랜덤 트리거 → 해당 보너스 랜덤 진입
```

### 캐시 심볼 히트 판정
- 동일 로우에 인접한 캐시 심볼 2개 이상 → 히트
- 히트된 심볼들이 프레임으로 합쳐짐
- 업그레이드: 가장 왼쪽 심볼 숫자 +1씩 증가
- 잭팟: 해당 자리 0으로, 잭팟 팝업

### Roar 보너스 (3x5)
- 리셋 리스핀 3회
- 캐시 심볼 등장 시 고정 + 리스핀 리셋
- 종료 후 히트된 캐시 심볼에 프라이즈/잭팟/업그레이드 결정

### Flame 보너스 (8x5)
- 스핀 1회만 제공
- 종료 후 히트된 캐시 심볼에 피처 결정

### Roar & Flame 보너스 (8x5+그랜드)
- 3x5에서 시작, 캐시 심볼 누적으로 로우 확장
- 확장 필요 캐시 개수: 6, 9, 14, 19, 25, 32(그랜드)
- 8x5 전체 확장 후 32개 모으면 그랜드 잭팟 영역 활성

### 잭팟 구성
| 잭팟 | 기본금액(TB) | Progressive |
|------|-------------|-------------|
| Grand | 500 | 0.002 |
| Mega | 100 | 0.002 |
| Major | 50 | 0.003 |
| Minor | 10 | 0.003 |
| Mini | 5 | 0.005 |

## GameProgress (영구 상태)

```typescript
interface IRNFProgressData {
  pot_collect_count: number;   // 보너스 Pot 적립 횟수
  roar_pot_level: number;      // 타이거 Pot 레벨
  flame_pot_level: number;     // 피닉스 Pot 레벨
}
```

## 릴 구성

| Index | 용도 |
|-------|------|
| 0 | 베이스 게임 릴 (일반) |
| 1 | 베이스 게임 릴 (캐시 많은) |
| 2 | 베이스 게임 릴 (보너스 많은) |
| 3 | Roar 보너스 페이크 릴 (15개) |
| 4 | Flame 보너스 페이크 릴 (40개) |

## Paytable (Base Bet 100 기준)

| 심볼 | 3 hit | 4 hit | 5 hit |
|------|-------|-------|-------|
| H1 (Turtle) | 30 | 80 | 250 |
| M1 (Lantern) | 20 | 50 | 150 |
| M2 (Lotus) | 20 | 50 | 150 |
| M3 (Invitation) | 12 | 40 | 100 |
| M4 (Coin) | 12 | 40 | 100 |
| L1-L4 | 8 | 15 | 40 |

## Gotchas / 주의사항

1. **client 레포**: 클라이언트가 `client/` 레포에 있음
2. **캐시 히트 판정**: 동일 로우 + 인접 2개 이상이 조건 (단순 개수가 아님)
3. **히트된 가장 왼쪽 심볼**: 잭팟/업그레이드 등장 불가 (예외 처리)
4. **earn_credit 0 방지**: Roar&Flame 보너스는 earn_credit==0이면 재시도 (while 루프)
5. **릴 적립 3개 조건**: 캐시 심볼 3개 이상 등장 시 상단 UI 적립 연출
6. **캐시 보너스 전제조건**: 릴에 캐시 1개 이상 + 일반 페이아웃 없음 + 보너스 트리거 없음
7. **SB는 client 레포**: `client_sub`가 아닌 `client/` 레포에서 작업 (기존 IB/기본 보너스는 `client_sub`)
8. **SB 티켓 bet_credit 필수**: useTicket 응답의 bonusResult.result에 `bet_credit: baseBet + extraBet` 반드시 포함
9. **SB vs IB 잭팟 필드명**: IB는 `jackpot_list`, SB는 `jackpot_info_list` 사용
10. **SB OnPlay 흐름**: IB는 즉시 결과, SB는 free_spin 1회 -> Spin API 호출 -> 실제 결과 수신
11. **Spin BBParameter 수동 초기화**: FeatureController에서 Spin action 직접 호출 시 betCredit/extraBetCredit/customData 반드시 설정

## SB 연출 스펙

### 피닉스 밀기 (L2/L3 공통)

- **방향**: 좌→우, row 단위 (위→아래 순차)
- **도착지**: col 4부터 역순 채움 (5릴 밖으로 밀리지 않음)
- **피닉스 이펙트 위치**: col 2 (중앙 릴) 좌표
- **타이밍**: 10프레임(피닉스 진입) → col당 15프레임(124px 이동) × 5칸
- **블랭크 처리**: 블랭크는 이동하지 않음, 피닉스만 지나감
- **정산**: 밀기 완료 후 concatenatedValuesPerRow 기준으로 정산

### 호랑이 배수 (L1/L3)

- L1: 리스핀 후 배수만
- L3: 피닉스 밀기 → (phaseBridgeDelay) → 호랑이 배수
- multiplierPerRow[row] > 1인 row에만 적용
