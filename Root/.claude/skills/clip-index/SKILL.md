---
name: clip-index
description: 애니메이션 클립 검색을 위한 인덱스 관리. 클립 찾기, 태그 기반 검색, 카테고리 필터링 시 자동으로 사용됨. (project)
---

# Clip Index 스킬

애니메이션 클립을 인덱싱하여 태그/카테고리/속성 기반으로 검색합니다.

## 언제 사용하나요?

### 자동 트리거 조건

| 트리거 | 상황 |
|--------|------|
| 클립 검색 | "idle 애니메이션 찾아줘", "walk 클립 어디있어" |
| 태그 검색 | "combat 태그가 있는 클립", "locomotion 카테고리" |
| 속성 검색 | "루핑 클립만", "2초 이상 클립" |
| 인덱스 갱신 | "클립 인덱스 업데이트", "새 클립 추가됨" |

### 자동 트리거되는 에이전트

| 에이전트 | 활용 시점 |
|----------|----------|
| **executor-unity** | 애니메이션 작업 시 클립 검색 |
| **architect** | 애니메이션 시스템 설계 시 클립 분석 |
| **planner** | 애니메이션 요구사항 분석 |

## 서버 실행

```bash
# HTTP 모드 (권장)
cd tools/clip-index-server
npm run start:http

# 또는 개발 모드
npm run dev:http
```

기본 포트: `17901`

## MCP Tools

### MCP Tools (clip-index-server)

| Tool | 설명 | 파라미터 |
|------|------|----------|
| `playables.list_clips` | 클립 검색 | tags?, anyTags?, category?, sourceType?, isLooping?, minDuration?, maxDuration?, pathLike?, limit?, offset? |
| `playables.stats` | 인덱스 통계 | - |

**인덱싱은 CLI 스크립트로 실행** (`npm run index`)

## 사용 예시

### 클립 검색

```json
// 태그로 검색 (모든 태그 포함)
{"tool": "playables.list_clips", "args": {"tags": ["walk", "forward"]}}

// 태그로 검색 (하나라도 포함)
{"tool": "playables.list_clips", "args": {"anyTags": ["idle", "stand"]}}

// 카테고리로 검색
{"tool": "playables.list_clips", "args": {"category": "locomotion"}}

// 복합 검색
{"tool": "playables.list_clips", "args": {
  "category": "combat",
  "isLooping": false,
  "minDuration": 0.5,
  "maxDuration": 3.0
}}
```

### 인덱스 갱신

```bash
# Claude Code에서 /reindex-clips 명령어 사용
# 내부 동작:
cd tools/clip-index-server && npm run index

# 전체 재구축:
cd tools/clip-index-server && npm run index -- --full
```

## 카테고리

| 카테고리 | 설명 | 키워드 |
|----------|------|--------|
| `locomotion` | 이동 | walk, run, sprint, jog, strafe |
| `combat` | 전투 | attack, hit, block, dodge |
| `idle` | 대기 | idle, stand, wait, breath |
| `aerial` | 공중 | jump, fall, land, fly |
| `interaction` | 상호작용 | grab, push, pull, pick |
| `emote` | 감정표현 | wave, bow, dance, cheer |

## 태그 추출 규칙

경로에서 자동으로 태그 추출:
```
Assets/Animations/Locomotion/Walk/Walk_Forward.anim
→ tags: [locomotion, walk, forward]
→ category: locomotion
```

## 검색 결과 형식

```json
{
  "success": true,
  "count": 5,
  "clips": [
    {
      "id": "locomotion_walk_forward_abc123",
      "path": "Assets/Animations/Locomotion/Walk/Walk_Forward.anim",
      "clipName": "Walk_Forward",
      "sourceType": "anim",
      "duration": 1.033,
      "isLooping": true,
      "category": "locomotion",
      "tags": ["locomotion", "walk", "forward"]
    }
  ]
}
```

## 에러 핸들링

### 서버 미실행
```
Error: Connection refused
→ cd tools/clip-index-server && npm run start:http
```

### 인덱스 비어있음
```
{"success": true, "count": 0, "clips": []}
→ /reindex-clips 명령어 실행 (파일 스캔 + 인덱싱)
```

## 인덱스 최신화 전략

| 상황 | 권장 액션 |
|------|----------|
| 새 애니메이션 추가 | `/reindex-clips` (CLI 스크립트) |
| FBX 임포트 후 | `/reindex-clips` |
| 브랜치 전환 후 | `/reindex-clips full` |
| 전체 재구축 | `/reindex-clips full` |

**동작 방식**: glob으로 `**/*.anim`, `**/*.fbx.meta` 파일 스캔 → js-yaml 파싱 → SQLite 저장
