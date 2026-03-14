# Design Only Workflow

설계 문서만 작성하고 구현은 나중에 하는 워크플로우입니다.

## 흐름

```
planner → critic → architect → critic
```

## 사용 조건

| # | 상황 |
|---|------|
| 1 | 설계 검토가 먼저 필요 |
| 2 | 팀 논의 전 문서화 |
| 3 | 구현 일정이 미정 |
| 4 | 기술 스파이크 |

## 단계별 상세

### 1. Planner (필수)

**목적:** 요구사항 정리

**산출물:**
- 질문셋
- `docs/tasks/<task-id>.md`

**다음 단계:** critic

---

### 2. Critic #1 (필수)

**목적:** 요구사항 검토

**검토 항목:**
- 범위 적절성
- 누락된 고려사항
- 실현 가능성

**다음 단계:** architect

---

### 3. Architect (필수)

**목적:** 상세 설계

**산출물:**
- `docs/design/<task-id>.md`
  - 구조 설계
  - 인터페이스 정의
  - 의존성 분석
  - 구현 가이드

**다음 단계:** critic

---

### 4. Critic #2 (필수)

**목적:** 설계 최종 검토

**산출물:**
- `docs/critic/<task-id>.md`
  - 리스크 분석
  - 대안 검토
  - 권고사항

---

## 예상 소요

4단계

## 산출물 요약

| 문서 | 내용 |
|------|------|
| `docs/tasks/<task-id>.md` | 확정된 스펙 |
| `docs/design/<task-id>.md` | 상세 설계 |
| `docs/critic/<task-id>.md` | 리스크/대안 분석 |

## 구현 진행 시

나중에 구현할 때:

```
/workflow quick-fix   # 설계대로 간단히 구현
또는
executor → reviewer   # 직접 시작
```

이미 설계가 있으므로 planner/architect 단계 스킵 가능
