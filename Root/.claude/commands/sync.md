# GitHub Sync Command

현재 브랜치의 변경사항을 GitHub에 동기화합니다.

## 실행 단계

1. **현재 상태 확인**
   - `git status`로 변경사항 확인
   - `git branch`로 현재 브랜치 확인

2. **커밋되지 않은 변경이 있으면**
   - 변경 내용 요약 (테이블 형식)
   - 커밋 메시지 제안 → 사용자 승인 대기
   - 승인 후 `git add .` 및 `git commit`

3. **원격에 푸시**
   - `git push -u origin <branch>` 실행
   - 완료 메시지 출력

**PR 생성은 `/finish` 커맨드 사용**

## 주의사항

- main/master 브랜치에서는 직접 푸시하지 않고 경고
- 커밋 전 변경 내용 반드시 사용자 확인
- force push 절대 금지
