# Workflow Command

워크플로우 프리셋을 로드하고 실행합니다.

## 사용법

```
/workflow <preset-name>
```

## 사용 가능한 프리셋

| # | 프리셋 | 설명 | 흐름 |
|---|--------|------|------|
| 1 | `full-feature` | 새 기능 전체 개발 | planner → critic → architect → critic → executor → reviewer |
| 2 | `quick-fix` | 간단한 버그 수정 | executor → reviewer |
| 3 | `exploration` | 코드베이스 탐색/분석 | architect만 |
| 4 | `refactor` | 리팩토링 작업 | critic → architect → executor → reviewer |
| 5 | `design-only` | 설계 문서만 작성 | planner → critic → architect → critic |

## 실행 단계

### 1. 프리셋 로드

`.claude/workflows/<preset-name>.md` 파일 읽기

### 2. 사용자 확인

```markdown
## 🔄 워크플로우: [preset-name]

**흐름:** planner → critic → architect → executor → reviewer

**포함 단계:**
1. planner: 요구사항 정리, 질문셋 작성
2. critic: 질문셋 검토
3. architect: 설계 문서 작성
4. critic: 설계 검토
5. executor: 구현
6. reviewer: 리뷰 및 승인

이 워크플로우로 시작할까요? (y/n)
```

### 3. 첫 단계 실행

승인 시 첫 번째 agent 자동 호출

### 4. 단계별 진행

각 단계 완료 시:
- 표준화된 헤더로 상태 출력
- 다음 단계 자동 제안
- 사용자 승인 후 다음 agent 호출

## 프리셋 없이 실행 시

```
/workflow
```

사용 가능한 프리셋 목록 표시

## 주의사항

- 프리셋 파일이 없으면 에러 메시지 출력
- 중간에 워크플로우 중단 가능 (사용자가 "중단" 입력)
- 각 단계에서 critic은 선택적 스킵 가능 (간단한 작업일 경우)
