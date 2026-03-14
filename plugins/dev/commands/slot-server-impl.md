---
description: Start a slot game server logic implementation session
argument-hint: <game-title-or-id>
---

# Slot Server Implementation

Implement server-side slot game logic in the games repository.

## Instructions

1. **Identify the game**
   - Resolve the game title or ID to a file path (`games/src/slot_game/{game_code}.ts`)
   - Check if files already exist (new game vs modification)
   - Confirm the bonus ID range: `gameId * 100 + 0~9`, ticket: `gameId * 100 + 50`

2. **Gather context**
   - Read the planning document (PLANNING.md) or user requirements
   - Read the math document (MATH.md) if available: reel strips, pay tables, RTP targets
   - Identify the feature type: Hold&Spin, Cascade, Nudge, Mystery, Wheel, Infinity Reel, Bingo, etc.
   - Check existing similar games for reference patterns

3. **Implement server logic** (3 files per game)
   - `{game_code}.ts` — Main game logic (RNG, grid, win calculation, bonus triggers)
   - `{game_code}.value.ts` — Reel strips, pay tables, probability constants
   - `{game_code}.interface.ts` — TypeScript types for CustomData, bonus responses
   - Follow coding rules: use `reel_sequence_list` from value file, never hardcode payouts
   - Use `_GenerateBonusResult()` for bonus creation
   - All RNG must be server-authoritative

4. **Self-review checklist**
   - [ ] Reel strips from `.value.ts`, not hardcoded
   - [ ] PayTable referenced, not hardcoded payouts
   - [ ] Weight totals are arrays (per RTP), not single values
   - [ ] `Array<T>` syntax not used (use `T[]` instead)
   - [ ] All switch statements have default case
   - [ ] 2-space indentation, no duplicate imports
   - [ ] File ends with newline
   - [ ] Bonus IDs follow `gameId * 100 + sequence` convention
   - [ ] Mystery Symbol has fallback logic for invalid symbols

5. **Simulation** (if requested)
   - Point to `/dev:run-sim` for RTP verification
