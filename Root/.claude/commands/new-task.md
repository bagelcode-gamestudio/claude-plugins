# New Task Command

새 태스크를 시작합니다. task-id 생성, 브랜치 생성, 태스크 문서 생성을 한번에 처리합니다.

## 실행 단계

1. **현재 브랜치 확인 및 Parent 기록**
   - 현재 브랜치를 **Parent Branch**로 기록
   - agent/* 브랜치에서 실행 시 경고: "이미 작업 브랜치입니다. 계속하시겠습니까?"
   - 최신 상태로 pull

2. **task-id 생성을 위한 질문**

   **Q1. 우선순위는?**
   - H (High) - 긴급/핵심 기능
   - M (Mid) - 일반 기능
   - L (Low) - 개선/리팩토링

   **Q2. 담당자 이니셜은?**
   - 예: ds, jk, mh

   **Q3. 키워드는?**
   - kebab-case로 2-3단어
   - 예: lobby-ui, player-movement, bug-login

3. **task-id 생성**
   ```
   {priority}-{date}-{initials}-{keyword}
   예: H1-2025-12-03-ds-lobby-ui
   ```

   > ⚠️ **날짜는 반드시 환경 정보(env)의 `Today's date`에서 가져옵니다.**
   > 대화 내용이나 예시에서 추측하지 마세요.

4. **브랜치 생성**
   ```
   git checkout -b agent/<task-id>
   ```

5. **태스크 문서 생성**
   - `docs/tasks/<task-id>.md` 파일 생성
   - task-create 스킬의 템플릿 사용

6. **다음 단계 안내**
   ```
   ✅ 태스크 생성 완료!

   📋 Task: <task-id>
   🌿 Branch: agent/<task-id>
   🔀 Parent: <parent-branch>
   📄 Doc: docs/tasks/<task-id>.md

   다음 단계:
   1. 요구사항이 불명확하면 question-set 스킬로 정리
   2. 스펙 확정 후 구현 시작
   3. 완료 시 /finish → Parent 브랜치로 머지
   ```
