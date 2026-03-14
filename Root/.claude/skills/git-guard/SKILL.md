---
name: git-guard
description: Git 커밋 시 안전 규칙을 확인하는 가이드. 커밋이나 브랜치 작업 시 자동으로 사용됨.
---

# Git Guard 스킬

Git 작업 시 안전 규칙을 확인하고 실수를 방지합니다.

## 언제 사용하나요?

- 커밋할 때
- 브랜치 변경/생성할 때
- 푸시할 때
- 머지할 때

---

## 1. 브랜치 규칙 확인

### 커밋 전 확인

```bash
git branch --show-current
```

| # | 상황 | 동작 |
|---|------|------|
| 1 | main/master 브랜치 | ⚠️ 경고 + 확인 요청 |
| 2 | agent/* 브랜치 | ✅ 정상 진행 |
| 3 | 그 외 브랜치 | ⚠️ 브랜치 규칙 안내 |

### main 브랜치 보호

main/master에서 직접 커밋 시도 시:

```markdown
⚠️ main 브랜치 직접 커밋 감지

현재 main 브랜치에 있습니다.
직접 커밋하려면 명시적 확인이 필요합니다.

권장: `agent/<task-id>` 브랜치에서 작업 후 머지

계속하시겠습니까? (y/n)
```

---

## 2. 커밋 메시지 컨벤션

### 형식 확인

```
<type>(<scope>): <subject>

<body>

task-id: <task-id>
```

### 타입 검증

| 타입 | 용도 |
|------|------|
| feat | 새 기능 |
| fix | 버그 수정 |
| refactor | 리팩토링 |
| test | 테스트 |
| docs | 문서 |
| chore | 기타 |

### task-id 확인

브랜치가 `agent/<task-id>` 형식이면:
- 커밋 메시지에 task-id 포함 권장
- 없으면 자동 추가 제안

---

## 3. 위험한 작업 방지

### 금지 명령어

| # | 명령어 | 위험도 | 대응 |
|---|--------|--------|------|
| 1 | `git push --force` | 🔴 | 차단 + 경고 |
| 2 | `git reset --hard` | 🔴 | 확인 요청 |
| 3 | `git clean -fd` | 🟡 | 확인 요청 |
| 4 | `git rebase -i` | 🟡 | 대화형 불가 안내 |

### Force Push 차단

```markdown
🚫 Force Push 감지

`git push --force`는 위험한 작업입니다.
다른 사람의 커밋을 덮어쓸 수 있습니다.

정말 필요한 경우 명시적으로 확인해주세요.
```

---

## 4. 커밋 전 체크리스트

### 자동 확인 항목

| # | 항목 | 확인 방법 |
|---|------|-----------|
| 1 | 민감 정보 없음 | `.env`, `credentials` 패턴 검색 |
| 2 | 디버그 코드 없음 | `console.log`, `Debug.Log` 검색 |
| 3 | TODO 확인 | `TODO`, `FIXME` 검색 (경고만) |

### 경고 시 동작

```markdown
⚠️ 커밋 전 확인 필요

발견된 항목:
- [ ] `Assets/Scripts/Test.cs:15` - Debug.Log 발견
- [ ] `.env.local` - 민감 파일 포함됨

계속 커밋하시겠습니까? (y/n)
```

---

## 5. 리포트 형식

커밋 완료 후:

```markdown
## ✅ Git 커밋 완료

| 항목 | 값 |
|------|-----|
| 브랜치 | `agent/H1-2025-12-05-ds-example` |
| 커밋 | `abc1234` |
| 메시지 | `feat(lobby): add login button` |
| 변경 파일 | 3개 |

다음: `git push` 또는 `/sync`
```
