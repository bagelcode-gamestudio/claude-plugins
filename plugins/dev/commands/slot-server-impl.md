---
description: Start a slot game server logic implementation session
argument-hint: <game-title-or-id>
---

# Slot Server Implementation

## 0. Session Guide

이 커맨드는 슬롯 게임 서버 로직을 **처음부터 끝까지** 구현합니다.

**전체 흐름:**
1. Identify — 게임 식별, 파일 경로 확인
2. Gather — 기획서/수학 문서 읽기, 레퍼런스 게임 탐색
3. Design — 피처 구조 설계, 보너스 ID 할당, CustomData 정의
4. Implement — interface → value → game 순서로 3파일 작성
5. Build & Lint — 컴파일/린트 통과 확인
6. Self-review — 코딩 규칙 체크리스트
7. Simulation file — 시뮬레이션 파일 작성
8. RTP verification — 시뮬레이션 실행, 결과 해석

**Skill Set:**
- **slot-server-guide** — 코딩 규칙, 파일 구조, 보너스 패턴
- **feature-patterns** — 피처별 구현 패턴 (Hold&Spin, Cascade, Nudge 등)
- **bonus-flow** — 보너스 생명주기 (트리거 → 클레임 → 실행 → 종료)
- **math-model** — value 파일 구조, RTP 검증

> 위 스킬만 참조할 것. 다른 스킬이 필요하면 사용자에게 확인 후 확장.

**필요 파일:**
| 타입 | 경로 |
|------|------|
| 게임 로직 | `games/src/slot_game/{game_code}.ts` |
| 인터페이스 | `games/src/slot_game/{game_code}.interface.ts` |
| 값 정의 | `games/src/slot_game/{game_code}.value.ts` |
| 시뮬레이션 | `games/src/slot_simulation/{game_code}.ts` |
| 기획서 | `PLANNING.md` (사용자 제공) |
| 수학 문서 | `MATH.md` (사용자 제공) |

**단계 완료 기준:** 각 단계 완료 시 사용자에게 결과를 보고하고, 다음 단계 진행 여부를 확인할 것.

---

## 1. Identify the game

- Resolve the game title or ID to a file path (`games/src/slot_game/{game_code}.ts`)
- Check if files already exist (new game vs modification)
- Confirm the bonus ID range with user: `gameId * 100 + 0~9`, ticket: `gameId * 100 + 50`

## 2. Gather context

- Read the planning document (PLANNING.md) or user requirements
- Read the math document (MATH.md) if available: reel strips, pay tables, RTP targets
- Identify the feature type: Hold&Spin, Cascade, Nudge, Mystery, Wheel, Infinity Reel, Bingo, etc.
- **Find reference game** — search existing games with similar feature:
  ```bash
  # 예: Hold&Spin 피처를 사용하는 기존 게임 찾기
  grep -rl "link_bonus_state\|remainingSpins\|LINK_BONUS" games/src/slot_game/ --include="*.ts" | head -5
  ```
- Read the reference game's 3 files (`.ts`, `.value.ts`, `.interface.ts`) to understand the pattern

## 3. Design

구현 전 설계를 정리하고 사용자 확인을 받는다.

- **피처 구조** — 보너스 종류, 트리거 조건, 상태 전이 흐름
- **보너스 ID 할당표** — `gameId * 100 + sequence` 규칙에 따라 할당
  ```
  예: 게임 365
  ├─ 36500: Free Spin
  ├─ 36501: Pick Bonus
  ├─ 36502: Lightning
  └─ 36550: Buy A Bonus
  ```
- **CustomData 구조** — 보너스 간 전달할 상태 데이터 정의
- **사용자 확인 후 구현 진입**

## 4. Implement server logic (순서 중요)

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

d. **Debug support**
   - `SLOT_DEBUG.INSTANT_BONUS` (400) 처리 필요 여부 확인
   - 필요시 `modifyDebugParam` + `getGameResult`에 `force_instant_bonus` 로직 추가

## 5. Build & Lint

```bash
cd games
npx tsc --noEmit
npx eslint src/slot_game/{game_code}.ts src/slot_game/{game_code}.value.ts src/slot_game/{game_code}.interface.ts
```

- 타입 에러 0건 확인
- ESLint 에러 0건 확인 (Array<T> 금지, switch default 필수 등)
- 에러 발생 시 수정 후 재실행

## 6. Self-review checklist

- [ ] Reel strips from `.value.ts`, not hardcoded
- [ ] PayTable referenced, not hardcoded payouts
- [ ] Weight totals are arrays (per RTP), not single values
- [ ] `Array<T>` syntax not used (use `T[]` instead)
- [ ] All switch statements have default case
- [ ] 2-space indentation, no duplicate imports
- [ ] File ends with newline
- [ ] Bonus IDs follow `gameId * 100 + sequence` convention
- [ ] Mystery Symbol has fallback logic for invalid symbols

## 7. Simulation file

시뮬레이션 파일을 작성한다: `games/src/slot_simulation/{game_code}.ts`

- 레퍼런스 게임의 시뮬레이션 파일을 참고하여 작성
- 기본 시뮬레이션 (1M spins)
- 필요 시 변형 파일 작성:
  - `{game_code}_ib.ts` — Instant Bonus 시뮬레이션 (100K spins)
  - `{game_code}_sb.ts` — Special Bonus 시뮬레이션 (100K spins)

## 8. RTP verification

```bash
cd games
ts-node src/slot_simulation/{game_code}.ts -- -s 1000000 -t 10
```

- `-s`: Spin count (기본 1M, 정밀 검증 시 100M)
- `-r`: RTP index (0=LOW, 1=MID, 2=HIGH)
- `-t`: Thread count (권장 10)

**결과 해석:**
```
RTP :
total       base        free        bonusonbase bonusonfree
0.960123    0.421234    0.312456    0.112345    0.114088
```

- `total`: target RTP 대비 **±0.5%** 이내인지 확인
- 보너스 트리거 빈도가 설계 의도와 일치하는지 확인
- 이상 시 value 파일의 weight/reel strip을 점검하고 재실행
