---
description: "Game ID mapping guide. Game ID ranges, bonus ID rules, file locations."
globs:
  - "**/slot_game/*.ts"
  - "**/slot_game/*.value.ts"
---

# 게임 ID 매핑 가이드

클라이언트-서버 간 게임 ID 매핑과 통계 정보를 제공합니다.

## 게임 ID 범위

| 게임 유형 | 서버 ID 범위 | 클라이언트 위치 |
|----------|-------------|----------------|
| 슬롯 게임 | 1-362 | Contents Group 1/ |
| 비디오 포커 | 100001-100003, 400001-400017 | Contents Group 2/ |
| 케노 | 200001-200005, 500001-500005 | Contents Group 3/ |
| 테이블 게임 | 600001-600004 | Contents Group 4/ |
| 갬블 미니게임 | - | Gamble/ |

## 보너스 ID 규칙

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

**예시**:
```
게임 343 (Supercharge X-treme)
├─ 34300: Free Game
├─ 34301: Pick Bonus
├─ 34302: Lightning
├─ 34303: Multiplier Lightning
└─ 34350: Buy A Bonus
```

## 주요 게임 매핑

| 게임명 | 서버 ID | 서버 파일 | 클라이언트 폴더 |
|--------|---------|-----------|----------------|
| Supercharge X-treme | 343 | supercharge_x_treme.ts | Contents Group 1/Supercharge X-treme |
| Buffalo Burst | 282 | buffalo_burst.ts | Contents Group 1/Buffalo Burst |
| Bingo Ahoy | 308 | bingo_ahoy.ts | Contents Group 1/Bingo Ahoy |
| Pearlinko | 245 | pearlinko.ts | Contents Group 1/Pearlinko |
| Cash Hits Blitz | 285 | cash_hits_blitz.ts | Contents Group 1/Cash Hits Blitz |
| Crystal Nudge | 337 | crystal_nudge.ts | Contents Group 1/Crystal Nudge |

## 클라이언트 폴더 구조

```
Contents Group 1/{GameName}/
├── FSM/Game Logic/
│   ├── GameLogic_FSM.asset
│   └── GameLogic_*.asset
├── Data/
│   ├── Schema.json
│   └── Schema.txt
├── Scripts/
├── Prefabs/
├── Scenes/
└── gameInfo.json
    {
      "gameId": 343,
      "name": "scx",
      "displayName": "Supercharge X-treme"
    }
```

## 통계

| 항목 | 수량 |
|------|------|
| 총 게임 수 | 384개 |
| 슬롯 게임 | 356개 |
| 비디오 포커 | 20개 |
| 케노 | 10개 |
| 테이블 게임 | 4개 |
| FSM Action | 635개 |
| FSM Condition | 74개 |

## 파일 위치

| 타입 | 경로 |
|------|------|
| 서버 게임 로직 | `games/src/slot_game/{game_name}.ts` |
| 서버 인터페이스 | `games/src/slot_game/{game_name}.interface.ts` |
| 서버 값 정의 | `games/src/slot_game/{game_name}.value.ts` |
| 클라이언트 FSM | `client/Assets/Contents/Contents Group 1/{GameName}/FSM/` |
| 클라이언트 Schema | `client/Assets/Contents/Contents Group 1/{GameName}/Data/Schema.json` |
