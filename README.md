# claude-plugins

GS팀의 Claude Code 플러그인 마켓플레이스

## 포함된 플러그인

**starter** (v0.1.0)
- `/starter:review [file]` — 빠른 코드 리뷰
- `team-conventions` 스킬 — .ts/.js/.cs 파일 작업 시 팀 컨벤션 자동 적용

## 설치 방법

```bash
# 1. 마켓플레이스 추가
/plugin marketplace add bagelcode-gamestudio/claude-plugins

# 2. 플러그인 설치
/plugin install starter@claude-plugins
```

## 로컬 테스트 (GitHub에 올리기 전)

```bash
# 이 레포를 clone한 디렉토리에서
/plugin marketplace add .
/plugin install starter@claude-plugins
```

## 사용법

```bash
# 커맨드: 직접 호출
/starter:review src/app.ts

# 스킬: .ts/.js/.cs 파일 작업 시 자동 활성화 (별도 호출 불필요)
```

## 구조

```
.claude-plugin/
  marketplace.json        ← 마켓플레이스 카탈로그
plugins/
  starter/
    .claude-plugin/
      plugin.json          ← 플러그인 메타데이터
    commands/
      review.md            ← /starter:review 커맨드
    skills/
      team-conventions/
        SKILL.md           ← 자동 활성화 스킬
```
