---
name: schema-sync
description: Schema.json 동기화 가이드. 스키마, CustomData 동기화 시 자동으로 사용됨.
trigger: 스키마 동기화, Schema.json, CustomData 타입 관련 작업 시, *.interface.ts 수정 시
priority: medium
---

# Schema.json 동기화

서버 CustomData와 클라이언트 Schema.json을 동기화합니다.

## 언제 사용하나요?

- "스키마 동기화해줘", "Schema 확인해줘" 요청 시
- 서버 CustomData 수정 후
- 클라이언트에서 데이터 파싱 오류 발생 시
- **`*.interface.ts` 파일 수정 시** (executor가 자동 질문)

## ⚠️ 자동 트리거 (executor 연동)

executor agent가 다음 파일을 수정하면 이 스킬을 참조하여 동기화합니다:

| 감지 파일 | 동기화 대상 |
|-----------|-------------|
| `games/src/slot_game/*.interface.ts` | 해당 게임의 `schema.json` |
| `games/src/slot_game/*_response*.ts` | 관련 게임의 `schema.json` |
| `server/src/**/*Response*.ts` | 관련 `schema.json` |

> **참고**: executor는 수정 후 사용자에게 동기화 여부를 질문합니다.

## Schema.json 구조

```json
{
  "CustomData{gameId}": {
    "type": "object",
    "properties": {
      "fieldName": {
        "type": "number|string|boolean|array|object",
        "description": "필드 설명"
      },
      "nestedObject": {
        "type": "object",
        "properties": {
          "subField": { "type": "number" }
        }
      },
      "arrayField": {
        "type": "array",
        "items": { "type": "number" }
      }
    }
  }
}
```

## 동기화 체크리스트

### 1. 서버 인터페이스 확인

```typescript
// games/src/slot_game/{game}.interface.ts
export interface CustomData{gameId} {
    fieldName: number;
    nestedObject: {
        subField: number;
    };
    arrayField: number[];
}
```

### 2. 클라이언트 Schema.json 위치

```
client/Assets/Contents/Contents Group 1/{GameName}/Data/Schema.json
```

### 3. 타입 매핑

| TypeScript | Schema.json |
|------------|-------------|
| `number` | `"type": "number"` |
| `string` | `"type": "string"` |
| `boolean` | `"type": "boolean"` |
| `T[]` | `"type": "array", "items": {...}` |
| `object` | `"type": "object", "properties": {...}` |
| `T \| null` | 추가: `"nullable": true` |

### 4. 검증 명령어

```bash
# Schema.json 파일 찾기
find client/Assets/Contents -name "Schema.json" | head -20

# 특정 게임 스키마 확인
cat "client/Assets/Contents/Contents Group 1/{GameName}/Data/Schema.json"

# 서버 인터페이스 확인
cat games/src/slot_game/{game_name}.interface.ts
```

## 자주 발생하는 문제

### 1. 필드 누락
- 서버에서 새 필드 추가했으나 Schema.json 미반영
- **해결**: Schema.json에 해당 필드 추가

### 2. 타입 불일치
- 서버: `number[]`, 클라이언트: `"type": "number"`
- **해결**: `"type": "array", "items": { "type": "number" }`로 수정

### 3. Optional 필드
- 서버: `field?: number`
- 클라이언트: 필드 존재 확인 필요

## Blackboard 경로

스키마에 정의된 데이터는 다음 경로에 저장됨:

| 경로 | 용도 |
|------|------|
| `./spin/response/customData/CustomData{gameId}/` | 스핀 응답 |
| `./bonus/response/customData/CustomData{gameId}/` | 보너스 응답 |

## 연관 스킬

| 스킬 | 용도 |
|------|------|
| `game-mapping` | 게임 ID 확인 |
| `blackboard-path` | 데이터 경로 |
