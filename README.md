# claude-plugins

GS팀의 Claude Code 플러그인 마켓플레이스

## 포함된 플러그인

| 플러그인 | 설명 | 커맨드 | 스킬 |
|---------|------|--------|------|
| **dev** | 클라이언트(Unity/C#) · 서버(Node.js/TS) 개발 도구 | `/dev:code-review` | `conventions` |
| **qa** | QA · 테스트 도구 | `/qa:test-case` | `bug-report` |
| **ta** | 테크니컬 아트 파이프라인 도구 | `/ta:asset-check` | `performance-budget` |
| **design** | 기획 도구 | `/design:spec-review` | `data-table-rules` |
| **sound** | 사운드 디자인 도구 | `/sound:audio-check` | `naming-conventions` |
| **pm** | 프로젝트 관리 도구 | `/pm:sprint-summary` | `meeting-notes` |

## 설치 방법

```bash
# 1. 마켓플레이스 추가
/plugin marketplace add bagelcode-gamestudio/claude-plugins

# 2. 플러그인 설치 (필요한 것만 골라서)
/plugin install dev@claude-plugins
/plugin install qa@claude-plugins
/plugin install design@claude-plugins
```

## 로컬 테스트 (GitHub에 올리기 전)

```bash
# 이 레포를 clone한 디렉토리에서
/plugin marketplace add .
/plugin install dev@claude-plugins
```

## 사용법

```bash
# 커맨드: 직접 호출
/dev:code-review src/app.ts
/qa:test-case 로그인 기능
/design:spec-review docs/gacha-spec.md

# 스킬: 관련 파일 작업 시 자동 활성화 (별도 호출 불필요)
```

## 구조

```
.claude-plugin/
  marketplace.json          ← 마켓플레이스 카탈로그
plugins/
  {plugin-name}/
    .claude-plugin/
      plugin.json            ← 플러그인 메타데이터
    commands/
      {command}.md           ← 슬래시 커맨드
    skills/
      {skill-name}/
        SKILL.md             ← 자동 활성화 스킬
```