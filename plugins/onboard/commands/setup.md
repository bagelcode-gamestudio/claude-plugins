---
description: GS Claude Plugins marketplace onboarding — connect marketplace, pick role, install plugins
argument-hint: [role: dev|qa|ta|design|sound|pm|all]
---

# GS Marketplace Onboarding

새로운 환경에서 GS Claude Plugins 마켓플레이스를 연결하고 역할에 맞는 플러그인을 설치합니다.

## Procedure

### Step 1: Marketplace 연결 확인

```bash
/plugin marketplace add bagelcode-gamestudio/claude-plugins
```

이미 연결된 경우 스킵.

### Step 2: 역할 확인

argument로 역할이 지정된 경우 바로 Step 3으로.
없으면 사용자에게 질문:

> 어떤 역할로 작업하시나요? (복수 선택 가능, 쉼표로 구분)
>
> | 역할 | 플러그인 | 주요 기능 |
> |------|---------|----------|
> | **dev** | dev | 슬롯 게임 서버 개발, 시뮬레이션, 보너스/스핀 플로우 |
> | **qa** | qa | 테스트 케이스 생성, 버그 리포트 템플릿 |
> | **ta** | ta | 에셋 검증, 퍼포먼스 버짓, 텍스처/메시 최적화 |
> | **design** | design | 기획 스펙 리뷰, 데이터 테이블 규칙 |
> | **sound** | sound | 오디오 체크, 네이밍 컨벤션 |
> | **pm** | pm | 스프린트 요약, 회의록 |
> | **all** | 전체 | 위 전부 설치 |
>
> 추가 옵션:
> | 옵션 | 플러그인 | 설명 |
> |------|---------|------|
> | **adb-mcp** | adb-mcp | Adobe MCP 설정 (Photoshop/Premiere 등 AI 연동) |

### Step 3: 플러그인 설치

선택된 역할에 해당하는 플러그인을 순차 설치:

```bash
/plugin install {plugin-name}@claude-plugins
```

`all` 선택 시 전체 설치:
```bash
/plugin install dev@claude-plugins
/plugin install qa@claude-plugins
/plugin install ta@claude-plugins
/plugin install design@claude-plugins
/plugin install sound@claude-plugins
/plugin install pm@claude-plugins
```

`adb-mcp`는 명시적 요청 시에만 설치 (Adobe 앱 사용자 전용).

### Step 4: 설치 확인

```bash
/plugin list
```

설치된 플러그인 목록과 사용 가능한 커맨드를 표시:

| 플러그인 | 커맨드 | 용도 |
|---------|--------|------|
| dev | `/dev:slot-server-impl`, `/dev:run-sim` | 슬롯 구현, 시뮬레이션 |
| qa | `/qa:test-case` | 테스트 케이스 생성 |
| ta | `/ta:asset-check` | 에셋 검증 |
| design | `/design:spec-review` | 스펙 리뷰 |
| sound | `/sound:audio-check` | 오디오 체크 |
| pm | `/pm:sprint-summary` | 스프린트 요약 |
| adb-mcp | `/adb-mcp:setup` | Adobe MCP 설정 |

### Step 5: 완료 메시지

```
✅ GS Claude Plugins 설정 완료!

설치된 플러그인: {설치된 목록}

시작하기:
  /dev:slot-server-impl 343    — 슬롯 서버 구현
  /qa:test-case 로그인 기능      — 테스트 케이스 생성
  /design:spec-review spec.md  — 스펙 리뷰

전체 커맨드 보기: /plugin list
```

## Error Handling

| 에러 | 해결 |
|------|------|
| marketplace add 실패 | 네트워크 확인, `gh auth status`로 GitHub 인증 확인 |
| plugin install 실패 | `/plugin marketplace add bagelcode-gamestudio/claude-plugins` 재실행 |
| 이미 설치됨 경고 | 정상 — 스킵하고 다음 진행 |
