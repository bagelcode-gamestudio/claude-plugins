---
description: Slot game math model design guide. Reel strips, probabilities, pay tables, RTP calculation.
globs:
  - "**/*.value.ts"
  - "**/slot_simulation/*.ts"
---

# Math Model Guide

When designing or reviewing slot game math models, follow these patterns:

## Core Responsibilities

- **Reel strip design**: Symbol distribution, weights per reel
- **Probability calculation**: Win probability, bonus trigger rates
- **Pay table structure**: Per-symbol payouts, payline/ways
- **Bonus math**: Expected value, volatility per bonus type
- **RTP calculation**: Total Return to Player and component breakdown

## Value File Structure

```typescript
// {game_code}.value.ts
SYMBOLS = { WILD: 0, H1: 1, H2: 2, ... }
REEL_STRIPS = { ... }      // Reel strip arrays per RTP level
PAY_TABLE = { ... }         // Symbol payout table
BONUS_TRIGGER = { ... }     // Bonus trigger conditions
```

## Key Rules

- Weight totals must be **arrays** (per RTP level), not single values
- Use `reel_sequence_list` from value file, never generate reel strips inline
- Mystery Symbol weights: add fallback when total is 0 (prevents invalid symbol selection)

## RTP Verification

```bash
cd games && ts-node src/slot_simulation/{game_code}.ts -- -s 1000000 -t 10
```

| Check | Criteria |
|-------|----------|
| RTP | Target +/- 0.5% |
| Volatility | Within target range |
| Bonus frequency | Matches design intent |
| Jackpot probability | Matches design probability |

## Math Document Template

```markdown
# [Game Title] - Math Model

## Basic Structure
- Reel layout:
- Paylines/Ways:
- Target RTP:
- Volatility:

## Symbols & Pay Table

## Reel Strips

## Bonus Math
### Bonus 1: [Name]
- Trigger probability:
- Expected spins:
- Expected payout:

## RTP Breakdown
| Component | RTP Contribution |
|-----------|-----------------|
| Base game | X% |
| Bonus 1   | Y% |
| Total     | Z% |
```
