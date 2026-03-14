# Finish Branch Command

현재 브랜치 작업을 완료하고 **Parent 브랜치**에 반영합니다.

## 실행 단계

1. **현재 상태 및 Parent 브랜치 확인**
   - `git status`로 미커밋 변경 확인
   - `docs/tasks/<task-id>.md`에서 **Parent 브랜치** 확인
   - Parent 브랜치가 없으면 기본값 `main` 사용
   - `git log <parent>..HEAD`로 브랜치 커밋 목록 확인

2. **미커밋 변경 처리**
   - 변경사항 있으면 커밋 여부 확인
   - 커밋 또는 stash 처리

3. **완료 체크리스트 확인** ⚠️ 필수

   다음 항목을 확인하고 결과를 출력:

   | # | 항목 | 확인 방법 | 필수 |
   |---|------|----------|------|
   | 1 | 리뷰 문서 존재 | `docs/reviews/<task-id>.md` 파일 확인 | ✅ |
   | 2 | 컴파일 에러 없음 | Unity 콘솔 또는 빌드 확인 | ✅ |
   | 3 | 태스크 문서 갱신 | `docs/tasks/<task-id>.md` 상태 업데이트 | ✅ |
   | 4 | 레트로 문서 (문제 있었을 경우) | `docs/retro/<task-id>.md` | 조건부 |

4. **리뷰 미완료 시 처리**

   리뷰 문서가 없으면:
   ```
   ⚠️ 리뷰가 완료되지 않았습니다.

   선택:
   A) reviewer 호출하여 리뷰 진행
   B) 간략 리뷰로 진행 (mini-review)
   C) 리뷰 건너뛰기 (사유 필요)
   ```

   - **A 선택**: `reviewer` subagent 호출 → 리뷰 완료 후 계속
   - **B 선택**: 변경 사항 요약 + 기본 체크리스트만 확인 → `docs/reviews/<task-id>.md` 생성
   - **C 선택**: 사유 기록 후 진행 (태스크 문서에 "리뷰 스킵: [사유]" 추가)

5. **원격 동기화**
   - `git push -u origin <branch>` 실행

6. **반영 방식 선택** (사용자에게 질문)
   - A) Parent 브랜치(`<parent>`)에 바로 머지 (로컬)
   - B) PR 생성 (GitHub) - base를 `<parent>`로 설정
   - C) 취소

7. **선택에 따른 실행**

   **A) 바로 머지 선택 시:**
   ```
   git checkout <parent>
   git pull origin <parent>
   git merge <branch> --no-ff
   git push origin <parent>
   git branch -d <branch>  # 로컬 브랜치 삭제
   git push origin --delete <branch>  # 원격 브랜치 삭제 (확인 후)
   ```

   **B) PR 생성 선택 시:**
   ```
   gh pr create --base <parent> --fill
   ```
   PR URL 출력

## Mini-Review 템플릿

리뷰 건너뛰기(B) 선택 시 자동 생성:

```markdown
# [task-id] Mini-Review

## 요약
- 상태: 간략 리뷰 완료
- 날짜: [날짜]

## 변경 사항
[git diff --stat 결과]

## 기본 체크리스트
- [ ] 컴파일 에러 없음
- [ ] 주요 기능 동작 확인

## 비고
간략 리뷰로 진행됨
```

## 주의사항

- 통합 브랜치(main, develop 등)에서 실행 시 경고 후 중단
- 머지 전 Parent 브랜치 최신화 필수
- 브랜치 삭제 전 확인 받기
- **리뷰 문서 없이 머지 시 경고 표시**
