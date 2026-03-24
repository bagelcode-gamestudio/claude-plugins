# gws-cli-setup

Google Workspace CLI (gws) 설치 및 설정 자동화 플러그인.

## Installation

gs-plugins marketplace에 포함되어 있습니다. marketplace가 설치되어 있다면 자동으로 사용 가능합니다.

## Usage

```
/gws-cli-setup
```

또는 다음과 같이 자연어로 트리거:
- "GWS CLI 설치해줘"
- "gws setup"
- "Google Workspace CLI 세팅"

## What it does

1. 환경 감지 (OS, 아키텍처, 패키지 매니저)
2. 의존성 확인 및 설치 (Node.js 18+, Homebrew, Cargo)
3. GWS CLI 설치 (npm / brew / cargo 중 최적 방법 선택)
4. OAuth 인증 설정 가이드 (사용자 핸드오프)
5. 최종 검증 (실제 API 호출로 동작 확인)

## Trigger Conditions

- GWS CLI가 설치되어 있지 않은 환경
- GWS CLI 설치 및 전체 설정이 필요한 경우
- 사용자가 'GWS 설치', 'gws setup', 'Google Workspace CLI 세팅' 요청 시
