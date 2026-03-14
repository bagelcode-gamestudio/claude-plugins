---
name: new-game
description: 새 게임 개발 가이드. 새 게임 만들기, 게임 추가 시 자동으로 사용됨.
trigger: 새 게임, 게임 추가, 신규 게임 ID 관련 작업 시
priority: medium
---

# 새 게임 개발 가이드

새 슬롯 게임을 추가할 때 사용하는 체크리스트입니다.

## 언제 사용하나요?

- "새 게임 만들어줘", "게임 추가해줘" 요청 시
- 신규 게임 ID 할당 요청 시

## 새 게임 개발 10단계 체크리스트

### 1단계: 게임 ID 할당
```
games/src/GAMES/games.ts에서 새 게임 ID 확인/할당
- 슬롯: 1-362 범위
- 비디오포커: 100001-100003, 400001-400017
- 케노: 200001-200005, 500001-500005
- 테이블: 600001-600004
```

### 2단계: 서버 게임 파일 생성
```
games/src/slot_game/{game_name}.ts
games/src/slot_game/{game_name}.interface.ts
games/src/slot_game/{game_name}.value.ts
```

### 3단계: 클라이언트 폴더 생성
```
client/Assets/Contents/Contents Group 1/{GameName}/
├── FSM/
│   └── Game Logic/
├── Data/
├── Scripts/
├── Prefabs/
└── Scenes/
```

### 4단계: Schema.json 생성
```json
{
  "CustomData{gameId}": {
    "type": "object",
    "properties": {
      // 게임별 커스텀 데이터 정의
    }
  }
}
```

### 5단계: gameInfo.json 생성
```json
{
  "gameId": {gameId},
  "name": "{short_name}",
  "displayName": "{Display Name}"
}
```

### 6단계: FSM 생성
- GameLogic_FSM.asset (메인 상태 머신)
- GameLogic_Idle_BT.asset (대기 상태)
- 보너스 FSM/BT (필요시)

### 7단계: SlotMachine 프리팹 생성
- 릴 구성 (5x3, 6x4 등)
- 심볼 설정
- 페이라인/Ways 설정

### 8단계: 보너스 ID 설정
```
보너스 ID = gameId × 100 + sequence
예: gameId 343 → 34300, 34301, 34302...
```

### 9단계: 서버 빌드
```bash
cd games && npm run dist-ts
cd server && npm run dist-ts
```

### 10단계: 테스트
- 로컬 스핀 테스트
- 보너스 트리거 테스트
- RTP 시뮬레이션

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `game-mapping` | 게임 ID 확인 |
| `build-games` | 게임 빌드 |
| `build-server` | 서버 빌드 |
| `schema-sync` | Schema 동기화 |
