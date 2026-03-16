---
description: "New API endpoint addition guide for LOGIC server."
user-invocable: false
globs:
  - "**/LOGIC/routes/*.ts"
  - "**/LOGIC/app.ts"
---

# 새 API 추가 가이드

LOGIC 서버에 새 API 엔드포인트를 추가할 때 사용합니다.

## API 추가 단계

### 1단계: 라우터 파일 생성/수정

```typescript
// server/src/LOGIC/routes/route_{feature}.ts

import { Router, Request, Response } from 'express';
import { authMiddleware } from '../middleware/auth';

const router = Router();

// GET 엔드포인트
router.get('/v3/{feature}/info', authMiddleware, async (req: Request, res: Response) => {
    try {
        const userId = req.user.id;

        // 로직 처리
        const result = await getFeatureInfo(userId);

        res.json({
            error: 0,
            contents: result
        });
    } catch (error) {
        res.json({
            error: 1,
            message: error.message
        });
    }
});

// POST 엔드포인트
router.post('/v3/{feature}/action', authMiddleware, async (req: Request, res: Response) => {
    try {
        const { param1, param2 } = req.body;
        const userId = req.user.id;

        // 파라미터 검증
        if (!param1) {
            throw new Error('param1 is required');
        }

        // 로직 처리
        const result = await performAction(userId, param1, param2);

        res.json({
            error: 0,
            contents: result,
            user_sync_info: await getUserSyncInfo(userId)
        });
    } catch (error) {
        res.json({
            error: 1,
            message: error.message
        });
    }
});

export default router;
```

### 2단계: 라우터 등록

```typescript
// server/src/LOGIC/app.ts

import featureRouter from './routes/route_{feature}';

app.use('/api', featureRouter);
```

### 3단계: 타입 정의

```typescript
// server/src/LOGIC/types/{feature}.ts

export interface FeatureRequest {
    param1: string;
    param2?: number;
}

export interface FeatureResponse {
    result: any;
    timestamp: number;
}
```

### 4단계: 클라이언트 연동

```csharp
// MetaSystem에서 호출
MetaSystem.Request<FeatureResponse>(
    "/v3/{feature}/action",
    new FeatureRequest { param1 = "value" },
    (response) => {
        // 응답 처리
        Blackboard.Set("./feature/result", response.result);
    }
);
```

## 기존 API 참고

| API | 용도 |
|-----|------|
| `POST /v3/slot/spin` | 스핀 요청 |
| `POST /v3/slot/claim` | 보너스 클레임 |
| `POST /v3/slot/end-turn` | 턴 종료 |
| `GET /v2/user/profile` | 프로필 조회 |
| `POST /v2/user/sync` | 정보 동기화 |

## 응답 형식 규칙

```json
{
    "error": 0,           // 0: 성공, 1+: 에러코드
    "contents": { },      // 실제 응답 데이터
    "user_sync_info": {   // 유저 동기화 정보 (선택)
        "credit": 1000000,
        "level": 42
    }
}
```

## 파일 위치

| 타입 | 경로 |
|------|------|
| 라우터 | `server/src/LOGIC/routes/` |
| 로직 | `server/src/LOGIC/logic/` |
| 타입 | `server/src/LOGIC/types/` |
