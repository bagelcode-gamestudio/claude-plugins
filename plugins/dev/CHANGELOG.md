# Changelog

## 0.2.0 — 2026-03-16

## Overview

`/dev:slot-server-impl` 커맨드의 실전 투입을 위한 최적화 및 기능 확장.
컨텍스트 노이즈 47% 감소, 커맨드 절차 6단계 → 9단계(0~8)로 확장.

---

## Glob Optimization

불필요한 스킬 자동 트리거를 제거하여 `**/slot_game/*.ts` 작업 시 로드되는 스킬을 8개 → 3개로 축소.

- `spin-flow` — `**/slot_game/*.ts` 제거, `**/LOGIC/routes/*.ts` 추가
- `data-systems` — `**/slot_game/*.ts` 제거 (interface 전용으로 한정)
- `blackboard-path` — 서버 glob 전체 제거, 클라이언트 glob(`*Controller*.cs`, `*Module*.cs`, `FSM/**/*.asset`)으로 교체
- `contents-event` — `**/slot_game/*.ts` 제거

## Skill Consolidation

- `game-mapping` → `slot-server-guide`에 병합 (보너스 ID 시퀀스 규칙, 파일 경로 테이블)
- `game-mapping` 스킬 삭제 — 12개 → 11개

## Skill Visibility

- 전 스킬에 `user-invocable: false` 적용
- 슬래시 메뉴에 스킬이 노출되지 않도록 설정 (CLI 정상 작동, VSCode 미지원 — 제품 버그)

## Command Enhancement — `slot-server-impl`

기존 6단계에서 9단계(0~8)로 재구성.

| 단계 | 이름 | 상태 |
|------|------|------|
| 0 | Session Guide | **신규** |
| 1 | Identify the game | 유지 |
| 2 | Gather context | 유지 |
| 3 | Design | **신규** |
| 4 | Implement server logic | 유지 (Debug support 통합) |
| 5 | Build & Lint | **신규** |
| 6 | Self-review checklist | 유지 |
| 7 | Simulation file | **신규** |
| 8 | RTP verification | **신규** (기존 run-sim 연결 → 실행+해석으로 확장) |

### 신규 단계 상세

- **Step 0 (Session Guide)** — 전체 흐름 요약, 스킬 셋 선언, 필요 파일 경로, 단계별 완료 기준
- **Step 3 (Design)** — 피처 구조 정리, 보너스 ID 할당표, CustomData 설계, 사용자 확인 후 구현 진입
- **Step 5 (Build & Lint)** — `tsc --noEmit` + `eslint` 실행, 에러 0건 확인
- **Step 7 (Simulation file)** — `slot_simulation/{game_code}.ts` 작성, instant/special bonus 변형
- **Step 8 (RTP verification)** — 시뮬레이션 실행, 결과 해석 (target RTP ±0.5%), 이상 시 원인 분석

## Plugin Metadata

- `marketplace.json`, `plugin.json` description을 범용 개발 도구로 수정
- "Slot game server development tools" → "Development tools"

## Dev Plugin Conventions

- `plugins/dev/CLAUDE.md` 신규 작성
- Skill 작성 규약: `user-invocable: false` 필수, globs 최소화
- Command 작성 규약: Skill Set 섹션 필수, Self-review checklist 포함
- Versioning 규약: 커밋 시 `plugin.json` 버전 + `CHANGELOG.md` 함께 업데이트 (semver)

---

## Impact

| 지표 | Before | After |
|------|--------|-------|
| `slot_game/*.ts` 트리거 스킬 | 8개 (~2,400줄) | 3개 (~1,290줄) |
| 컨텍스트 감소 | — | ~47% |
| 스킬 수 | 12개 | 11개 |
| 커맨드 커버리지 | 학습→구현→리뷰 | 학습→설계→구현→검증→시뮬 |
