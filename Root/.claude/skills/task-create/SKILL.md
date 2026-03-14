---
name: task-create
description: task-id를 생성하고 docs/tasks/ 에 태스크 문서를 생성하는 가이드. 새 작업을 시작하거나 브랜치를 생성할 때 자동으로 사용됨.
---

# 태스크 생성 스킬

새 작업 시작 시 task-id를 생성하고 태스크 문서를 만듭니다.

## 언제 사용하나요?

- 새 작업을 시작할 때
- 브랜치를 생성해야 할 때
- 요구사항이 확정되어 실행 단계로 넘어갈 때

## task-id 형식

```
{priority}-{date}-{initials}-{keyword}
```

### 구성 요소

| 요소 | 설명 | 예시 |
|------|------|------|
| priority | H(High), M(Mid), L(Low) + 숫자 | H1, M2, L1 |
| date | YYYY-MM-DD (**환경 정보의 Today's date 사용**) | 2025-12-03 |
| initials | 담당자 이니셜 (자동 추출 가능) | ds, jk, mh |
| keyword | 핵심 키워드 (kebab-case) | agents-workflow |

### 이니셜 결정 규칙

1. **사용자 지정**: 사용자가 명시적으로 지정한 경우 해당 이니셜 사용
2. **자동 추출**: 미지정 시 `git config user.name`에서 추출
   - 한글 이름: 성 제외, 이름 첫 글자들 로마자화 (예: "신동호" → "dh")
   - 영문 이름: first/last name 첫 글자 (예: "John Doe" → "jd")

```bash
# 이니셜 확인 명령
git config user.name
```

> ⚠️ **주의:** 이니셜을 추측하지 말고, 불확실하면 사용자에게 확인하세요.

## 날짜 결정

task-id의 날짜는 반드시 **환경 정보(env)에서 제공된 `Today's date`**를 사용합니다.

> ⚠️ **주의:**
> - 대화 내용에서 날짜를 추측하지 마세요
> - 이전 예시의 날짜를 재사용하지 마세요
> - 항상 시스템이 제공한 오늘 날짜를 사용하세요

### 예시

```
H1-2025-12-03-ds-tentacle-ai
M2-2025-12-03-jk-lobby-ui
L1-2025-12-03-mh-docs-update
```

## 태스크 문서 템플릿

`docs/tasks/<task-id>.md` 파일 생성:

```markdown
# [task-id]

## 요약
[한 줄 요약]

## 배경
[왜 이 작업이 필요한지]

## 요구사항
- [ ] 요구사항 1
- [ ] 요구사항 2
- [ ] 요구사항 3

## 스펙 (질문셋 결과)
[question-set 스킬로 확정된 스펙]

## 관련 문서
- 설계: `docs/design/<task-id>.md`
- 리뷰: `docs/reviews/<task-id>.md`

## 브랜치
- **Parent**: `<parent-branch>` (이 브랜치에서 분기됨)
- **Current**: `agent/<task-id>`

## 담당
- Planner: [이름]
- Executor: [이름]
- Reviewer: [이름]

## 상태
- [ ] 계획
- [ ] 설계
- [ ] 구현
- [ ] 리뷰
- [ ] 완료
```

## 브랜치 생성

태스크 문서 생성 후 브랜치 생성:

```bash
git checkout -b agent/<task-id>
```

## 워크플로우

1. 요구사항 확정 (question-set 스킬 사용)
2. task-id 생성
3. `docs/tasks/<task-id>.md` 생성
4. `agent/<task-id>` 브랜치 생성
5. 작업 시작

## 완료 조건

- task-id가 형식에 맞게 생성됨
- `docs/tasks/<task-id>.md` 파일 존재
- `agent/<task-id>` 브랜치 생성됨
