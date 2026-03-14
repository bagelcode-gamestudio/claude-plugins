---
description: Slot server game logic implementation guide. RNG, win calculation, bonus logic, API response design.
globs:
  - "**/slot_game/*.ts"
  - "**/slot_game/*.value.ts"
  - "**/slot_game/*.interface.ts"
---

# Slot Server Dev Guide

When implementing or modifying slot game server logic, follow these patterns:

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

## Core Principles

- **Server-Authoritative RNG**: All random numbers generated on server
- **Programmed Win/Loss**: RTP managed via `getSlotGameResultWithProgrammedLoss()`
- **Bonus ID**: `gameId * 100 + sequence` (e.g., 343 -> 34300, 34301...)
- **Ticket Bonus**: `gameId * 100 + 50` (e.g., 343 -> 34350)

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

## Bonus Creation Pattern

```typescript
bonusResultList.push(_GenerateBonusResult(
  bonusType.NEW_BONUS.BONUS_ID,   // gameId * 100 + sequence
  BonusResultType.FREE_SPIN,       // NORMAL=1, FREE_SPIN=2, SELECTABLE=3, RESPIN=4, PARTY=5
  0,                                // earnCredit (0 for free spin)
  types.CLAIM_TYPE.ALWAYS,          // ALWAYS=1, OPTIONAL=2, NONE=3
  { added_spin_count: 8 }           // bonus-specific response
));
```

## Schema Sync Reminder

When modifying `.interface.ts`, remind the user about client Schema.json sync.
