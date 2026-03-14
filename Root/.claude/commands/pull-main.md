# Pull Main Command

main 브랜치의 최신 변경사항을 현재 브랜치로 가져옵니다.

## 실행 단계

1. **현재 상태 확인**
   - `git branch`로 현재 브랜치 확인
   - main 브랜치인 경우 경고 후 중단
   - `git status`로 uncommitted 변경사항 확인

2. **uncommitted 변경이 있으면**
   - 변경 내용 요약 표시
   - 먼저 커밋하거나 stash할지 사용자에게 질문
   - 사용자 선택에 따라 처리

3. **main 브랜치 최신화**
   - `git fetch origin main` 실행

4. **현재 브랜치에 머지**
   - `git merge origin/main` 실행
   - 충돌 발생 시 충돌 파일 목록 표시
   - 충돌 해결 가이드 제공

5. **결과 보고**
   - 머지 성공 시 가져온 커밋 수 표시
   - 이미 최신인 경우 "Already up to date" 메시지

## 주의사항

- main 브랜치에서는 실행 불가
- uncommitted 변경이 있으면 먼저 처리 필요
- 충돌 발생 시 수동 해결 후 `git commit` 필요
