# Context Command

현재 작업 컨텍스트를 요약해서 출력합니다.

## 실행 단계

### 1. Git 상태 확인
```bash
git branch --show-current
git status --short
git log --oneline -5
```

### 2. 현재 task-id 파악
- 브랜치명에서 추출: `agent/<task-id>` → `<task-id>`
- 없으면 "task-id 없음" 표시

### 3. 관련 문서 확인
task-id가 있으면:
```
docs/tasks/<task-id>.md     - 스펙
docs/design/<task-id>.md    - 설계
docs/reviews/<task-id>.md   - 리뷰
docs/retro/<task-id>.md     - 레트로
docs/critic/<task-id>.md    - 크리틱
```

### 4. 출력 형식

```markdown
## 📍 현재 컨텍스트

| 항목 | 값 |
|------|-----|
| 브랜치 | `agent/H1-2025-12-05-ds-example` |
| task-id | `H1-2025-12-05-ds-example` |
| 미커밋 변경 | 3개 파일 |
| 최근 커밋 | `feat: add lobby UI` |

## 📄 관련 문서

| 문서 | 상태 |
|------|------|
| 스펙 | ✅ 존재 |
| 설계 | ✅ 존재 |
| 리뷰 | ❌ 없음 |
| 레트로 | ❌ 없음 |
| 크리틱 | ❌ 없음 |

## 📝 스펙 요약
[스펙 문서의 ## 요약 섹션 내용]

## 🔄 권장 다음 단계
- [ ] 설계 완료 → executor 호출
- [ ] 또는 critic 검토 권장
```

## 주의사항

- main 브랜치면 task-id 없음으로 표시
- 문서가 없어도 에러 아님, 상태만 표시
- 스펙 요약은 있을 때만 출력
