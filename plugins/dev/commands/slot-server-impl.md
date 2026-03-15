---
description: Start a slot game server logic implementation session
argument-hint: <game-title-or-id>
---

# Slot Server Implementation

Implement server-side slot game logic in the games repository.

## Skill Set

이 세션에서 사용할 스킬:
- **slot-server-guide** — 코딩 규칙, 파일 구조, 보너스 패턴
- **feature-patterns** — 피처별 구현 패턴 (Hold&Spin, Cascade, Nudge 등)
- **bonus-flow** — 보너스 생명주기 (트리거 → 클레임 → 실행 → 종료)
- **math-model** — value 파일 구조, RTP 검증

> 위 스킬만 참조할 것. 다른 스킬이 필요하면 사용자에게 확인 후 확장.

## Instructions

1. **Identify the game**
   - Resolve the game title or ID to a file path (`games/src/slot_game/{game_code}.ts`)
   - Check if files already exist (new game vs modification)
   - Confirm the bonus ID range with user: `gameId * 100 + 0~9`, ticket: `gameId * 100 + 50`

2. **Gather context**
   - Read the planning document (PLANNING.md) or user requirements
   - Read the math document (MATH.md) if available: reel strips, pay tables, RTP targets
   - Identify the feature type: Hold&Spin, Cascade, Nudge, Mystery, Wheel, Infinity Reel, Bingo, etc.
   - **Find reference game** — search existing games with similar feature:
     ```bash
     # 예: Hold&Spin 피처를 사용하는 기존 게임 찾기
     grep -rl "link_bonus_state\|remainingSpins\|LINK_BONUS" games/src/slot_game/ --include="*.ts" | head -5
     ```
   - Read the reference game's 3 files (`.ts`, `.value.ts`, `.interface.ts`) to understand the pattern

3. **Implement server logic** (순서 중요)
   a. **interface 먼저**: `{game_code}.interface.ts`
      - CustomData 인터페이스 정의
      - 보너스 응답 인터페이스 정의
   b. **value 다음**: `{game_code}.value.ts`
      - MATH.md의 릴스트립/페이테이블 변환
      - RTP별 배열 구조 (`reel_sequence_list`, `PayTable`, weight totals)
   c. **game 마지막**: `{game_code}.ts`
      - `getGameResult` 구현 (레퍼런스 게임 패턴 따르기)
      - `_GenerateBonusResult()`로 보너스 생성
      - 모든 RNG는 서버 권한

4. **Debug support** (시뮬레이션용)
   - `SLOT_DEBUG.INSTANT_BONUS` (400) 처리 필요 여부 확인
   - 필요시 `modifyDebugParam` + `getGameResult`에 `force_instant_bonus` 로직 추가

5. **Self-review checklist**
   - [ ] Reel strips from `.value.ts`, not hardcoded
   - [ ] PayTable referenced, not hardcoded payouts
   - [ ] Weight totals are arrays (per RTP), not single values
   - [ ] `Array<T>` syntax not used (use `T[]` instead)
   - [ ] All switch statements have default case
   - [ ] 2-space indentation, no duplicate imports
   - [ ] File ends with newline
   - [ ] Bonus IDs follow `gameId * 100 + sequence` convention
   - [ ] Mystery Symbol has fallback logic for invalid symbols

6. **Simulation** (if requested)
   - Use `/dev:run-sim` for RTP verification
