# Status Command

프로젝트의 현재 상태를 요약합니다.

## 실행 단계

1. **Git 상태**
   ```
   git branch -v          # 현재 브랜치
   git status --short     # 변경 파일
   git stash list         # stash 목록
   ```

2. **브랜치 현황**
   ```
   git branch -a          # 모든 브랜치
   git log --oneline -5   # 최근 커밋 5개
   ```

3. **진행 중인 태스크**
   - `docs/tasks/` 폴더의 미완료 태스크 목록
   - 각 태스크의 상태 체크박스 확인

4. **Unity 상태** (Unity 프로젝트인 경우)
   ```
   unity.compilation.status {}
   unity.editor.console.read { "level": "error", "limit": 5 }
   ```

5. **요약 출력**
   ```
   📍 브랜치: agent/xxx
   📝 변경: 3 files modified
   📋 태스크: 2 in progress
   ⚠️ 에러: 0
   ```
